# Appium Hybrid App Testing

*Another major type of app is the hybrid app. Using Appium's Context API, you can switch back and forth between the native UI and any webviews in the app.*

Hybrid mobile apps are those that contain both native components as well as a webview. Here I have the iOS simulator up, with TheApp running on it. Everything we see here is native. Native UI elements, a native list, and so on. 

<img src="https://user-images.githubusercontent.com/70295997/224513173-63441300-2926-490c-852a-9830e537d288.png" width=350><img src="https://user-images.githubusercontent.com/70295997/224513164-5d49e9a4-29ce-4e56-b538-5947375473f9.png" width=350>


But if I tap on "Webview Demo", we get to a hybrid view. There are some native controls like text fields and buttons, but there's also a webview here. It may not look like it, but the text that says "Please navigate to a webpage" is actually a webpage itself! 

<img src="https://user-images.githubusercontent.com/70295997/224513670-1bf1d4db-d899-4df6-8998-e57bae11a585.png" width=350><img src="https://user-images.githubusercontent.com/70295997/224513672-577fbf7c-e924-4750-9078-d9fe0bce548a.png" width=350>


So let's type in the suggested page here, and hit Go. And we can see the Appium Pro website loaded! I can interact with it just like I would in Safari. In fact, Safari itself is a hybrid app, with some native controls, and a webview! So, this is what we mean by hybrid apps. The challenge for Appium testing is to figure out how to automate both the native layer as well as the web layer.

<img width="300" src="https://user-images.githubusercontent.com/70295997/225201258-bba84a20-f0b7-4a50-b822-b869b5dc8fbf.png">
<img width="500" src="https://user-images.githubusercontent.com/70295997/225201443-df56b194-6f0b-4eb4-84dc-7b2145b032e5.png">
<img width="390" src="https://user-images.githubusercontent.com/70295997/225201288-edd5b2ba-9abb-429b-b4cf-6949a7f3797b.png">

Hybrid Automation Overview |
---- |
Appium automates webviews using the same technology as for automating mobile browsers. |
Appium exposes a Context API that allows you to select a native or web context for your commands to target. |
Everything we learned for automating web browsers is relevant for automating hybrid apps' webviews. |
Android webviews must be debuggable if they are to be automatable by ChromeDriver. Android app developers can turn debugging on for all WebViews easily: <code>WebView.setWebContentsDebuggingEnabled(true);</code> |


Fortunately, Appium can automate webviews the same way it automates web browsers. Webviews within mobile apps are built using all the same technology as the browsers themselves, so we have the same access to web content inside webviews as inside the browsers. For you as the test author, Appium exposes something called the Context API, which allows switching back and forth between the native layer as well as the web layer. We'll see examples of the Context API in a moment. But in general, everything we learned about automating web applications using Appium is relevant for automating hybrid apps. All the Chromedriver management stuff we talked about holds true for Android webviews as well. There is one important note for Android, however. By default, Android webviews are not debuggable, and are therefore not accessible to Chromedriver. If you run into this situation where Chromedriver can't find any automatable webviews, make sure to mention to the app developer that they can turn debugging on for all webviews in the app by running a certain command on the <code>WebView</code> class: <code>WebView.setWebContentsDebuggingEnabled(true);</code>. This, of course, would be in a Java or Kotlin code that defines the Android app. OK, on to the Context API!

| The Context API |  |
| ---- | ---- |
| <code>ctxs = driver.contexts</code> | Returns a list of available contexts. "NATIVE_APP" plus any webview context IDs. |
| <code>ctx = driver.context</code> | Returns the currently-active context. |
| <code>driver.switch_to.context(ctx)</code> | Switches to one of the available contexts. |

There are 3 commands in the context API to learn about.

