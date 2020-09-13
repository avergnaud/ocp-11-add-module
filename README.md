# Java 11. modules

zoo.staff example from [https://www.selikoff.net/ocp11-1/](https://www.selikoff.net/ocp11-1/)

## project structure

build-output: contains the .class files
			
mods: contains the jar module(s)

staff: staff module sources

use: regular java sources

## building the module jar file 

Using openjdk version "12.0.1"

Creates the .class files in build-output/staff
```
javac -d build-output/staff/ staff/zoo/staff/internal/Util.java staff/zoo/staff/api/Staff.java staff/module-info.java
```

Packages the .class files in a jar module
```
jar -cvf mods/zoo.staff.jar -C build-output/staff/ .
```

## using zoo.staff.jar as an "unnamed module"
Uses the zoo.staff.jar module in a regular classpath (build), as an "unnamed module"
```
javac -cp mods/zoo.staff.jar -d build-output/use/ use/com/Main.java
```

Uses the zoo.staff.jar module in a regular classpath (run), as an "unnamed module"
```
java -cp mods/zoo.staff.jar:build-output/use/ com.Main
```
prints:
```
zoo staff internal Util get value
From Staff: zoo staff internal Util get value
```
This execution does not use zoo.staff.jar as a module. com.Main has access to the zoo.staff.internal package although it is not exported...

## using zoo.staff.jar as a module, enforcing its access restrictions
What if we wanted to use zoo.staff.jar on the classpath, but as a module?

Adding zoo.staff.jar on the module-path instead of the class-path is not enough:
```
java -p mods/zoo.staff.jar -cp build-output/use/ com.Main
```
prints:
```
Exception in thread "main" java.lang.NoClassDefFoundError: zoo/staff/internal/Util
	at com.Main.main(Main.java:10)
Caused by: java.lang.ClassNotFoundException: zoo.staff.internal.Util
	...
```

The solution is the --add-modules option:
```
java -p mods/zoo.staff.jar -cp build-output/use/ --add-modules zoo.staff com.Main
```
This adds zoo.staff.jar on the module path and declares zoo.staff as a module. And we get the correct Error:
```
Exception in thread "main" java.lang.IllegalAccessError: class com.Main (in unnamed module @0x6d311334) cannot access class zoo.staff.internal.Util (in module zoo.staff) because module zoo.staff does not export zoo.staff.internal to unnamed module @0x6d311334
	at com.Main.main(Main.java:10)
```

## Any module is readable by other *CLASSES* or *JARS* on the classpath
In order to test this assertion, we first build the jar file from the previously created .class files
```
jar -cvf build-output/use.jar -C build-output/use/ .
```

And we run the same java command, with a different cp:
```
java -p mods/zoo.staff.jar -cp build-output/use.jar --add-modules zoo.staff com.Main
```
this returns the same result as before:
```
Exception in thread "main" java.lang.IllegalAccessError: class com.Main (in unnamed module @0x448139f0) cannot access class zoo.staff.internal.Util (in module zoo.staff) because module zoo.staff does not export zoo.staff.internal to unnamed module @0x448139f0
	at com.Main.main(Main.java:10)
```

What if we remove the --add-modules option?
```
java -p mods/zoo.staff.jar -cp build-output/use.jar com.Main
```
this returns the same result as before:
```
Exception in thread "main" java.lang.NoClassDefFoundError: zoo/staff/internal/Util
	at com.Main.main(Main.java:10)
Caused by: java.lang.ClassNotFoundException: zoo.staff.internal.Util
	...
```

What is the meaning of "Any module is readable by other *CLASSES* or *JARS* on the classpath"?

This means that both:
```
java -cp mods/zoo.staff.jar:build-output/use.jar com.Main
```
and:
```
java -cp mods/zoo.staff.jar:build-output/use/ com.Main
```
return:
```
zoo staff internal Util get value
From Staff: zoo staff internal Util get value
```

