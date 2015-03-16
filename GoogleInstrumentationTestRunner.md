# DEPRECATED (see [AndroidJUnitRunner](https://code.google.com/p/android-test-kit/wiki/AndroidJUnitRunnerUserGuide)) #

## Overview ##
In order to improve stability of instrumentation tests, we have made improvements to the default Android [InstrumentationTestRunner](http://developer.android.com/reference/android/test/InstrumentationTestRunner.html). Our test runner:
  * Guarantees that application onCreate has finished before testing begins (executing a test during app startup may lead to surprising results)
  * Ensures that all activities launched in the instrumentation are finished before instrumentation exits
  * Provides a more reliable Activity monitoring mechanism (see javadoc for [ActivityLifecycleMonitorRegistry](https://android-test-kit.googlecode.com/git/docs/javadocs/com/google/android/apps/common/testing/testrunner/ActivityLifecycleMonitorRegistry.html))
  * Enables [Mockito](https://code.google.com/p/mockito) to work with Eclair (API 7)

## Additional Notes ##
  * GITR is required by [Espresso](Espresso.md). However, you can use it standalone to run other instrumentation tests.
  * Have a custom test runner of your own? You can still take advantage of the features above by inheriting from [GoogleInstrumentation](https://code.google.com/p/android-test-kit/source/browse/testrunner/src/main/java/com/google/android/apps/common/testing/testrunner/GoogleInstrumentation.java)