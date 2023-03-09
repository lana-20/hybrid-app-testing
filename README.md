# Appium Hybrid App Testing

*Another major type of app is the hybrid app. Using Appium's Context API, you can switch back and forth between the native UI and any webviews in the app.*

Hybrid mobile apps are those that contain both native components as well as a webview. Here I have the iOS simulator up, with TheApp running on it. Everything we see here is native. Native UI elements, a native list, and so on. But if I tap on "Webview Demo", we get to a hybrid view. There are some native controls like text fields and buttons, but there's also a webview here. It may not look like it, but the text that says "Please navigate to a webpage" is actually a webpage itself! So let's type in the suggested page here, and hit Go. And we can see the Appium Pro website loaded! I can interact with it just like I would in Safari. In fact, Safari itself is a hybrid app, with some native controls, and a webview! So, this is what we mean by hybrid apps. The challenge for Appium testing is to figure out how to automate both the native layer as well as the web layer.





