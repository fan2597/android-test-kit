# Version 2.0 (Released on: 2014.12.19) #

## Breaking Changes ##

  * Espresso has moved to a new namespace: **com.google.android.apps.common.testing.ui**.espresso -> **android.support.test**.espresso
  * Espresso artifacts have been renamed
    * **espresso**-1.1.jar -> **espresso-core**-release-2.0.jar
    * IdlingResource interface has been moved into a separate lib: **espresso-idling-resource**-release-2.0.jar
    * CountingIdlingResource now lives in **espresso-contrib**-release-2.0.jar (as it always should have)
  * Optional (a guava dependency) has been removed from the public API in order to support repackaging the guava dependency and avoid dex collision (a major source of pain). Affected methods:
    * `ViewAssertion.check`
    * `HumanReadables.getViewHierarchyErrorMessage`


## New Features<sup>1</sup> ##

  * Actions
    * ViewActions
      * `replaceText`
      * `openLink`
      * swipe up/down
    * espresso-contrib
      * RecyclerViewActions: handles interactions with RecyclerViews
      * PickerActions: handles interactions with Date and Time pickers

  * Matchers
    * RootMatchers
      * `isPlatformPopup`
    * ViewMatchers
      * `isJavascriptEnabled`
      * `withSpinnerText`
      * `withHint`
      * `isSelected`
      * `hasLinks`
    * LayoutMatchers: matchers for i18n-related layout testing
    * CursorMatchers: a collection of matchers for Cursors

  * Assertions
    * PositionAssertions (`isLeftOf`, `isAbove`, etc): collection of ViewAssertions for checking relative position of elements on the screen
    * LayoutAssertions: assertions for i18n-related layout testing<sup>2</sup>

  * Testapp: Many new sample activities/tests

  * Other
    * Espresso.unregisterIdlingResources and Espresso.getIdlingResources: provides additional flexibility for working with IdlingResources
    * ViewInteraction.withFailureHandler: allows overriding the failure handler from `onView`
    * `onData` support for AdapterViews backed by CursorAdapters


  * Bug fixes
    * ViewMatchers.isDisplayed matches views that take up the entire screen, but are less than 90% displayed
    * Performing swipe action call to DrawerActions.openDrawer() results in IdlingResourceTimeoutException
    * And [many more](https://code.google.com/p/android-test-kit/issues/list?can=1&q=status%3AFixed)


  * Other notable changes
    * Switched from building with Maven to Gradle
    * Moved espresso dependencies (Guava, Dagger, Hamcrest) out of the way to avoid DEX collisions
    * Changed to return success or failure when registering and unregistering idling resources
    * Lollipop support: Place message.recycle() behind an interface to account for version related changes
    * Switched target SDK 21 (mostly affects the testapp)


`[1]`: What about WebView support? While testing internally, we discovered some issues, which required major changes. We are putting the finishing touches on a new and improved API - it is coming shortly and it promises to be awesome.

`[2]`: The Google Accessibility team is planning additional work in this area, which will be integrated in future version of Espresso


---


# Version 1.1 (Released on: 2014.1.8) #

## Espresso ##
  * New swipeLeft and swipeRight ViewActions ([change](https://code.google.com/p/android-test-kit/source/detail?r=c4e4da01ca8d0fab31129c87f525f6e9ba1ecc02))
  * Multi-window support - an advanced feature that enables picking the target window on which Espresso should run the operation. ([change](https://code.google.com/p/android-test-kit/source/detail?r=1e5ee056231f7feb8e2a9704872a4520197f9ba2))
  * Improvements to TypeTextAction - allows typing text into a pre-focused view, which makes it easier to append text ([change](https://code.google.com/p/android-test-kit/source/detail?r=390ecfe1e41ab9b1a5bada2e6893d665c8d2d7d8))
  * Numerous bug fixes

## Espresso Contrib Library ##
  * This new library contains features that supplement Espresso, but are not part of the core library.
  * New DrawerActions for operating on DrawerLayout - has a dependency on android support library, hence we are keeping it outside of the core Espresso lib. ([change](https://code.google.com/p/android-test-kit/source/detail?r=cf47b3d1c9bcd7e0f1af1f748f2c4c52cfe7e5cd))

## Sample Tests ##
  * ... have been relocated to live in the same package as the testapp. ([change](https://code.google.com/p/android-test-kit/source/detail?r=353c1c8f67cfdf27e2b8dad1a83fdca1e07b0113))
  * maven POMs have been fixed to remove duplicate guava deps (mvn install should work now)

---