1. First, we have the <code>contexts</code> property available on the <code>driver</code> object. This property will return a list of all the contexts Appium can find within the current view. There will always be at least one context, called <code>NATIVE_APP</code>, because every app has at least a native context. In addition, there might be one or more other IDs in the list. These will be IDs of webviews Appium has found. You can't necessarily assume these IDs will be the same from test to test, which is why it's always important to find the ID by accessing this property, rather than assuming it's the same as it was before. In a typical case, there will be just 2 items in the list for a hybrid app: the native app ID, and then a single other ID, used to refer to the webview that's present. We can use that ID later on to switch into a webview.
2. Second, we have the <code>context</code> property, the singular version of the word, which will tell us the currently active context. The value of this property will be the ID of that active context, either <code>NATIVE_APP</code> or the ID of a webview if we're automating a webview at the moment. The purpose of this property is to be able to check what context we're in before performing actions that might only make sense in a given context.
3. Finally, we have the command that lets us switch into another context, <code>driver.switch_to.context()</code>. It takes a single parameter, which is the ID of a context that is present in the list returned by <code>driver.contexts</code>. This is important. Appium can only switch to an available context, so it's always good to check <code>driver.contexts</code> before attempting to switch, to make sure the context you have in mind is available. When you switch into a context, Appium will determine whether it is a native or a web context. If it's a native context, all your commands will take place in the native layer. If it's a web context, all your commands will take place in the webview. Depending on the context, you'll need to use different locator strategies, and have access to different commands. Just like you have access to certain locator strategies for web automation and others for mobile app automation, you'll need to make sure you use the appropriate strategies for hybrid apps depending on context. Webviews require web-based locator strategies!

