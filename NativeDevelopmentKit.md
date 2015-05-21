# Introduction #

# Features #

Currently, the Android Maven Plugins NDK mojo supports the following scenarios when developing with native code:

  * Compile native shared library (.so) for use by other projects
    * Can depend on other native shared/static libraries
      * Declared as dependencies in the pom.xml
  * Include native library in your APK
    * Defined as a dependency in the pom.xml
    * Defined as 'mixed' with the APK code
  * Compile a native static library (.a) to be shared with other projects
    * Shares header files as part of project artifacts
      * Used by projects depending on static library as part of the build

# Prerequisites #

  * Android NDK [r7](https://code.google.com/p/maven-android-plugin/source/detail?r=7) or later

# Setup Environment #

Set the `ANDROID_NDK_HOME` the the root directory of your NDK installation.

For example:

```
REM Windows
SET ANDROID_NDK_HOME=C:\opt\android-ndk-r7b
```

```
# Linux
export ANDROID_NDK_HOME=$HOME/opt/android-ndk-r7b
```

These settings are best to persist by adding as permanent environment variables.

# Configuring Your Maven Project #

There are many variations on how the NDK

  * [Native Code Lives in the APK project](NativeDevelopmentKit#Native_Code_Lives_in_the_APK_project.md)
  * [Native Code Lives in the APKLIB project](NativeDevelopmentKit#Native_Code_Lives_in_the_APKLIB_project.md)

## Native Code Lives in the APK project ##

### Setup pom.xml ###

The following instructions apply to the setup of a apk with a native library.

### Cleanup ###

The NDK does not compile into the `target` directory but into `obj` and `libs` so they need to be cleaned as well:

```
      <plugin>
        <artifactId>maven-clean-plugin</artifactId>
        <configuration>
          <filesets>
            <fileset>
              <directory>libs</directory>
            </fileset>
            <fileset>
              <directory>obj</directory>
            </fileset>
          </filesets>
        </configuration>
      </plugin>
```

### Build Setup ###

Native buuld is activated by the `ndk-build` goal. Also the plugin need to know the location of the NDK installation.

```
      <plugin>
        <groupId>com.jayway.maven.plugins.android.generation2</groupId>
        <artifactId>android-maven-plugin</artifactId>
        <goals>
          <goal>ndk-build</goal>
        </goals>
        <configuration>
          <ndk>
            <path>${env.ANDROID_NDK_HOME}</path>
          </ndk>
        </configuration>
        <extensions>true</extensions>
      </plugin>
```

### Execute Build ###

Use the following two command to build (you need to set the pluginGroup as described in TipsAndTricks):

```
mvn android:ndk-build
mvn install
```

## Native Code Lives in the APKLIB project ##

The following instructions apply to the setup of a APK Library project with a native library. Note that the Library must have the same name as the artifact id. You can not add multiple native Libraries to a single APKLIB

### Cleanup ###

You should add a `maven-clean-plugin` section like the one described in [Native Code Lives in the APK project](NativeDevelopmentKit#Native_Code_Lives_in_the_APK_project.md).

### Library POM ###

The POM should be set up as `<packaging>apklib</packaging>`. Taking it from there you set the configuration as follow:

```
      <plugin>
        <groupId>com.jayway.maven.plugins.android.generation2</groupId>
        <artifactId>android-maven-plugin</artifactId>
        <goals>
          <goal>ndk-build</goal>
        </goals>
        <configuration>
          <deleteConflictingFiles>true</deleteConflictingFiles>
          <attachNativeArtifacts>true</attachNativeArtifacts>
          <clearNativeArtifacts>false</clearNativeArtifacts>
          <sign>
            <debug>false</debug>
          </sign>
          <proguard>
            <skip>true</skip>
          </proguard>
        </configuration>
        <extensions>true</extensions>
      </plugin>
```

### Application POM ###

Add an apklib dependency:

```
    <dependency>
      <groupId>my.package</groupId>
      <artifactId>Native-Lib</artifactId>
      <version>0.0.0</version>
      <type>apklib</type>
    </dependency>
```

### Execute Build ###

Use the following two command to build (you need to set the pluginGroup as described in TipsAndTricks):

```
mvn --projects ../Native-Lib android:ndk-build
mvn install
```