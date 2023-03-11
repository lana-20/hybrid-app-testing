# Appium Hybrid App Testing

*Another major type of app is the hybrid app. Using Appium's Context API, you can switch back and forth between the native UI and any webviews in the app.*

Hybrid mobile apps are those that contain both native components as well as a webview. Here I have the iOS simulator up, with TheApp running on it. Everything we see here is native. Native UI elements, a native list, and so on. But if I tap on "Webview Demo", we get to a hybrid view. There are some native controls like text fields and buttons, but there's also a webview here. It may not look like it, but the text that says "Please navigate to a webpage" is actually a webpage itself! So let's type in the suggested page here, and hit Go. And we can see the Appium Pro website loaded! I can interact with it just like I would in Safari. In fact, Safari itself is a hybrid app, with some native controls, and a webview! So, this is what we mean by hybrid apps. The challenge for Appium testing is to figure out how to automate both the native layer as well as the web layer.

Fortunately, Appium can automate webviews the same way it automates web browsers. Webviews within mobile apps are built using all the same technology as the browsers themselves, so we have the same access to web content inside webviews as inside the browsers. For you as the test author, Appium exposes something called the Context API, which allows switching back and forth between the native layer as well as the web layer. We'll see examples of the Context API in a moment. But in general, everything we learned about automating web applications using Appium is relevant for automating hybrid apps. All the Chromedriver management stuff we talked about holds true for Android webviews as well. There is one important note for Android, however. By default, Android webviews are not debuggable, and are therefore not accessible to Chromedriver. If you run into this situation where Chromedriver can't find any automatable webviews, make sure to mention to the app developer that they can turn debugging on for all webviews in the app by running a certain command on the <code>WebView</code> class: <code>WebView.setWebContentsDebuggingEnabled(true);</code>. OK, on to the Context API!

There are 3 commands in the context API to learn about.

1. First, we have the <code>contexts</code> property available on the <code>driver</code> object. This property will return a list of all the contexts Appium can find within the current view. There will always be at least one context, called <code>NATIVE_APP</code>, because every app has at least a native context. In addition, there might be one or more other IDs in the list. These will be IDs of webviews Appium has found. You can't necessarily assume these IDs will be the same from test to test, which is why it's always important to find the ID by accessing this property, rather than assuming it's the same as it was before. In a typical case, there will be just 2 items in the list for a hybrid app: the native app ID, and then a single other ID, used to refer to the webview that's present. We can use that ID later on to switch into a webview.
2. Second, we have the <code>context</code> property, the singular version of the word, which will tell us the currently active context. The value of this property will be the ID of that active context, either <code>NATIVE_APP</code> or the ID of a webview if we're automating a webview at the moment. The purpose of this property is to be able to check what context we're in before performing actions that might only make sense in a given context.
3. Finally, we have the command that lets us switch into another context, <code>driver.switch_to.context()</code>. It takes a single parameter, which is the ID of a context that is present in the list returned by <code>driver.contexts</code>. This is important. Appium can only switch to an available context, so it's always good to check <code>driver.contexts</code> before attempting to switch, to make sure the context you have in mind is available. When you switch into a context, Appium will determine whether it is a native or a web context. If it's a native context, all your commands will take place in the native layer. If it's a web context, all your commands will take place in the webview. Depending on the context, you'll need to use different locator strategies, and have access to different commands. Just like you have access to certain locator strategies for web automation and others for mobile app automation, you'll need to make sure you use the appropriate strategies for hybrid apps depending on context. Webviews require web-based locator strategies!

That's pretty much all we need to know about the Context API from a theoretical perspective, so now let's dive into an example. What we want to do is get to the Webview Demo and then add a click at the end:

wait.until(EC.presence_of_element_located(
        (MobileBy.ACCESSIBILITY_ID, 'Webview Demo'))).click()
        
 






