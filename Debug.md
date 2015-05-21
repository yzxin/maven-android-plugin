# Introduction #

When an Android app has Maven `<dependencies>` to some jar library, it may not be possible to debug it from Eclipse by choosing "Debug as Android application", because the dependencies are not included then (depending on your configuration).

This is how you debug the app anyway.

# Details #

1. Build your app with Maven and deploy it to the device/emulator:
  * `mvn install android:deploy`

(See TipsAndTricks for how to enable the `android:` alias.)

You may wish to specify which device, in case you have more than one connected:
  * `mvn install android:deploy -Dandroid.device=usb    ` ...or...
  * `mvn install android:deploy -Dandroid.device=emulator`

2. Launch the app in the emulator or device.


## If you use Eclipse ##
3. Make sure the app project is open, and only just open, in Eclipse.

4. Open the DDMS in Eclipse.

5. Click the app's process line and then click the little bright green bug:

![http://maven-android-plugin.googlecode.com/svn/wiki/eclipse-ddms-debug.png](http://maven-android-plugin.googlecode.com/svn/wiki/eclipse-ddms-debug.png)

You are now debugging the app! Any breakpoints should work.

## If you are using IDEA (or something else) ##
3. Make sure the app project is open in your IDE.

4. Open the standalone DDMS.

5. Click the app's process line, and note the port number. In this case: `8700`.

![http://maven-android-plugin.googlecode.com/svn/wiki/ddms-debug.png](http://maven-android-plugin.googlecode.com/svn/wiki/ddms-debug.png)

6. Set up "Remote debugging" for `localhost` on the DDMS port, `8700`:

![http://maven-android-plugin.googlecode.com/svn/wiki/idea-remote-debug.png](http://maven-android-plugin.googlecode.com/svn/wiki/idea-remote-debug.png)

7. Launch the remote debugging configuration.

You are now debugging the app! Any breakpoints should work.