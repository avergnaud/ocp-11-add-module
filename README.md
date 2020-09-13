# Java 11. modules

zoo.staff example from [https://www.selikoff.net/ocp11-1/](https://www.selikoff.net/ocp11-1/)

## project structure

build-output: contains the .class files

mods: contains the jar module(s)

staff: staff module sources

use: regular java sources

## commands

Using openjdk version "12.0.1"

Creates the .class files in build-output/staff
'''
javac -d build-output/staff/ staff/zoo/staff/internal/Util.java staff/zoo/staff/api/Staff.java staff/module-info.java
'''

Packages the .class files in a jar module
'''
jar -cvf mods/zoo.staff.jar -C build-output/staff/ .
'''

Uses the zoo.staff.jar module in a regular classpath (build), as an "unnammed module"
'''
javac -cp mods/zoo.staff.jar -d build-output/use/ use/com/Main.java
'''

Uses the zoo.staff.jar module in a regular classpath (run), as an "unnamed module"
'''
java -cp mods/zoo.staff.jar:build-output/use/ com.Main
'''
prints:
'''
zoo staff internal Util get value
From Staff: zoo staff internal Util get value
'''


