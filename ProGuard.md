# Introduction #

When packaging an apk, all classes of all libraries used by the program will be included, this makes the apk very huge, even exceeds [the capacity of dex](http://code.google.com/p/android/issues/detail?id=7147#c6). ProGuard can strip unused classes and methods, make it as small as possible.

# Details - New way #

AS of the version 3.0.0 the android maven plugin has built in support for proguard. It supports using the proguard.cfg file and you can activate it with
```
<proguard>
  <skip>false</skip>
</proguard>
```

More documentation is in the plugin help available with
```
mvn android:help -Dgoal=proguard -Ddetail=true
```

Also see
```
<proguard>
  <skip>false</skip>
  <config>proguard.cfg</config>
  <proguardJarPath>someAbsolutePathToProguardJar</proguardJarPath>
  <jvmArguments>
    <jvmArgument>-Xms256m</jvmArgument>
    <jvmArgument>-Xmx512m</jvmArgument>
  </jvmArguments>
</proguard>
```


# Details - Old way #

The possibility to use ProGuard together with android-maven-plugin was introduced in version 2.6.0 of this plugin, with this patch:

http://github.com/jayway/maven-android-plugin/pull/2

NOTE: For this to work with version 3.0.0+ of the plugin you need to add
```
<execution>
  <id>unpack</id>
  <phase>process-classes</phase>
  <goals>
    <goal>unpack</goal>
  </goals>
</execution>
```
to the android maven plugin configuration.

To let ProGuard process the project, add proguard-maven-plugin

```
  <build>
    <plugins>
      <plugin>
        <groupId>com.pyx4me</groupId>
        <artifactId>proguard-maven-plugin</artifactId>
        <version>2.0.4</version>
        <executions>
          <execution>
            <phase>process-classes</phase>
            <goals>
              <goal>proguard</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <injar>android-classes</injar>
          <libs>
            <lib>${java.home}/lib/rt.jar</lib>
          </libs>
          <obfuscate>false</obfuscate>
          <options>
            <option> -keep public class * extends android.app.Activity</option>
            <option>-keep public class * extends android.app.Application</option>
            <option>-keep public class * extends android.app.Service</option>
            <option>-keep public class * extends
              android.content.BroadcastReceiver</option>
            <option>-keep public class * extends
              android.content.ContentProvider</option>
            <option>-dontskipnonpubliclibraryclasses</option>
            <option>-dontoptimize</option>
            <option>-printmapping map.txt</option>
            <option>-printseeds seed.txt</option>
            <option>-ignorewarnings</option>
          </options>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

For options please refer to [proguard-maven-plugin](http://pyx4me.com/pyx4me-maven-plugins/proguard-maven-plugin/) and [ProGuard](http://proguard.sourceforge.net/). Depending on libraries you include in your application your proguard configuration might have to change.

For running this on MacOSX you might have to create a symlink for rt.jar. See instructions here http://bruehlicke.blogspot.com/2009/11/missing-rtjar-mac-os-x-using-proguard.html. A more generic approach for the different path name is using profiles and an example can be found in the MorseFlash sample.

# Samples #

Complete setup with more options and operating system switch for different rt.jar and so on

https://github.com/jayway/maven-android-plugin-samples/blob/stable/morseflash/morseflash-app/pom.xml

Use android-maven-plugin with scala and ProGuard

http://github.com/jayway/maven-android-plugin-samples/tree/stable/scala/

# proguard.cfg #

Instead of adding the configurations option into to `pom.xml` you can also add them to a separate file typically called `proguard.cfg`. This way you avoid the trouble with the '<' and '>' characters (mentioned in the comments) and have a easier to edit set-up if you have a more complex project.

```
  <build>
    <plugins>
      <plugin>
        <groupId>com.pyx4me</groupId>
        <artifactId>proguard-maven-plugin</artifactId>
        <version>2.0.4</version>
        <executions>
          <execution>
            <phase>process-classes</phase>
            <goals>
              <goal>proguard</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <injar>android-classes</injar>
          <libs>
            <lib>${java.home}/lib/rt.jar</lib>
          </libs>
          <obfuscate>false</obfuscate>
          <options>
            <option>@proguard.cfg</option>
          </options>
        </configuration>
      </plugin>
    </plugins>
  </build>
```