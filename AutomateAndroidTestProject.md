# Automating Android test projects with Maven #

This guide will show how an android test project created using Eclipse can be set up to build and execute using the Android Maven Plugin. This article assumes you have already performed the steps in the QuickStartForEclipseProject article.

To enable maven builds in your test project you need to create the following pom file in the project root.

```
<?xml version="1.0" encoding="UTF-8"?>

<project
  xmlns='http://maven.apache.org/POM/4.0.0'
  xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance'
  xsi:schemaLocation='http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd'
>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.yourcompany</groupId>
  <artifactId>your-test-artifact-id</artifactId>
  <packaging>apk</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>Your Test Android project Maven Enabled</name>
  <dependencies>
    <dependency>
      <groupId>com.google.android</groupId>
      <artifactId>android</artifactId>
      <version>2.3.1</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>com.google.android</groupId>
      <artifactId>android-test</artifactId>
      <version>2.3.1</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>com.yourcompany</groupId>
      <artifactId>your-artifact-id</artifactId>
      <type>apk</type>
      <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>com.yourcompany</groupId>
      <artifactId>your-artifact-id</artifactId>
      <scope>provided</scope>
      <type>jar</type>
      <version>0.0.1-SNAPSHOT</version>
    </dependency>
  </dependencies>
  <build>
    <sourceDirectory>src</sourceDirectory>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>1.6</source>
          <target>1.6</target>
        </configuration>
      </plugin>
      <plugin>
        <groupId>com.jayway.maven.plugins.android.generation2</groupId>
        <artifactId>android-maven-plugin</artifactId>
        <configuration>
          <sdk>
            <path>${env.ANDROID_HOME}</path>
            <platform>1.6</platform>
          </sdk>
        </configuration>
        <extensions>true</extensions>
      </plugin>
    </plugins>
  </build>
</project>
```

This pom is similar to the pom in the main project but has two more dependencies. They are both pointing to the main project, one to the apk and one to the jar file.

The dependency to the jar file is to enable the compiler to find your java classes from the main project. For this you use the provided scope so the classes are not actually included into your test project.

The dependency to the apk is to enable the android maven plugin to find the apk that it will run the tests against on the device or emulator when executing.

The android maven plugin will now run tests automatically on `mvn install` using instrumentation, just like Eclipse does. It uses the same underlying tools.

The automatic execution will work only as long as exactly one an emulator or Android device is connected. If you have more then one device or emulator running you will need to specify which device to use, with either of these on command-line options:

  * -Dandroid.device=usb
  * -Dandroid.device=emulator
  * -Dandroid.device=_specificdeviceid_

You can also disable instrumentation tests, these on command-line options:

  * -Dandroid.enableIntegrationTest=false

Defaults for both `android.enableIntegrationTest` and `android.device` can be set in the pom. For example:

```
<project>
  …
  <properties>
    <android.device>emulator</android.device>
  </properties>
  …
</project>
```

In order to execute the tests the main project needs to be built and installed into the local Maven repository. This is done with the `mvn install` command in the main projects root folder. When the main project have been installed the same command, `mvn install`, will build and execute the tests when executed from within the test projects root.

# Libraries #

If your project set-up contains libraries then those too need to be added as `<scope>provided</scope>` otherwise they will be added to the test which will result in a duplication the error «Class ref in pre-verified class resolved to unexpected implementation».

```
    <dependency>
      <groupId>com.yourcompany</groupId>
      <artifactId>your-library-artifact-id</artifactId>
      <scope>provided</scope>
      <type>jar</type>
      <version>0.0.1-SNAPSHOT</version>
    </dependency>
```

This is the workaround for [Bug #142](https://code.google.com/p/maven-android-plugin/issues/detail?id=142).

# Proguard #

If you use pro-guard then  there is the danger that the code you want to test as well as the code you use to do the testing is optimized away. You have to provide appropiate ´-keep´ options to prevent this:

```
   <option>-keep class com.yourcompany.test.R*</option>
   <option>-keep class com.yourcompany.**.Test_* { public void test_* (); }</option>
```

This is of course only an example. For it to work  all test classes need to start with `Test_` and all test methods with `test_`.

For the classes to be tested the following keep option might be (temporary) helpful:

```
-keepclassmembers public class com.mycompany.myclass {
    public *;
}
```

# Scala #

The Author made good experience with using Scala for instrumentation test. There was nothing special to observe apart from the need to use Proguard.