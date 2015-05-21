# Native Libraries as Dependencies #

In an attempt reduce build times for projects involving a lot of native code, it would be useful to deploy the native libraries (.so) into the Maven Repository & only refer to them from the Maven pom (of the Android application).

As of version 2.8.3, native dependencies are supported directly by the Android Maven plugin.  All that is needed is to declare the dependencies as shown below.

## Add the required dependency ##

This defines the actual dependency on the native library.  The library will be placed in the `lib/armeabi` directory within the APK.

```
  <!-- Declare the dependency on a native library, already deployed in the Maven repository -->
  <dependency>
    <groupId>com.acme.android</groupId>
    <artifactId>libsample_jni</artifactId>
    <version>0.1</version>
    <scope>runtime</scope>
    <type>so</type>
  </dependency>
```

**Note**: The artifact id of the dependency will be the final name of the library when packaged in the APK.  Code loading the library in this particular case would use:

```
  System.loadLibrary("sample_jni");
```