

<br><b>Note:</b> Feel like something is missing in the list above? Feel free to contribute an additional sample!<br>

<h1>ViewMatchers</h1>
<h2>Matching a view next to another view</h2>
A layout could contain certain views that are not unique by themselves (e.g. a repeating call button in a table of contacts could have the same R.id, contain the same text and have the same properties as other call buttons within the view hierarchy) For example, in this activity, the view with text "7" repeats across multiple rows:<br>
<br>
<img src='http://wiki.android-test-kit.googlecode.com/git/hasSibling.png' />


Often, the non-unique view will be paired with some unique label that's located next to it (e.g. a name of the contact next to the call button). In this case, you can use the hasSibling matcher to narrow down your selection:<br>
<pre><code>onView(allOf(withText("7"), hasSibling(withText("item: 0"))))<br>
  .perform(click());<br>
</code></pre>

<h2>Matching data using onData and a custom ViewMatcher</h2>
The activity below contains a ListView, which is backed by a <a href='http://developer.android.com/reference/android/widget/SimpleAdapter.html'>SimpleAdapter</a> that holds data for each row in a Map<String, Object>. Each map has an entry with key "STR" that contains the content (string, "item: x") and a key "LEN" that contains an Integer, the length of the content.<br>
<br>
<img src='http://wiki.android-test-kit.googlecode.com/git/list_activity.png' />

