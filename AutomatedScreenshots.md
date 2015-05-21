Visual design is (increasingly) important to Android apps, and though devising tests for design is hard, you _can_ make visual checks much easier by capturing screenshots from your device during your normal testing. As of v3.1.2, the android-maven-plugin includes support for taking screenshots whenever requested to do so by your instrumentation test code. This
functionality is supplied by GitHub's [Android Screenshot](https://github.com/rtyley/android-screenshot-lib#readme) library, which uses [ddmlib](http://sourceforge.net/apps/trac/android4maven/wiki/ddmlib) to capture screenshots.

The screenshot capture speed is about 1 frame per second, so you can't get full motion video, but all screenshots captured for a particular test run are also compressed into an animated gif. This gif can be posted to your build radiator or dev chatroom on each build, to give quick visual feedback on the current state of the app.

# Usage #

Getting a screenshot is simple - you just invoke `poseForScreenshot()`
when you want one :

```
import static com.github.rtyley.android.screenshot.celebrity.Screenshots.poseForScreenshot;

    public void testAppearance() {
        startActivitySync(ConfigureMorseActivity.class);
        Instrumentation instrumentation = getInstrumentation();

        sleep(500); // robotium provides neater ways of waiting for the activity to initialise

        poseForScreenshot();
        instrumentation.sendStringSync("s");
        poseForScreenshot();
        instrumentation.sendStringSync("o");
        poseForScreenshot();
        instrumentation.sendStringSync("s");
        poseForScreenshot();
    }
```

The `poseForScreenshot()` method is a small dependency you need to add to your integration-test project, available from Maven Central as the [android-screenshot-celebrity](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.github.rtyley%22%20AND%20a%3A%22android-screenshot-celebrity%22) library.

The rest of the test is just setting up the activity so it's ready to have it's screenshot taken - ie startActivitySync() and then waiting a short period for the activity to be ready. The screenshots will be placed in a 'screenshots' folder under your instrumentation test target directory, for example:

morseflash/morseflash-instrumentation/target/screenshots

They're named sequentially by default (0000.png, 0001.png, etc), but you can set an explicit name for the shot using `poseForScreenshotNamed("foobar")` if you like. In addition, for each android device used in the tests, an animated-gif will be created showing all screenshots taken.

# Examples #

The morseflash project has an [example test](https://github.com/jayway/maven-android-plugin-samples/commit/8094291fee576fb21f202e320b56552a46a81957#L1R47) that takes screenshots and there's also an [example robotium test](https://github.com/github/gauges-android/blob/gauges-android-1.1/integration-tests/src/main/java/com/github/mobile/gauges/test/AppearanceTest.java#L88-109) in the open-source [Gaug.es](https://play.google.com/store/apps/details?id=com.github.mobile.gauges) android app by GitHub.