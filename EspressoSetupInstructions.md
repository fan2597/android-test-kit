# Getting Started #

## Updating Espresso to use Android testing support library ##

You can now use the **Android Support Repository**, available through the SDK Manager to download the latest version. Find more detailed instructions on how to download the repository see below.

1. In your `build.gradle` file, replace the dependency on the external library jar:
```
  androidTestCompile files('libs/espresso-1.1-bundled.jar')  
  androidTestCompile files('libs/testrunner-1.1*.jar')
```
with
```
  androidTestCompile 'com.android.support.test.espresso:espresso-core:2.0'
  androidTestCompile 'com.android.support.test:testing-support-lib:0.1'
```

2. Update your code to use the new namespace. Rename,
```
  com.google.android.apps.common.testing.ui.espresso
```

to:

```
  android.support.test.espresso
```


3. Remove unneeded files from libs/.

## Installing from scratch ##

This guide covers installing Espresso using the SDK Manager and building it using Gradle. Android Studio is recommended.

### Setup your test environment ###

  * To avoid flakiness, we highly recommend that you turn off system animations on the virtual or physical device(s) used for testing.
  * Under 'Settings->Developer options' disable the following 3 settings and restart the device:
> It is also possible to do this [programmatically](https://code.google.com/p/android-test-kit/wiki/DisablingAnimations).

## Download Espresso ##
  * Open the **SDK manager** (in Android Studio it's in the Tools menu | Android | SDK Manager).
  * Make sure you have installed the **Android Support Repository** under **Extras** (espresso 2.0 is available from rev 11).
  * Open your app's `build.gradle` file. This is usually not the top-level `build.gradle` file but `app/build.gradle`.
  * Add the following lines inside dependencies:
```
androidTestCompile 'com.android.support.test.espresso:espresso-core:2.0'
androidTestCompile 'com.android.support.test:testing-support-lib:0.1'
```
  * If the project uses functionality from `espresso-contrib`, add:
```
androidTestCompile 'com.android.support.test.espresso:espresso-contrib:2.0'
```

## Set the instrumentation runner ##
  * Add to the same build.gradle file the following line in `android.defaultConfig`:
```
testInstrumentationRunner 
"android.support.test.runner.AndroidJUnitRunner"
```

## Example build.gradle file ##
```

apply plugin: 'com.android.application'

android {
    compileSdkVersion 21
    buildToolsVersion "21.1.2"

    defaultConfig {
        applicationId "com.my.awesome.app"
        minSdkVersion 10
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

    }
    ...
}

dependencies {
    // App's dependencies, including test
    compile 'com.android.support:support-annotations:21.0.3'

    // Testing-only dependencies
    androidTestCompile 'com.android.support.test:testing-support-lib:0.1'
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.0'
}

```

## Javadoc ##
  * [espresso-core-resource-2.0-javadoc](https://android-test-kit.googlecode.com/git/docs/javadocs/espresso/espresso-core-2.0-javadoc/reference/index.html)
  * [espresso-contrib-resource-2.0-javadoc](https://android-test-kit.googlecode.com/git/docs/javadocs/espresso/espresso-contrib-2.0-javadoc/reference/index.html)
  * [espresso-idling-resource-2.0-javadoc](https://android-test-kit.googlecode.com/git/docs/javadocs/espresso/espresso-idling-resource-2.0-javadoc/reference/index.html)


## Analytics ##

In order to make sure we are on the right track with each new release, the test runner collects analytics. More specifically, it uploads a hash of the package name of the application under test for each invocation. This allows us to measure both the count of unique packages using Espresso as well as the volume of usage.

If you do not wish to upload this data, you can opt out by passing the following argument to the test runner: `disableAnalytics "true"` (see [how to pass arguments](http://developer.android.com/reference/android/test/InstrumentationTestRunner.html)).

## Add the first test ##

Android Studio creates tests by default in `src/androidTest/java/com.example.package/`

Example test:
```
@LargeTest
public class HelloWorldEspressoTest extends ActivityInstrumentationTestCase2<MainActivity> {

    public HelloWorldEspressoTest() {
            super(MainActivity.class);
        }

    @Override
    public void setUp() throws Exception {
        super.setUp();
        getActivity();
    }

    public void testListGoesOverTheFold() {
        onView(withText("Hello world")).check(matches(isDisplayed()));
    }
}
```


## Create a test configuration ##
In Android Studio:
  * Open **Run menu** | **Edit Configurations**
  * Add a new **Android Tests** configuration
  * Choose a module
  * Add a specific instrumentation runner:
```
android.support.test.runner.AndroidJUnitRunner
```

## Run the configuration ##

Run the newly created configuration
Make sure tests are executed and pass