The code for a click on the row with "item: 50" looks like this:<br>
<pre><code>onData(allOf(is(instanceOf(Map.class)), hasEntry(equalTo("STR"), is("item: 50")))<br>
  .perform(click());<br>
</code></pre>
Let's take apart the Matcher<code>&lt;Object&gt;</code> inside onData:<br>
<pre><code>is(instanceOf(Map.class)) <br>
</code></pre>
narrows the search to any item of the AdapterView, which is a Map.<br>
<br>
In our case, this is every row of the list view, but we want to click specifically on "item: 50", so we narrow the search further with:<br>
<pre><code>hasEntry(equalTo("STR"), is("item: 50"))<br>
</code></pre>
This Matcher<code>&lt;String, Object&gt;</code> will match any Map that contains an entry with any key and value = "item: 50". As the code to look up this is long and we want to reuse it in other locations - let us write a custom "withItemContent" matcher for that.<br>
<br>
<pre><code>public static Matcher&lt;Object&gt; withItemContent(final Matcher&lt;String&gt; itemTextMatcher){<br>
  checkNotNull(itemTextMatcher);<br>
  return new BoundedMatcher&lt;Object, Map&gt;(Map.class) {<br>
    @Override<br>
    public boolean matchesSafely(Map map) {<br>
      return hasEntry(equalTo("STR"), itemTextMatcher).matches(map);<br>
    }<br>
<br>
    @Override<br>
    public void describeTo(Description description) {<br>
      description.appendText("with item content: ");<br>
      itemTextMatcher.describeTo(description);<br>
    }<br>
  };<br>
}<br>
</code></pre>

We use a <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/matcher/BoundedMatcher.java'>BoundedMatcher</a> as a base because we want to be able to only match on Objects of class Map. We override the matchesSafely method, put in the matcher we found earlier and match it against a Matcher<code>&lt;String&gt;</code> that can be passed as an argument. This allows us to do withItemContent(equalTo("foo")). For code brevity, we create another matcher that already does the equalTo for us and accepts a String.<br>
<pre><code>public static Matcher&lt;Object&gt; withItemContent(String expectedText) {<br>
  checkNotNull(expectedText);<br>
  return withItemContent(equalTo(expectedText));<br>
}<br>
</code></pre>
Now the code to click on the item is simple:<br>
<pre><code>onData(withItemContent("item: 50")) .perform(click());<br>
</code></pre>
For the full code of this test, take a look at <a href='https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/AdapterViewTest.java#48'>AdapterViewTest#testClickOnItem50</a> and the <a href='https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/LongListMatchers.java'>custom matcher</a>.<br>
<br>
<h2>Matching a specific child view of a view</h2>

The sample above issues a click in the middle of the entire row of a ListView. But what if we want to operate on a specific child of the row? For example, we would like to click on the second column of the row of the <a href='https://code.google.com/p/android-test-kit/source/browse/testapp/src/main/java/com/google/android/apps/common/testing/ui/testapp/LongListActivity.java'>LongListActivity</a>, which displays the String.length of the first row (to make this less abstract, you can imagine the G+ app that shows a list of comments and each comment has a +1 button next to it):<br>
<br>
<img src='http://wiki.android-test-kit.googlecode.com/git/item50.png' />

No problem. Just add an onChildView specification to your DataInteraction:<br>
<pre><code>onData(withItemContent("item: 60"))<br>
  .onChildView(withId(R.id.item_size))<br>
  .perform(click());<br>
</code></pre>
<b>Note:</b> This sample uses the "withItemContent" matcher from the sample above it!<br>
Take a look at <a href='https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/AdapterViewTest.java#61'>ApdaterViewTest#testClickOnSpecificChildOfRow60</a> here!<br>
<br>
<h2>Matching a view that is a footer/header in a ListView</h2>

Headers and footers are added to ListViews via the addHeaderView/addFooterView APIs. To load them using Espresso.onData, make sure to set the data object (second param) to a preset value. For example:<br>
<pre><code>public static final String FOOTER = "FOOTER";<br>
...<br>
View footerView = layoutInflater.inflate(R.layout.list_item, listView, false);<br>
((TextView) footerView.findViewById(R.id.item_content)).setText("count:");<br>
((TextView) footerView.findViewById(R.id.item_size)).setText(String.valueOf(data.size()));<br>
listView.addFooterView(footerView, FOOTER, true);<br>
</code></pre>

Then, you can write a matcher that matches this object:<br>
<pre><code>import static org.hamcrest.Matchers.allOf;<br>
import static org.hamcrest.Matchers.instanceOf;<br>
import static org.hamcrest.Matchers.is;<br>
<br>
@SuppressWarnings("unchecked")<br>
public static Matcher&lt;Object&gt; isFooter() {<br>
  return allOf(is(instanceOf(String.class)), is(LongListActivity.FOOTER));<br>
}<br>
</code></pre>

And loading the view in a test is trivial:<br>
<pre><code>import static com.google.android.apps.common.testing.ui.espresso.Espresso.onData;<br>
import static com.google.android.apps.common.testing.ui.espresso.action.ViewActions.click;<br>
import static com.google.android.apps.common.testing.ui.espresso.sample.LongListMatchers.isFooter;<br>
<br>
public void testClickFooter() {<br>
  onData(isFooter())<br>
    .perform(click());<br>
  ...<br>
}<br>
</code></pre>

Take a look at the full code sample at: <a href='https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/AdapterViewTest.java#90'>AdapterViewtest#testClickFooter</a>

<h2>Matching a view that is inside an ActionBar</h2>

The <a href='https://code.google.com/p/android-test-kit/source/browse/testapp/src/main/java/com/google/android/apps/common/testing/ui/testapp/ActionBarTestActivity.java'>ActionBarTestActivity</a> has two different action bars: a normal <a href='http://developer.android.com/guide/topics/ui/actionbar.html'>ActionBar</a> and a contextual action bar that is created from a <a href='http://developer.android.com/guide/topics/ui/menus.html#options-menu'>options menu</a>. Both action bars have one item that is always visible and two items that are only visible in overflow menu. When an item is clicked, it changes a TextView to the content of the clicked item.<br>
<br>
Matching visible icons on both of the action bars is easy:<br>
<pre><code>public void testClickActionBarItem() {<br>
  // We make sure the contextual action bar is hidden.<br>
  onView(withId(R.id.hide_contextual_action_bar))<br>
    .perform(click());<br>
<br>
  // Click on the icon - we can find it by the r.Id.<br>
  onView(withId(R.id.action_save))<br>
    .perform(click());<br>
<br>
  // Verify that we have really clicked on the icon by checking the TextView content.<br>
  onView(withId(R.id.text_action_bar_result))<br>
    .check(matches(withText("Save")));<br>
}<br>
</code></pre>
Screenshot:<br>
<br>
<img src='http://wiki.android-test-kit.googlecode.com/git/actionbar_normal_icon.png' />

The code looks identical for the contextual action bar:<br>
<pre><code>public void testClickActionModeItem() {<br>
  // Make sure we show the contextual action bar.<br>
  onView(withId(R.id.show_contextual_action_bar))<br>
    .perform(click());<br>
<br>
  // Click on the icon.<br>
  onView((withId(R.id.action_lock)))<br>
    .perform(click());<br>
<br>
  // Verify that we have really clicked on the icon by checking the TextView content.<br>
  onView(withId(R.id.text_action_bar_result))<br>
    .check(matches(withText("Lock")));<br>
}<br>
</code></pre>

Screenshot:<br>
<br>
<img src='http://wiki.android-test-kit.googlecode.com/git/actionbar_contextual_icon.png' />


Clicking on items in the overflow menu is a bit trickier for the normal action bar as some devices have a hardware overflow menu button (they will open the overflowing items in an options menu) and some devices have a software overflow menu button (they will open a normal overflow menu). Luckily, Espresso handles that for us.<br>
<br>
For the normal action bar:<br>
<pre><code>public void testActionBarOverflow() {<br>
  // Make sure we hide the contextual action bar.<br>
  onView(withId(R.id.hide_contextual_action_bar))<br>
    .perform(click());<br>
<br>
  // Open the overflow menu OR open the options menu,<br>
  // depending on if the device has a hardware or software overflow menu button.<br>
  openActionBarOverflowOrOptionsMenu(getInstrumentation().getTargetContext());<br>
<br>
  // Click the item.<br>
  onView(withText("World"))<br>
    .perform(click());<br>
<br>
  // Verify that we have really clicked on the icon by checking the TextView content.<br>
  onView(withId(R.id.text_action_bar_result))<br>
    .check(matches(withText("World")));<br>
}<br>
</code></pre>

This is how this looks on devices without a hardware overflow menu button:<br>
<br>
<img src='http://wiki.android-test-kit.googlecode.com/git/actionbar_normal_hidden_overflow.png' />

This is how this looks on devices with a hardware overflow menu button:<br>
<br>
<img src='http://wiki.android-test-kit.googlecode.com/git/actionbar_normal_hidden_no_overflow.png' />


For the contextual action bar it is really easy again:<br>
<pre><code>public void testActionModeOverflow() {<br>
  // Show the contextual action bar.<br>
  onView(withId(R.id.show_contextual_action_bar))<br>
    .perform(click());<br>
<br>
  // Open the overflow menu from contextual action mode.<br>
  openContextualActionModeOverflowMenu();<br>
<br>
  // Click on the item.<br>
  onView(withText("Key"))<br>
    .perform(click());<br>
<br>
  // Verify that we have really clicked on the icon by checking the TextView content.<br>
  onView(withId(R.id.text_action_bar_result))<br>
    .check(matches(withText("Key")));<br>
  }<br>
</code></pre>

<img src='http://wiki.android-test-kit.googlecode.com/git/actionbar_contextual_hidden.png' />


See the full code for these samples: <a href='https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/ActionBarTest.java'>ActionBarTest.java</a>

<h2>Matching and asserting views with text</h2>
Check out this blog post by Denys Zelenchuk: <a href='http://qathread.blogspot.com/2014/01/discovering-espresso-for-android.html'>http://qathread.blogspot.com/2014/01/discovering-espresso-for-android.html</a>

<h1>ViewAssertions</h1>
<h2>Asserting that a view is not displayed</h2>
After performing a series of actions, you will certainly want to assert the state of the UI under test. Sometimes, this may be a negative case (for example, something is not happening). Keep in mind that you can turn any hamcrest view matcher into a <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/ViewAssertion.java'>ViewAssertion</a> by using ViewAssertions.<a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/assertion/ViewAssertions.java#58'>matches</a>.<br>
<br>
In the example below, we take the isDisplayed matcher and reverse it using the standard "not" matcher:<br>
<pre><code>import static com.google.android.apps.common.testing.ui.espresso.Espresso.onView;<br>
import static com.google.android.apps.common.testing.ui.espresso.assertion.ViewAssertions.matches;<br>
import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.isDisplayed;<br>
import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.withId;<br>
import static org.hamcrest.Matchers.not;<br>
<br>
onView(withId(R.id.bottom_left))<br>
  .check(matches(not(isDisplayed())));<br>
</code></pre>
The above approach works if the view is still part of the hierarchy. If it is not, you will get a NoMatchingViewException and you need to use ViewAssertions.doesNotExist (see below).<br>
<br>
<h2>Asserting that a view is not present</h2>

If the view is gone from the view hierarchy (e.g. this may happen if an action caused a transition to another activity), you should use ViewAssertions.<a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/assertion/ViewAssertions.java#42'>doesNotExist</a>:<br>
<pre><code>import static com.google.android.apps.common.testing.ui.espresso.Espresso.onView;<br>
import static com.google.android.apps.common.testing.ui.espresso.assertion.ViewAssertions.doesNotExist;<br>
import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.withId;<br>
<br>
onView(withId(R.id.bottom_left))<br>
  .check(doesNotExist());<br>
</code></pre>

<h2>Asserting that a data item is not in an adapter</h2>

To prove a particular data item is not within an AdapterView you have to do things a little differently. We have to find the AdapterView we're interested in and interrogate the data its holding. We don't need to use onData(). Instead, we use onView to find the AdapterView and then use another matcher to work on the data inside the view.<br>
<br>
First the matcher:<br>
<pre><code>private static Matcher&lt;View&gt; withAdaptedData(final Matcher&lt;Object&gt; dataMatcher) {<br>
  return new TypeSafeMatcher&lt;View&gt;() {<br>
<br>
    @Override<br>
    public void describeTo(Description description) {<br>
      description.appendText("with class name: ");<br>
      dataMatcher.describeTo(description);<br>
    }<br>
<br>
    @Override<br>
    public boolean matchesSafely(View view) {<br>
      if (!(view instanceof AdapterView)) {<br>
        return false;<br>
      }<br>
      @SuppressWarnings("rawtypes")<br>
      Adapter adapter = ((AdapterView) view).getAdapter();<br>
      for (int i = 0; i &lt; adapter.getCount(); i++) {<br>
        if (dataMatcher.matches(adapter.getItem(i))) {<br>
          return true;<br>
        }<br>
      }<br>
      return false;<br>
    }<br>
  };<br>
}<br>
</code></pre>
Then the all we need is an onView that finds the AdapterView:<br>
<pre><code>@SuppressWarnings("unchecked")<br>
public void testDataItemNotInAdapter(){<br>
  onView(withId(R.id.list))<br>
      .check(matches(not(withAdaptedData(withItemContent("item: 168")))));<br>
  }<br>
</code></pre>

And we have an assertion that will fail if an item that is equal to "item: 168" exists in an adapter view with the id list.<br>
<br>
For the full sample look at <a href='https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/AdapterViewTest.java#99'>AdapterViewTest#testDataItemNotInAdapter</a>.<br>
<br>
<h1>Other</h1>

<h2>Using registerIdlingResource to synchronize with custom resources</h2>

The centerpiece of Espresso is its ability to seamlessly synchronize all test operations with the application under test. By default, Espresso waits for UI events in the current message queue to process and default AsyncTasks<code>*</code> to complete before it moves on to the next test operation. This should address the majority of application/test synchronization in your application.<br>
<br>
However, there are instances where applications perform background operations (such as communicating with web services) via non-standard means; for example: creation and management of threads directly and the use of custom services.<br>
<br>
In such cases, the first thing we suggest is that you don your testability hat and ask whether the user of non-standard background operations is warranted. In some cases, it may have happened due to poor understanding of Android and the application could benefit from refactoring (for example, by converting custom creation of threads to AsyncTasks). However, sometimes refactoring is not possible. The good news? Espresso can still synchronize test operations with your custom resources.<br>
<br>
Here's what you need to do:<br>
<br>
<ul><li>Implement the <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/IdlingResource.java'>IdlingResource</a> interface and expose it to your test.<br>
</li><li>Register one or more of your IdlingResource(s) with Espresso by calling <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Espresso.java#78'>Espresso.registerIdlingResource</a> in test setup.</li></ul>

To see how IdlingResource can be used take a look at the <a href='https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/AdvancedSynchronizationTest.java'>AdvancedSynchronizationTest</a> and the <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/contrib/CountingIdlingResource.java'>CountingIdlingResource</a> class.<br>
<br>
<code>*</code> Espresso synchronizes with the default AsyncTask thread pool. If your app uses another thread pool, treat the task as a custom resource and use the technique described above to register it with Espresso.<br>
<br>
<h2>Using a custom failure handler</h2>

Replacing the default FailureHandler of Espresso with a custom one allows for additional (or different) error handling - e.g. taking a screenshot or dumping extra debug information.<br>
<br>
<br>
The <a href='https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/CustomFailureHandlerTest.java'>CustomFailureHandlerTest</a> example demonstrates how to implement a custom failure handler:<br>
<pre><code>private static class CustomFailureHandler implements FailureHandler {<br>
  private final FailureHandler delegate;<br>
<br>
  public CustomFailureHandler(Context targetContext) {<br>
    delegate = new DefaultFailureHandler(targetContext);<br>
  }<br>
<br>
  @Override<br>
  public void handle(Throwable error, Matcher&lt;View&gt; viewMatcher) {<br>
    try {<br>
      delegate.handle(error, viewMatcher);<br>
    } catch (NoMatchingViewException e) {<br>
      throw new MySpecialException(e);<br>
    }<br>
  }<br>
}<br>
</code></pre>
This failure handler throws a MySpecialException instead of a NoMatchingViewException and delegates all other failures to the DefaultFailureHandler. The CustomFailureHandler can be registered with Espresso in the setUp() of the test:<br>
<pre><code>@Override<br>
public void setUp() throws Exception {<br>
  super.setUp();<br>
  getActivity();<br>
  setFailureHandler(new CustomFailureHandler(getInstrumentation().getTargetContext()));<br>
}<br>
</code></pre>

For more information see the <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/FailureHandler.java'>FailureHandler</a> interface and <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Espresso.java#89'>Espresso.setFailureHandler</a>.<br>
<br>
<h2>Using inRoot to target non-default windows</h2>
Surprising, but true - Android supports multiple <a href='http://developer.android.com/reference/android/view/Window.html'>windows</a>. Normally, this is transparent (pun intended) to the users and the app developer, yet in certain cases multiple windows are visible (e.g. an auto-complete window gets drawn over the main application window in the search widget). To simplify your life, by default Espresso uses a heuristic to guess which Window you intend to interact with. This heuristic is almost always "good enough"; however, in rare cases, you'll need to specify which window an interaction should target. You can do this by providing your own root window (aka <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/Root.java'>Root</a>) matcher:<br>
<br>
<pre><code>onView(withText("South China Sea"))<br>
  .inRoot(withDecorView(not(is(getActivity().getWindow().getDecorView()))))<br>
  .perform(click());<br>
</code></pre>

As is the case with <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/matcher/ViewMatchers.java'>ViewMatchers</a>, we provide a set of pre-canned <a href='https://code.google.com/p/android-test-kit/source/browse/espresso/lib/src/main/java/com/google/android/apps/common/testing/ui/espresso/matcher/RootMatchers.java'>RootMatchers</a>. Of course, you can always implement your own <code>Matcher&lt;Root&gt;</code>.<br>
<br>
Take a look at the full code sample <a href='https://code.google.com/p/android-test-kit/source/browse/testapp_test/src/main/java/com/google/android/apps/common/testing/ui/testapp/MultipleWindowTest.java'>here</a>.