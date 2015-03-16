Problem: Intermittent NoActivityResumedException
(https://code.google.com/p/android-test-kit/issues/detail?id=80)

Solution: This is likely caused by a system dialog that can pop up in case of ANRs or other issues unrelated to the app under test. The solution is to control your test environment to prevent unwanted system popups. Details are outlined in our GTAC 2014 presentation: https://www.youtube.com/watch?v=aHcmsK9jfGU#t=427


---


Problem: I get a 'java.lang.NullPointerException: Failed to get events for string...' when trying to use typeText(...) action.

Solution: typeText attempts to get the input key events for the given string from the system. If your current input method does not provide these keys, the method will fail. You can either enable the required input method on the test device or use replaceText to set the text directly in the input field.