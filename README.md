# Gradle 6.5.1 enforcedPlatform bug

## Description

Unsatisfied version-range constraint from enforcedPlatform causes Gradle to report `NullPointerException (no error message)` in the presence of transitive dependency.

* Steps to reproduce:
  * Check out repository
  * Run `./gradlew build`

## Dependency setup

```gradle
// Dependencies in root project

dependencies {
    implementation enforcedPlatform('com.example:platform-project:1.0.0')
    implementation 'com.example:project-b:1.0.0'
}

// Dependencies in 'com.example:project-b:1.0.0'

dependencies {
    api 'org.apache.commons:commons-lang3:3.10'
}

// Dependencies in 'com.example:platform-project:1.0.0'

dependencies {
    constraints {
        // Deliberately unsatisfiable constraint with version range
        api 'org.apache.commons:commons-lang3:[99,)'
    }
}
```

## Error message with NullPointerException

```
FAILURE: Build failed with an exception.

* What went wrong:
Could not determine the dependencies of task ':compileJava'.
> Could not resolve all dependencies for configuration ':compileClasspath'.
   > java.lang.NullPointerException (no error message)
```


## Expected error message
```
FAILURE: Build failed with an exception.

* What went wrong:
Could not determine the dependencies of task ':compileJava'.
> Could not resolve all task dependencies for configuration ':compileClasspath'.
   > Could not find any version that matches org.apache.commons:commons-lang3:[99,).
     Versions that do not match:
       - 3.10
       - 3.9
       - 3.8.1
       - 3.8
       - 3.7
       - + 11 more
     Searched in the following locations:
       - https://repo.maven.apache.org/maven2/org/apache/commons/commons-lang3/maven-metadata.xml
     Required by:
         project : > project :platform-project
```

## Workarounds for avoiding the NullPointerException message

* Swapping the order of dependency declaration in root project.
* Declaring direct dependency to `org.apache.commons:commons-lang3` in root project.
* Changing `enforcedPlatform` to `platform` in root project.
* Changing constraint in platform dependency to use concrete version (e.g. `99.0.0`)

## Gradle version tested

```
------------------------------------------------------------
Gradle 6.5.1
------------------------------------------------------------

Build time:   2020-06-30 06:32:47 UTC
Revision:     66bc713f7169626a7f0134bf452abde51550ea0a

Kotlin:       1.3.72
Groovy:       2.5.11
Ant:          Apache Ant(TM) version 1.10.7 compiled on September 1 2019
JVM:          1.8.0_212 ( 25.212-b03)
OS:           Windows 10 10.0 amd64
```