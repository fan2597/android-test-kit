

# Why Espresso #

Use Espresso to write concise, beautiful, and reliable Android UI tests like this:

```

public void testSayHello() {
  onView(withId(R.id.name_field))
    .perform(typeText("Steve"));
  onView(withId(R.id.greet_button))
    .perform(click());
  onView(withText("Hello Steve!"))
    .check(matches(isDisplayed()));
}

```

The core API is small, predictable, and easy to learn and yet remains open for customization. Espresso tests state expectations, interactions, and assertions clearly without the distraction of boilerplate content, custom infrastructure, or messy implementation details getting in the way.

Espresso tests run optimally fast! Leave your waits, syncs, sleeps, and polls behind and let Espresso gracefully manipulate and assert on the application UI when it is at rest. Enjoy writing and executing your tests today - try a shot of Espresso!

To learn more, check out our 15-minute Google Test Automation Conference 2013 talk.

<a href='http://www.youtube.com/watch?feature=player_embedded&v=T7ugmCuNxDU' target='_blank'><img src='http://img.youtube.com/vi/T7ugmCuNxDU/0.jpg' width='425' height=344 /></a>

<br>
<h1>Target Audience</h1>
Espresso is targeted at developers, who believe that automated testing is an integral part of the development life-cycle. While it can be used for black-box testing, Espresso's full power is unlocked by those who are familiar with the codebase under test.<br>
<br>
<br>
<h1>Backward Compatibility</h1>
Espresso is supported on the following APIs:<br>
<br>
<table><thead><th><b>Codename</b></th><th><b>API</b></th></thead><tbody>
<tr><td>Froyo</td><td>8 </td></tr>
<tr><td>Gingerbread</td><td>10</td></tr>
<tr><td>Ice Cream Sandwich</td><td>15</td></tr>
<tr><td>Jelly Bean</td><td>16,17,18</td></tr>
<tr><td>KitKat</td><td>19</td></tr>
<tr><td>Lollipop</td><td>21</td></tr></tbody></table>

Notes:<br>
<ul><li>We use the <a href='http://developer.android.com/about/dashboards/index.html#Platform'>platform versions dashboard</a> to decide which APIs are supported. As the number of users on older API levels falls off, we will deprecate support for those API levels (Froyo is almost there).<br>
</li><li>Future versions of Android will be supported.</li></ul>

<br>
<h1>Getting Started</h1>
<h2>Prerequisites</h2>
<h3>Setup your test project</h3>
<ul><li>Instructions for <a href='http://developer.android.com/tools/testing/testing_eclipse.html'>Eclipse with ADT</a>.<br>
</li><li><a href='http://developer.android.com/sdk/installing/studio.html'>Instruction for Android Studio</a>
<h3>Setup your test environment</h3>
</li><li>To avoid flakiness, we highly recommend that you turn off system animations on the virtual or physical device(s) used for testing.<br>
</li><li>Under 'Settings->Developer options' disable the following 3 settings and restart the device:<br>
<img src='http://wiki.android-test-kit.googlecode.com/git/developer_settings.png'>
</li><li>It is also possible to do this <a href='https://code.google.com/p/android-test-kit/wiki/DisablingAnimations'>programmatically</a>.</li></ul>

<h2>Espresso Setup Instructions</h2>
<a href='https://code.google.com/p/android-test-kit/wiki/EspressoSetupInstructions'>See here</a>

<h2>Quick Start Guide</h2>
<ul><li><a href='EspressoStartGuide.md'>Get started</a> with writing basic Espresso tests.</li></ul>

<h2>Samples</h2>
<ul><li>Here is a <a href='EspressoSamples.md'>collection of samples</a> for more advanced test interactions. You are welcome to contribute more if you see something missing.<br>
</li><li>If you prefer reading code (we do), all samples are part of our test codebase <a href='https://code.google.com/p/android-test-kit/source/browse/#git%2Fespresso%2Flibtests%2Fsrc%2Fmain%2Fjava%2Fcom%2Fgoogle%2Fandroid%2Fapps%2Fcommon%2Ftesting%2Fui%2Fespresso%2Fsample'>here</a>.