That's pretty much all we need to know about the Context API from a theoretical perspective, so now let's dive into an [example](https://github.com/lana-20/hybrid-app-testing/blob/main/hybrid_ios.py). What we want to do is get to the Webview Demo and then add a click at the end:

        wait.until(EC.presence_of_element_located(
                (MobileBy.ACCESSIBILITY_ID, 'Webview Demo'))).click()
        
Once we're there, what we want to do is type in a certain URL in the input field. I happen to know a good locator for this field, so we can find it and enter a URL straightaway:

        wait.until(EC.presence_of_element_located(
                (MobileBy.XPATH, '//XCUIElementTypeTextField[@name="urlInput"]')
            )).send_keys('https://appiumpro.com')

Now, there's a 'Go' button we need to tap in order to actually load the page, which we can find using an accessibility ID of 'navigateBtn':

    driver.find_element(MobileBy.ACCESSIBILITY_ID, 'navigateBtn').click()
    
At this point the webview should start loading. So how should we activate it? There's a bit of complexity at this point, because if we try to find the webview immediately, it might not yet be loaded, and it might not show up in our context list until a few seconds have passed. So we're in our classic race condition position. We could fix this by adding a static wait for several seconds to be sure the webview has loaded up, but we want to do better than a static wait.

Instead, I'm going to show you how we can create a custom expected condition to handle this scenario. We already have webdriverwaits and their <code>.until()</code> method, so why not put them to good use in waiting for a webview to be present and to activate for us?

The way we do this is by creating a new Python class, which I'll do up here a ways. I'll call it <code>webview_active</code>, so that our whole wait line will read "wait until webview active", which is pretty transparent and understandable in my opinion:

        class webview_active(object):   # wait.until(web_active())
            pass

So now we have an empty class. How do we fill it out? By convention, to create a custom expected condition, we need to fill out a Python magic method for this class, called <code>__call__</code> with two underscores on each side. This is the method which will be executed if an instance of the class is called as if it's a function:

        def __call__(self, driver):
                pass

This <code>__call__</code> method takes a <code>driver</code> object which we can use to implement our logic. Now this method needs to return a value which can be evaluated to either True or False. If it returns a value which is *truthy*, meaning not False or None, the webdriver wait method will consider that we have succeeded in our logic and will stop its retries. But if we return a value which is *falsey* like False or None, then the webdriverwait until method will run our method again, and will keep doing so until the wait times out. So what we want to do is try to find a web context, switch to it, and return true. If we can't find a web context, then we'll return false.

So what we want to first do is loop through our available contexts, checking to see if we find a context which isn't NATIVE_APP:

        for context in driver.contexts:
                    if context != 'NATIVE_APP':

In this case, we switch to the context and return True, which will also have the effect of short-circuiting any additional trips through the loop:

        driver.switch_to.context(context)
        return True

Now, outside the loop, if we get here, it means we didn't find any webview contexts, so here we can simply <code>return False</code>.

Now that we have our custom expected condition coded up, let's actually use it. Immediately after we tap the <code>navigateBtn</code>, let's form a new wait with our custom expected condition:

        wait.until(webview_active())

That's it! The webdriverwait logic will actually handle the retry for us, up to the 10 seconds we have defined for this wait object. Now, if we make it past this line, we know that we're inside a webview context, so we can do things that are unique to websites, like get the url or the title. Let's just get those properties and print them out:

        print(driver.current_url)
        print(driver.title)

Note: here is the entirety of the webview class for transcript readers:

        class webview_active(object):

            def __call__(self, driver):
                for context in driver.contexts:
                    if context != 'NATIVE_APP':
                        driver.switch_to.context(context)
                        return True

That's all we want to do for now. Let's see if the code we wrote works. I've got the Appium server up and running and my simulator loaded, so let's run the script. Observe the app up and running, and we're navigating to the webview demo. Our URL comes up, and the page is loading. It takes a few seconds to pop up, during which our webdriverwait and custom expected condition are doing their thing. When the test is over, we can check the output in the terminal. Indeed it has the URL and title that we expect. 

        https://appiumpro.com
        Appium Pro

So this is how you work with hybrid apps with Appium. Android is entirely the same, so I won't bother doing a separate demo for it.

There is one additional technique which is useful to know, however, and that is how to inspect webviews. If you tried to inspect a mobile app with a webview from within Appium desktop, it'd be a bit problematic, because by default all you'd see are the native elements. Typically, both iOS and Android decide to promote some web elements to show up as native elements for accessibility purposes, and this can be useful, you might be able to find these elements using Appium Desktop in native mode. But in general, you can't use Appium desktop to actually read any of the HTML code within a webview, or to use all the web-specific locator strategies and actions. But never fear, because we can actually inspect webview content just the same way we inspect other websites in Safari and Chrome on desktop. Let's see how it's done with Safari first. 

<img width="500" src="https://user-images.githubusercontent.com/70295997/225201867-808aecee-5512-4c96-aacc-5d1072d55de5.png">

<img width="1000" src="https://user-images.githubusercontent.com/70295997/224517132-30f3a75d-468f-4c89-b5f7-7a02b985a3ce.png">
<img width="1000" src="https://user-images.githubusercontent.com/70295997/224517181-c77078a1-e3f9-40aa-b2e3-9a9f173f8920.png">

The first thing I'll do is get back to my webview in the simulator, opening up TheApp, heading to the webview demo, and then typing in the website to go to. There, it's loaded. Now, I need to open up Safari on my desktop. Once it's open, I can go up to the "Develop" menu. In this menu I now see an entry for the simulator I'm running! If I open up that submenu, I'll see a list of any of the pages that it found that I can inspect. I see the Appium Pro website, so let's inspect that one. And up pops a whole developer tools panel! Here I can navigate through the HTML hierarchy, and even see parts of the webview highlight when I mouse over them. So I can use this technique to figure out locator strategies and selectors for webviews, just like I would with determining them for Selenium with desktop browsers.

<img width="1000" src="https://user-images.githubusercontent.com/70295997/224518684-9fa5f9d4-9093-4ffc-bb56-cab1d6f00e44.png">

<img width="550" src="https://user-images.githubusercontent.com/70295997/225201903-013cf0d8-4751-429b-985e-0724b6ecc20f.png">

It's time to discuss Chrome. I'll close down Safari and the Simulator, and show TheApp running on an emulator. And I'll navigate to the webview demo just like before, and type in the name of the website to visit. Alright, now the Appium Pro website is loaded. At this point I can open up Chrome on desktop. Now that it's open, I can type in a special URL, [chrome://inspect](chrome://inspect). This will open up an inspection interface that lists all the pages and devices available for inspection. Under Remote Target, you should be able to see your emulator that has a WebView open in TheApp. You can see it even mentions the Appium Pro website. Now we can click the little 'inspect' link, and up will pop a full-on devtools interface. As you can see it's quite comprehensive. I have a view of the page on the left, and I can even use the inspection tool to find elements and navigate directly to them in the HTML. As with Safari, all the typical developer tools are available, so I can use this strategy to quickly and easily figure out locator strategies and selectors for elements.

Of course, this strategy works for mobile web apps and not just mobile hybrid apps, so you can use it any time you need to inspect any web content that's loaded up in a mobile device.





