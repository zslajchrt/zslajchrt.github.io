---
layout: post
title: Developing Protean Application With Morpheus - Part 8
comments: true
permalink: developing-protean-applications-part8
---

```scala
case class Connection(userId: String)

case class MarketingPersonaDemographics(ageGroup: Option[Int], gender: Option[Int], salaryGroup: Option[Int], location: Option[Int], education: Option[Int], family: Option[Int])
case class MarketingPersonaProfessional(field: Option[Int], position: Option[Int], experience: Option[Int])
case class MarketingPersonaHobby(fields: List[Int])
case class MarketingPersona(demographics: Option[MarketingPersonaDemographics], professional: Option[MarketingPersonaProfessional], hobbies: Option[MarketingPersonaHobby])
```

TODO: link to the loaders, which are pretty same as the others


```scala
val regUserOrEmpModel = parse[
  (RegisteredUserEntity with RegisteredUserPublicCommon) or
    (EmployeeEntity with EmployeePublicCommon) with
    \?[PersonConnectionsEntity] with
    \?[MarketingPersonaEntity]
  ](true)
```

```scala
val regUserOrEmpLoaderRef: &[$[(RegisteredUserLoader or
  EmployeeLoader or
  PersonConnectionsLoader or
  MarketingPersonLoader) with UserDatasourcesMock]] = regUserOrEmpKernel

val regUserOrEmpLoaderKernel = *(regUserOrEmpLoaderRef, single[RegisteredUserLoader], single[EmployeeLoader],
  single[PersonConnectionsLoader], single[MarketingPersonLoader], single[UserDatasourcesMock])
```

```scala
```
