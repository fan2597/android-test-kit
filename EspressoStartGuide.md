
# Basics #

The Espresso API encourages test authors to think in terms of what a user might do while interacting with the application - locating UI elements and interacting with them. At the same time, the framework prevents direct access to activities and views of the application because holding on to these objects and operating on them off the UI thread is a major source of test flakiness. Thus, you will not see methods like getView and getCurrentActivity in the Espresso API. You can still safely operate on views by implementing your own [ViewAction](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/ViewAction.java)s and [ViewAssertion](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/ViewAssertion.java)s.


Here's an overview of the main components of Espresso:
  * [Espresso](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Espresso.java) – Entry point to interactions with views (via [onView](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Espresso.java#57) and [onData](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Espresso.java#68)). Also exposes APIs that are not necessarily tied to any view (e.g. [pressBack](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Espresso.java#145)).
  * [ViewMatchers](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/matcher/ViewMatchers.java) – A collection of objects that implement Matcher<? super View> interface. You can pass one or more of these to the onView method to locate a view within the current view hierarchy.
  * [ViewActions](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/action/ViewActions.java) – A collection of [ViewAction](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/ViewAction.java)s that can be passed to the ViewInteraction.perform method (for example, click).
  * [ViewAssertions](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/assertion/ViewAssertions.java) – A collection of [ViewAssertion](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/ViewAssertion.java)s that can be passed the ViewInteraction.check method. Most of the time, you will use the matches assertion, which uses a View matcher to assert the state of the currently selected view.

Example:
```
onView(withId(R.id.my_view))      // withId(R.id.my_view) is a ViewMatcher
  .perform(click())               // click() is a ViewAction
  .check(matches(isDisplayed())); // matches(isDisplayed()) is a ViewAssertion
```

## Finding a view with onView ##
In the vast majority of cases, the [onView](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Espresso.java#57) method takes a hamcrest matcher that is expected to match one — and only one — view within the current view hierarchy. Matchers are powerful and will be familiar to those who have used them with Mockito or Junit. If you are not familiar with hamcrest matchers, we suggest you start with a quick look at this [presentation](http://www.slideshare.net/shaiyallin/hamcrest-matchers).

Often the desired view has a unique R.id and a simple [withId](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/matcher/ViewMatchers.java#273) matcher will narrow down the view search. However, there are many legitimate cases when you cannot determine R.id at test development time. For example, the specific view may not have an R.id or the R.id is not unique. This can make normal instrumentation tests brittle and complicated to write because the normal way to access the view (with [findViewById()](http://developer.android.com/reference/android/app/Activity.html#findViewById(int))) does not work. Thus, you may need to access private members of the Activity or Fragment holding the view or find a container with a known R.id and navigate to its content for the particular view.

Espresso handles this problem cleanly by allowing you to narrow down the view using either existing [ViewMatchers](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/matcher/ViewMatchers.java) or your own custom ones.

Finding a view by its R.id is as simple as:
```
onView(withId(R.id.my_view))
```

Sometimes, R.id values are shared between multiple views. When this happens an attempt to use a particular R.id. gives you an AmbiguousViewMatcherException (for example). The exception message provides you with a text representation of the current view hierarchy, which you can search for and find the views that match the non-unique R.id:

```
java.lang.RuntimeException:
com.google.android.apps.common.testing.ui.espresso.AmbiguousViewMatcherException:
This matcher matches multiple views in the hierarchy: (withId: is <123456789>)
```

...

```
+----->SomeView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=false, enabled=true, selected=false, is-layout-requested=false, text=, root-is-layout-requested=false, x=0.0, y=625.0, child-count=1}
****MATCHES****
|
+------>OtherView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=true, enabled=true, selected=false, is-layout-requested=false, text=Hello!, root-is-layout-requested=false, x=0.0, y=0.0, child-count=1}
****MATCHES****
```

Looking through the various attributes of the views, you may find uniquely identifiable properties (in the example above, one of the views has the text "Hello!"). You can use this to narrow down your search by using combination matchers:
```
onView(allOf(withId(R.id.my_view), withText("Hello!")))
```

You can also use **not** to reverse any of the matchers:
```
onView(allOf(withId(R.id.my_view), not(withText("Unwanted"))))
```

See [ViewMatchers](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/matcher/ViewMatchers.java) for the view matchers provided by Espresso.

**Note:** In a well-behaved application, all views that a user can interact with should either contain descriptive text or have a content description (see [Android accessibility guidelines](http://developer.android.com/guide/topics/ui/accessibility/apps.html)). If you are not able to narrow down an onView search using 'withText' or 'withContentDescription', consider treating it as an accessibility bug.

**Note:** Use the least descriptive matcher that finds the one view you're looking for. Do not over-specify as this will force the framework to do more work than is necessary. For example, if a view is uniquely identifiable by its text, you need not specify that the view is also assignable from TextView. For a lot of views the R.id of the view should be sufficient.

**Note:** If the target view is inside an AdapterView (such as ListView, GridView, Spinner) the onView method might not work and it is recommended to use the onData method instead. See [Using onData with AdapterView controls for more information.](EspressoStartGuide#Using_onData_with_AdapterView_controls_(_ListView_,_GridView_,_..md)

## Performing an action on a view ##

When you have found a suitable matcher for the target view, it is possible to perform [ViewAction](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/ViewAction.java)s on it using the [perform](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/ViewInteraction.java#66) method.

For example, to click on the view:
```
onView(...).perform(click());
```
You can execute more than one action with one perform call:
```
onView(...).perform(typeText("Hello"), click());
```
If the view you are working with is located inside a ScrollView (vertical or horizontal), consider preceding actions that require the view to be displayed (like click and typeText) with scrollTo. This ensures that the view is displayed before proceeding to the other action:
```
onView(...).perform(scrollTo(), click());
```
**Note:** scrollTo will have no effect if the view is already displayed so you can safely use it in cases when the view is displayed due to larger screen size (for example, when your tests run on both smaller and larger screen resolutions).

See [ViewActions](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/action/ViewActions.java) for the view actions provided by Espresso.

## Checking if a view fulfills an assertion ##
Assertions can be applied to the currently selected view with the [check](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/ViewInteraction.java#118) method. The most used assertion is the [matches](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/assertion/ViewAssertions.java#58) assertion, it uses a ViewMatcher to assert the state of the currently selected view.

For example, to check that a view has the text "Hello!":
```
onView(...).check(matches(withText("Hello!")));
```

**Note:** Do not put "assertions" into the onView argument - instead, clearly specify what you are checking inside the check block. For example:

If you want to assert that "Hello!" is content of the view, the following is considered bad practice:

**~~onView(allOf(withId(...), withText("Hello!"))).check(matches(isDisplayed()));~~**

On the other hand, if you want to assert that a view with the text "Hello!" is visible  - for example after a change of the views visibility flag - the code is fine.

**Note:** Be sure to pay attention to the difference between [asserting that a view is not displayed](https://code.google.com/p/android-test-kit/wiki/EspressoSamples#Asserting_that_a_view_is_not_displayed) and [asserting that a view is not present](https://code.google.com/p/android-test-kit/wiki/EspressoSamples#Asserting_that_a_view_is_not_present) in the view hierarchy.

# Get started with a simple test using onView #
This simple test demonstrates how to use [onView](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Espresso.java#57). [SimpleActivity](https://code.google.com/p/android-test-kit/source/browse/testapp/src/main/java/com/google/android/apps/common/testing/ui/testapp/SimpleActivity.java) contains a Button and a TextView. When the Button is clicked the content of the TextView changes to "Hello Espresso!". Here's how to test this with Espresso (or take a look at the full test code at [BasicTest#testSimpleClickAndCheckText](https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/BasicTest.java#44) here).
### 1. Click on the button ###
The first step is to look for a property that helps to find the button. The button in the SimpleActivity has a unique R.id - perfect!
```
onView(withId(R.id.button_simple))
```
Now to perform the click:
```
onView(withId(R.id.button_simple)).perform(click());
```
### 2.  Check that the TextView now contains "Hello Espresso!" ###
The TextView with the text to verify has a unique R.id too:
```
onView(withId(R.id.text_simple))
```
Now to verify the content text:
```
onView(withId(R.id.text_simple)).check(matches(withText("Hello Espresso!")));
```

# Using onData with AdapterView controls (ListView, GridView, ...) #
[AdapterView](http://developer.android.com/reference/android/widget/AdapterView.html) is a special type of widget that loads its data dynamically from an Adapter. The most common example of an AdapterView is [ListView](http://developer.android.com/reference/android/widget/ListView.html). As opposed to static widgets like [LinearLayout](http://developer.android.com/reference/android/widget/LinearLayout.html), only a subset of the AdapterView children may be loaded into the current view hierarchy and a simple onView search would not find views that are not currently loaded. Espresso handles this by providing a separate onData interface which is able to first load the adapter item in question (bringing it into focus) prior to operating on it or any of its children.

**Note:** You may choose to bypass the "onData" loading action for items in adapter views that are initially displayed on screen because they are already loaded. However, it is safer to always use onData.

**Warning:** Custom implementations of AdapterView can have problems with the onData method, if they break inheritance contracts (particularly the getItem API). In such cases, the best course of action is to refactor your application code. If you cannot do so, you can implement a matching custom [AdapterViewProtocol](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/action/AdapterViewProtocol.java). Take a look at the default [AdapterViewProtocols](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/action/AdapterViewProtocols.java) provided by Espresso for more information.

# Get started with a simple test using onData #
This simple test demonstrates how to use [onData](https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Espresso.java#68). [SimpleActivity](https://code.google.com/p/android-test-kit/source/browse/testapp/src/main/java/com/google/android/apps/common/testing/ui/testapp/SimpleActivity.java) contains a Spinner with a few items - Strings that represent types of coffee beverages. When an item is selected, there is a TextView that changes to "One %s a day!" where %s is the selected item. The goal of this test is to open the Spinner, select a specific item and then verify that the TextView contains the item. As the Spinner class is based on AdapterView it is recommended to use onData instead of onView for matching the item. See [BasicTest#testClickOnSpinnerItemAmericano](https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/BasicTest.java#74) for the full sample code of the test.

### 1. Click on the Spinner to open the item selection ###
```
onView(withId(R.id.spinner_simple)).perform(click());
```

### 2. Click on the item "Americano" ###
For the item selection the Spinner creates a ListView with its contents - this can be very long and the element not contributed to the view hierarchy - by using onData we force our desired element into the view hierarchy. The items in the Spinner are Strings, we want to match an item that is a String and is equal to the String "Americano":
```
onData(allOf(is(instanceOf(String.class)), is("Americano")))
  .perform(click());
```

### 3. Verify that the TextView contains the String "Americano" ###
```
onView(withId(R.id.spinnertext_simple))
  .check(matches(withText(containsString("Americano"))));
```


For more information, check out the [Advanced Samples](https://code.google.com/p/android-test-kit/wiki/EspressoSamples) and let us know if you notice any documentation gaps or have any suggestions.

# Debugging #
Espresso provides useful debugging information when a test fails:
  * Logs all view actions to logcat
> For example:
```
  ViewInteraction: Performing 'single click' action on view with text: Espresso
```
  * Prints the view hierarchy in the exception string when onView fails
> If onView does not find the target view, a NoMatchingViewException is thrown. You can examine the view hierarchy in the exception string to analyze why the matcher did not match any views.
> If onView finds multiple views that match the given matcher, an AmbiguousViewMatcherException is thrown. The view hierarchy is printed and all views that were matched are marked with the MATCHES label:
```
java.lang.RuntimeException:
com.google.android.apps.common.testing.ui.espresso.AmbiguousViewMatcherException:
This matcher matches multiple views in the hierarchy: (withId: is <123456789>)
```
> ...
```
+----->SomeView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=false, enabled=true, selected=false, is-layout-requested=false, text=, root-is-layout-requested=false, x=0.0, y=625.0, child-count=1}
****MATCHES****
|
+------>OtherView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=true, enabled=true, selected=false, is-layout-requested=false, text=Hello!, root-is-layout-requested=false, x=0.0, y=0.0, child-count=1}
****MATCHES****
```
> When dealing with a complicated view hierarchy or unexpected behavior of widgets it is always helpful to use the [Android View Hierarchy Viewer](http://developer.android.com/tools/help/hierarchy-viewer.html) for an explanation.
  * Warns users about presence of AdapterView widgets
> When an onView operation throws a NoMatchingViewException and AdapterView widgets are present in the view hierarchy, the most common solution is to use onData. The exception message will include a warning with a list of the adapter views. You may use this information to invoke onData to load the target view.