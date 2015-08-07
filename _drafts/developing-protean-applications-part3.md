---
layout: post
title: Developing Protean Application With Morpheus - Part 3
comments: true
permalink: developing-protean-applications-part3
---

###Primary Protodata

A by-product of the company's flagship service.

There are two event streams: search queries and visits. The search query stream
contains all search queries issued by the search engine users. The visits
stream contains URLs visited through the links returned by the search engine.
Both streams are kept in a storage for a certain period of time. The records in
the two streams may be correlated by means of the `searchId` attribute.

```json
{
  "searchId": 80989080940,
  "userId": "COOKIE-789987987439",
  "time": "2013-05-23T00:00:00Z",
  "country": "US",
  "keywords": ["prague", "beer", "pubs"]
}
```

```json
{
  "searchId": 80989080940,
  "userId": "COOKIE-789987987439",
  "time": "2013-05-23T00:00:10Z",
  "country": "US",
  "visitedUrl": "http://www.praguebeergarden.com"
}
```

###Preprocessed Protodata

The company can easily perform some preprocessing of the protodata such as
search sessions or keyword frequency data sets. The results of this preprocessing,
which can be performed online or offline, is stored in a separate storage.

####Search Sessions

```json
{
  "searchId": 80989080940,
  "userId": "COOKIE-789987987439",
  "time": "2013-05-23T00:00:00Z",
  "country": "US",
  "keywords": ["prague", "beer", "pubs"],
  "visitedUrls": [
    "http://www.praguebeergarden.com",
    "http://www.ratebeer.com/places/city/prague/0/56/",
    "http://www.praguebeermuseum.com/en"
  ]
}
```

####Keyword Frequency

```json
{
  "userId": "COOKIE-789987987439",
  "time": "2013-05-23T00:00:10Z",
  "keywords": [
    {
      "keyword": "beer",
      "freq": 9
    },
    {
      "keyword": "news",
      "freq": 5
    },
    {
      "keyword": "tv",
      "freq": 1
    }
  ]
}
```

He or she might be a beer lover.

We wish we knew more about the person hidden behind the `cookieId`.

URLs are also protodata.

###Identifying Users

The GigaMail service is only available for those who are registered or are
employee of the Big Company.

A user visiting the service is requested to sign in or register. An employee
is not required to register since his or her profile is retrieved from the
company's internal database.

After signed in, the user is associated with his or her profile, which can
have two forms: the registered user's profile or the employee's profile.

```json
{
  "public": {
    "nick": "joe4",
    "firstName": "Joe4",
    "lastName": "Doe4",
    "email": "joe4@gmail.com",
    "male": true,
    "birthDate": "1971-02-21T00:00:00Z"
  },
  "license": {
    "premium": true,
    "validFrom": "2015-02-21T00:00:00Z",
    "validTo": "2016-02-21T00:00:00Z"
  }
}
```

This is an employee's record from the Big Company's internal database.

```json
{
  "employee": {
      "employeeCode": "xyz9000",
      "position": "Developer",
      "department": "R&D",
      "personalData": {
        "firstName": "Joe1",
        "middleName": "D.",
        "lastName": "Doe1",
        "title": "Mr."
      }
    }
}
```

###Modeling User Profiles

```scala
case class RegisteredUserPublic(nick: String, firstName: String, lastName: String, email: Option[String], male: Option[Boolean], birthDate: Option[Date])

case class RegisteredUserLicense(premium: Boolean, validFrom: Date, validTo: Date)

case class RegisteredUser(`public`: RegisteredUserPublic, license: Option[RegisteredUserLicense])
```

```scala
case class EmployeePersonalData(firstName: String, middleName: Option[String], lastName: String, title: String, isMale: Boolean, birth: Date)

case class Employee(employeeCode: String, position: String, department: String, personalData: EmployeePersonalData)
```

```scala
@fragment
trait RegisteredUserEntity {

  protected var regUser: RegisteredUser = _

  def registeredUser = regUser
}

@fragment
trait EmployeeEntity {

  protected var emp: Employee = _

  def employee = emp
}
```

```scala
val regUserKernel = singleton[RegisteredUserEntity]
println(regUserKernel.~.registeredUser) // prints null

val empKernel = singleton[EmployeeEntity]
println(empKernel.~.employee) // prints null
```

####Loading Profiles

```scala
@dimension
trait UserDatasources {

  def registeredUserJson: JValue

  def employeeJson: JValue

  def initSources(userId: String)
}
```

```scala
import org.morpheus.FragmentValidator._

@fragment
trait RegisteredUserLoader {
  this: UserDatasources with RegisteredUserEntity =>

  def load: ValidationResult[RegisteredUserEntity] = {
    implicit val formats = DefaultFormats
    this.registeredUserJson.extractOpt[RegisteredUser] match {
      case Some(ru) =>
        this.regUser = ru
        success[RegisteredUserEntity]
      case None =>
        failure[RegisteredUserEntity]("invalid content")
    }
  }
}
```

```scala
val regUserLoaderRef: &[$[RegisteredUserLoader with UserDatasourcesMock]] = regUserKernel
val regUserLoaderKernel = *(regUserLoaderRef, single[RegisteredUserLoader], single[UserDatasourcesMock])
```

TODO: include the link to the source code of UserDatasourcesMock

```scala
regUserLoaderKernel.~.initSources("4")
regUserLoaderKernel.~.load match {
  case Failure(_, reason) =>
    sys.error(s"Cannot load registered user: $reason")
  case Success(_) =>
    println(regUserKernel.~.registeredUser)

}
```

```scala
val empKernel = singleton[EmployeeEntity]
val empLoaderRef: &[$[EmployeeLoader with UserDatasourcesMock]] = empKernel
// analogous to the previous code
```
