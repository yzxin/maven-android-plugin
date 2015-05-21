# Getting Started #

## Prerequisites ##

  1. JDK 1.6+ installed as required for Android development
  1. Android SDK (`r21.1` or later, latest is best supported) installed, preferably with all platforms, see http://developer.android.com/sdk/index.html
  1. Maven 3.1.1+ installed, see http://maven.apache.org/download.html
  1. Set environment variable ANDROID\_HOME to the path of your installed Android SDK and add $ANDROID\_HOME/tools as well as $ANDROID\_HOME/platform-tools to your $PATH. (or on Windows %ANDROID\_HOME%\tools and %ANDROID\_HOME%\platform-tools).


## Recommended: Test your development environment ##

If you want to test your environment, download and run the [Samples](Samples.md)`*`.

## Create your own Android application (apk) easily ##

The easiest way to create you own Android application project for Maven, is to use these Maven Archetypes:

https://github.com/akquinet/android-archetypes/wiki

## Create your own Android application (apk) manually ##

Alternatively, these are the steps to create an Android project manually:
<ol>
<blockquote><li>Create a project using the <code>android</code> tool:</li>
<ul><li><a href='http://developer.android.com/training/basics/firstapp/creating-project.html'>http://developer.android.com/training/basics/firstapp/creating-project.html</a>
</li></ul><li>Create a <code>pom.xml</code> in your project, using the sample <code>helloflashlight/pom.xml</code> as template. (See <a href='Samples.md'>Samples</a><code>*</code>.)</li>
<ul><li>Make these changes:<br>
</li><li>Change <code>&lt;groupId&gt;</code>, <code>&lt;artifactId&gt;</code> and <code>&lt;name&gt;</code> to your own. (Some tips: <a href='http://maven.apache.org/guides/mini/guide-naming-conventions.html'>Maven - Guide to Naming Conventions</a>)<br>
</li><li>Set a <code>&lt;version&gt;</code> tag for your project, for example <code>&lt;version&gt;0.1.0-SNAPSHOT&lt;/version&gt;</code>.<br>
</li><li>Update the Android dependency to the platform version you want to be compatible with as a minimum:</li></ul></blockquote>

<pre><code>  &lt;dependency&gt;<br>
    &lt;groupId&gt;com.google.android&lt;/groupId&gt;<br>
    &lt;artifactId&gt;android&lt;/artifactId&gt;<br>
    &lt;version&gt;1.5_r4&lt;/version&gt;<br>
    &lt;scope&gt;provided&lt;/scope&gt;<br>
  &lt;/dependency&gt;<br>
</code></pre>

Check <a href='http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22android%22'>http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22android%22</a> for available versions.<br>
Use one of these depending on the platform version you are developing against.<br>
<br>
<ul><li>In the android-maven-plugin configuration, set the correct platform number which corresponds to the version you declared for the android dependency, for example: <code>&lt;platform&gt;7&lt;/platform&gt;</code>. You can use API level (e.g. 7) or platform version (e.g. 2.1)<br>
</li></ul><blockquote><li>You won't need these files/directories with Android Maven Plugin, so you can remove them:</li>
<pre><code>     rm -r bin build.xml build.properties libs<br>
</code></pre>
<blockquote><i>The <code>default.properties</code> file is currently not used by Android Maven Plugin, but will be, so leave that in place for now. <code>default.properties</code> and the <code>gen</code> folder are used by the ADT plugin in Eclipse, if you use that.</i><br />
<i>(Also leave the "test" folder for now. You can later move it out to its own project. If you want, you can then look at the <code>apidemos-15-instrumentationtest</code> sample project.)</i>
</blockquote><li>To build your apk, simply:</li>
<pre><code>     mvn clean install<br>
</code></pre>
<li>To deploy your apk to the connected device:</li>
<pre><code>     mvn android:deploy<br>
</code></pre></blockquote>

<sup><i>See <a href='TipsAndTricks.md'>TipsAndTricks</a> for instruction on how to use the android plugin name even when not in a android project directory.</i></sup>

</ol>

Notes:

  1. Do not put tests in `src/test/java` that you want to be run by the Android Maven Plugin. If you put your tests there, Maven runs them as normal JUnit tests in the JVM, with the JAR files that are accessible on the classpath, which includes the android.jar. Because the android.jar only contains empty methods that throw exceptions, you get exceptions of the form `java.lang.RuntimeException: Stub!`. Instead, the tests need to go in `src/main/java`, where they will not be run by Maven as tests during compilation, but will be included in the apk and deployed and run on the device (where the android.jar has method implementations instead of stubs).
  1. If you have JUnit tests that don't call any Android APIs (directly or transitively), put them in `src/test/java` so JUnit runs them locally and more quickly than if run on the device.
  1. Make sure the Android Maven goal is set to `<goal>jar-no-fork</goal>` instead of `<goal>test-jar-no-fork</goal>`. Do this even for projects that only contain tests.

## Run instrumentation tests on device ##
Check out the `apidemos-15-instrumentationtest` project in [Samples](Samples.md)`*`.

You can create a project like that, by moving your "test" folder out and renaming it to for example "myproject-instrumentationtest". Then rearrange the folders within it like above, and use the `pom.xml` from `apidemos-15-instrumentationtest` as template.

For a sample project with more feature features like jarsigning, zipaligin, release profile and proguard look at the morseflash example.