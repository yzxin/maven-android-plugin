# _Deprecation_ #

_You should probably use Android Library Projects (ApkLib) instead._

_ApkSourcesDependency was introduced in Android Maven Plugin long before Google supplied a way to use libraries in Android development. Now that both the official Android tools and Android Maven Plugin support proper Android Library Projects, use will probably want to use that instead: ApkLib_

# Introduction #

This is useful if you develop apk's which share some code, but still have some different content. The purpose is to share identical code through a "common-stuff" project, instead of copy/pasting it between two projects.

It might look like this:

```
  myapp-common-stuff
    - contains common java source, res resources, assets and aidl files. typically all domain objects, business logic, common strings and common UI elements.
    - 'mvn install' in this project generates myapp-common-stuff.apksources, which is used by the other specific projects.

  myapp-for-phoneA
    - depends on myapp-common-stuff.
    - contains extra UI elements, and perhaps code, specific to phoneA.
    - 'mvn install' in this project generates myapp-for-phoneA.apk, for deployment on phoneA devices.

  myapp-for-tabletB
    - depends on myapp-common-stuff.
    - contains extra UI elements, and perhaps code, specific to tabletB.
    - 'mvn install' in this project generates myapp-for-tabletB.apk, for deployment on tabletB devices.
```

Note: It is of course recommended (as far as possible) to keep everything in the same apk and just make it work dynamically on every device. When that's not possible, for example when the functionality and UI differs too much, this kind of dependency setup can be nice to do.

# Details #

## `myapp-common-stuff` ##
In the apk project you wish to make available to other project as sources, declare the `pom.xml` as:

```
 <packaging>apk</packaging>
```

Also, add this to the android-maven-plugin's `<configuration>` tag:

```
 <attachSources>true</attachSources>
```

Then do `mvn install` in that project. Its source code, res and assets directories will be collected and stored in your local Maven repo.

## `myapp-for-phoneA` and `myapp-for-tabletB` ##
In the apk projects where you wish to use the src/main/java, res and assets directories of the dependency project, add this to the `<dependencies>` tag of the pom:

```
 <dependency>
     <groupId>enter.groupid.here</groupId>
     <artifactId>myapp-common-stuff</artifactId>
     <type>apksources</type>
 </dependency>
```

All projects must be `<packaging>apk</packaging>`. Note the use of `<type>apksources</type>` when declaring how you depend on the common source project.

When you build these projects, they will pull in the sources from the first project, and include them. A new R.java and new .java files from .aidl files will be built, based on the entire codebase and the entire collection of <b>res</b>ources.


## Overwrite vs. Overlay ##

Be aware of the names you give to res files, for example `strings.xml`. Make sure they don't collide, because then they will be overwritten when copied on top of each other. Use for example `commonStrings.xml` in the project with common code and resources, and other specific names in each project.

### `res-overlay` ###

If you don't want a certain resource file to overwrite a dependency resource file with the same name, you can put it in a `res-overlay` directory instead of the `res` directory. That way, only the specific XML tags you specify in the overlay resource file, will overwrite those specific xml tags in its dependency resource counterpart.