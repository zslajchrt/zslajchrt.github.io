---
layout: post
title: Developing Protean Application With Morpheus - Part 4
comments: true
permalink: developing-protean-applications-part4
---

###Loading Profiles In Batch

```scala
import org.morpheus.FragmentValidator._

@dimension
trait FragmentLoader[F] {

  def load: ValidationResult[F]
}

@fragment
trait RegisteredUserLoader extends FragmentLoader[RegisteredUserEntity] {
  ...
}

@fragment
trait EmployeeLoader extends FragmentLoader[EmployeeEntity] {
  ...
}
```

```scala
val regUserOrEmpKernel = singleton[RegisteredUserEntity or EmployeeEntity]
val regUserOrEmpLoaderRef: &[$[(RegisteredUserLoader or EmployeeLoader) with UserDatasources]] = regUserOrEmpKernel
val regUserOrEmpLoaderKernel = *(regUserOrEmpLoaderRef, single[RegisteredUserLoader], single[EmployeeLoader],
  single[UserDatasourcesMock])
```

```scala
regUserOrEmpLoaderKernel.~.initSources("5") // 4 = a registered user, 5 = an employee
```

```scala
for (loader <- regUserOrEmpLoaderKernel) {
  println(loader.myAlternative)
  loader.load
}
```

```scala
List(org.cloudio.morpheus.dci.socnet.objects.RegisteredUserLoader, org.cloudio.morpheus.dci.socnet.objects.UserDatasourcesMock, org.cloudio.morpheus.dci.socnet.objects.RegisteredUserEntity)
List(org.cloudio.morpheus.dci.socnet.objects.EmployeeLoader, org.cloudio.morpheus.dci.socnet.objects.UserDatasourcesMock, org.cloudio.morpheus.dci.socnet.objects.EmployeeEntity)
```

```scala
val regUserEnt = asMorphOf[RegisteredUserEntity](regUserOrEmpKernel)
println(regUserEnt.registeredUser)
val empEnt = asMorphOf[EmployeeEntity](regUserOrEmpKernel)
println(empEnt.employee)
```

####Disabling Invalid Fragments

```scala
val regUserOrEmpModel = parse[RegisteredUserEntity or EmployeeEntity](true)
var failedFragments: Option[Set[Int]] = None
val morphStrategy = MaskExplicitStrategy(rootStrategy(regUserOrEmpModel), true, () => failedFragments)
val regUserOrEmpKernel = singleton(regUserOrEmpModel, morphStrategy)
```

```scala
failedFragments = Some((for (loader <- regUserOrEmpLoaderKernel;
                             loaderResult = loader.load
                             if !loaderResult.succeeded;
                             frag <- loaderResult.fragment) yield frag.index).toSet)
```

```scala
val regUserEnt = asMorphOf[RegisteredUserEntity](regUserOrEmpKernel)
val empEnt = asMorphOf[EmployeeEntity](regUserOrEmpKernel)
```

```
Exception in thread "main" org.morpheus.AlternativeNotAvailableException
	at org.morpheus.BridgeAlternativeComposer.convertToHolders(MorphingStrategy.scala:446)
	at org.morpheus.Morpher.makeFragHolders$1(Morpher.scala:27)
	at org.morpheus.Morpher.morph(Morpher.scala:43)
	at org.morpheus.Morpher$.morph(Morpher.scala:116)
	at org.morpheus.Morpher$.morph(Morpher.scala:112)
	at org.morpheus.MorphKernel.make(MorphKernel.scala:91)
	at org.cloudio.morpheus.dci.socnet.objects.PersonSample$.main(Loaders.scala:329)
	at org.cloudio.morpheus.dci.socnet.objects.PersonSample.main(Loaders.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:140)
```

```scala
regUserOrEmpKernel.~.remorph
select[RegisteredUserEntity](regUserOrEmpKernel.~) match {
  case Some(regUserEnt) =>
    println(regUserEnt.registeredUser)
  case None =>
    select[EmployeeEntity](regUserOrEmpKernel.~) match {
      case Some(empEnt) => println(empEnt.employee)
      case None => require(false)
    }
}
```

```
Exception in thread "main" org.morpheus.NoAlternativeChosenException
	at org.morpheus.Morpher.makeFragHolders$1(Morpher.scala:25)
	at org.morpheus.Morpher.morph(Morpher.scala:43)
	at org.morpheus.Morpher$.morph(Morpher.scala:116)
	at org.morpheus.MorphKernel$$anon$2.morph(MorphKernel.scala:110)
	at org.morpheus.MutableMorphContext.proxy$lzycompute(MorphContext.scala:297)
	at org.morpheus.MutableMorphContext.proxy(MorphContext.scala:288)
	at org.morpheus.MorphKernel.mutableProxy_(MorphKernel.scala:114)
	at org.morpheus.MorphKernel.make_$tilde(MorphKernel.scala:96)
	at org.morpheus.MorphKernel.$tilde$lzycompute(MorphKernel.scala:84)
	at org.morpheus.MorphKernel.$tilde(MorphKernel.scala:84)
	at org.cloudio.morpheus.dci.socnet.objects.PersonSample$.main(Loaders.scala:333)
	at org.cloudio.morpheus.dci.socnet.objects.PersonSample.main(Loaders.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:140)
```

```scala
try {
  regUserOrEmpKernel.~.remorph
  select[RegisteredUserEntity](regUserOrEmpKernel.~) match {
    case Some(regUserEnt) =>
      println(regUserEnt.registeredUser)
    case None =>
      select[EmployeeEntity](regUserOrEmpKernel.~) match {
        case Some(empEnt) => println(empEnt.employee)
        case None => require(false)
      }
  }
}
catch {
  case ae: NoViableAlternativeException =>
    println("Cannot load registered user or employee")
}
```
