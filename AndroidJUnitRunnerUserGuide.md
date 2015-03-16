# User guide for AndroidJUnitRunner #

## Overview ##

The `AndroidJUnitRunner` is a new unbundled test runner for Android, which is part of the **Android Support Test Library** and can be downloaded via the **Android Support Repository**. The new runner contains all improvements of [GoogleInstrumentationTestRunner](https://code.google.com/p/android-test-kit/wiki/GoogleInstrumentationTestRunner) and adds more features:

  * `JUnit4` support
  * Instrumentation Registry for accessing `Instrumentation`, `Context` and `Bundle` Arguments
  * Test Filters `@SdkSupress` and `@RequiresDevice`
  * Test timeouts
  * Sharding of tests
  * [RunListener](http://junit.sourceforge.net/javadoc/org/junit/runner/notification/RunListener.html) support to hook into the test run lifecycle
  * Activity monitoring mechanism `ActivityLifecycleMonitorRegistry`

This wiki page only covers the main features of the new runner. To set up  AndroidJUnitRunner please refer to the [Espresso Setup Instructions](https://code.google.com/p/android-test-kit/wiki/EspressoSetupInstructions)

## Javadoc ##
The docs for `AndroidJUnitRunner` will be temporarily hosted on android-test-kit and will be available on [d.android.com](http://d.android.com) soon.

Javadoc is available here:
[AndroidJUnitRunner Javadoc](https://android-test-kit.googlecode.com/git/docs/javadocs/testing-support-lib-0.1-javadoc/reference/packages.html)

## InstrumentationRegistry ##
`InstrumentationRegistry` is an exposed registry instance that holds a reference to the instrumentation running in the process and it's arguments and allows injection of the following instances:

  * `InstrumentationRegistry.getInstrumentation()`, returns the `Instrumentation` currently running.
  * `InstrumentationRegistry.getContext()`, returns the `Context` of this `Instrumentation`’s package.
  * `InstrumentationRegistry.getTargetContext()`, returns the  application `Context` of the target application.
  * `InstrumentationRegistry.getArguments()`, returns a copy of arguments `Bundle` that was passed to this `Instrumentation`. This is useful when you want to access the command line arguments passed to `Instrumentation` for your test.

## JUnit4 ##
JUnit4 API changes are largely an entirely new set of APIs compared to JUnit3 but the `AndroidJUnitRunner` provides full [JUnit4](https://github.com/junit-team/junit/wiki) support while being fully backwards compatible with legacy JUnit3 tests. `AndroidJUnitRunner` will pick up both type of tests and run them in the same test suite. Internally `AndroidRunnerBuilder` extends `JUnit`'s `AllDefaultPossibilitiesBuilder` to collect both JUnit3 and JUnit4 tests. This allows you to write new tests in JUnit4 without making any changes to your existing JUnit3 tests.

Please **DO NOT** mix and match JUnit3 and JUnit4 style tests in the same class because it will result in unexpected results.

To write a JUnit4 you need to annotate your test with `@RunWith(AndroidJUnit4.class)`. This way the runner will know that this is a JUnit4 test. `AndroidJUnitRunner` supports all JUnit4 Annotations and hence allows a more easy and readable running and writing of tests.

Some of the most frequently used annotations are:

  * **`@Test`** annotation is a replacement for the test-prefix naming convention used in JUnit3. JUnit4 test classes no longer require to extend `TestCase` or any of its subclasses. In fact JUnit4 tests cannot extend TestCase, otherwise AndroidJUnitRunner will treat them as JUnit3 tests.
  * **`@Test(expected=IllegalArgumentException.class)`**, in JUnit4 the expected parameter is used along with @Test annotation toreduces boilerplate code to a minimum. Just specify the expected parameter on the `@Test` annotation to replace the old JUnit3 convention:
```
try{ 
    foo.bar();
    fail("IllegalArgumentException expected");
} catch (IllegalArgumentException expected) {}
```
The same can be achieved with ExpectedException Rule, for more information please visit: [Exception Testing](https://github.com/junit-team/junit/wiki/Exception-testing).
    * **`@Before`**, JUnit4's equivalent to JUnit3's `setUp()` method and is invoked before every each test method. It is commonly used to setup your test fixture. You can have multiple `@Before` methods but the order which these methods are called is not fixed.
  * **`@After`**, JUnit4's equivalent to JUnit3's `tearDown()` method and runs after every test method. Use this to release any resources.
  * **`@BeforeClass`** annotation is used to annotate methods that should only be invoked once per test class. This is useful for expensive operations such as connecting to a database.
  * **`@AfterClass`**, called after all tests in the class ran. This is for releasing resources allocated in `BeforeClass`. For more information about test fixtures please visit: [Test Fixtures](https://github.com/junit-team/junit/wiki/Test-fixtures)

## Backwards compatibility to Android platform testing APIs ##

### `ActivityInstrumentationTestCase2` ###

Currently JUnit4 has limited native support for Android platform testing APIs. In order to use JUnit4 syntax with the existing Android platform testing APIs such as `ActivityInstrumentationTestCase2`, please follow these 5 steps:

  * Annotate your `!ActivityInstrumentationTestCase2` with `@RunWith(AndroidJUnit4.class)`.
  * Override `setUp()` method of `ActivityInstrumentationTestCase2` and annotate it with the `@Before` annotation and call the `super.setUp()` method.
  * Inject `Instrumentation` into the test class using `InstrumentationRegistry`. This was previously injected by the platforms old `InstrumentationTestRunner` and now need to be done manually.
```
    injectInstrumentation(InstrumentationRegistry.getInstrumentation());
```
  * Annotate all your tests with `@Test` so that they’ll be picked up by `AndroidJUnitRunner`
  * Override the `tearDown()` method of `ActivityInstrumentationTestCase2` and annotate it with `@After` and call the `super.tearDown()` method. This is important to not leak any of the any objects from your tests.

Here is a complete example of an `ActivityInstrumentationTestCase2` which uses JUnit4:
```
@RunWith(AndroidJUnit4.class)
@LargeTest
public class MyJunit4ActivityInstrumentationTest
            extends ActivityInstrumentationTestCase2<MyActivity> {

    private MyActivity mActivity;

    public MyJunit4ActivityInstrumentationTest() {
        super(MyActivity.class);
    }

    @Before
    public void setUp() throws Exception {
        super.setUp();
        injectInstrumentation(InstrumentationRegistry.getInstrumentation());
        mActivity = getActivity();
    }

    @Test
    public void checkPreconditions() {
        assertThat(mActivity, notNullValue());
        // Check that Instrumentation was correctly injected in setUp()
        assertThat(getInstrumentation(), notNullValue());
    }

    @After
    public void tearDown() throws Exception {
        super.tearDown();
    }

}
```

## Instrumentation Thread Handlers ##

Unlike `InstrumentationTestRunner`  and `GoogleInstrumentationRunner`, all `Handler`s must be created on the main thread with `AndroidJUnitRunner` . `Handler`s attached to the instrumentation thread will not work anymore and result in:

**`RuntimeException: "Can't create handler inside thread that has not called Looper.prepare()"`**

Creating `Handler`s on the main thread is pretty straight-forward and you can use one of these strategies:

  * Use the [`Instrumentation.runOnMainSync(Runnable)`](http://developer.android.com/reference/android/app/Instrumentation.html#runOnMainSync(java.lang.Runnable)) method to create your test `Handler` on the main thread. This works well if you want to setup a global scoped `Handler` in your test fixture.
  * Annotate all test cases that need to instantiate a local scope `Handler` with [@UiThreadTest](http://developer.android.com/reference/android/test/UiThreadTest.html) to instatiate a `Handler` on the main thread

**Please do not call `Looper.prepare()` in your test to workaround this issue**, doing this will result in unreliable, flaky and hard to reproduce bugs in your tests!