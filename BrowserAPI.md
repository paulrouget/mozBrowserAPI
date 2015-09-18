Mozilla Browser API
===================

This document is an up-to-date and exhaustive description of the Mozilla Browser API.

The goal of this document is to eventually become a reference for implementations and documentation.

- [Summary](#Summary)
- [Usage](#Usage)
- [WebIDL](#WebIDL)
- [Methods](#Methods)
- [Events](#Events)

# <a name="Summary">Summary</a>

The HTML Browser API is an extension of the HTML `<iframe>` element that allows web apps to implement browsers or browser-like applications. This entails two major aspects:

- Make the iframe appear like a top-level browser window to the embedded content. This means that `window.top`, `window.parent`, `window.frameElement`, etc. should not reflect the frame hierarchy. Optionally, the notion that the embedded is an Open Web App can be expressed as well.
- An API to manipulate and listen for changes to the embedded content's state.

In addition to that, it's also possible to express the notion that the embedded content is an Open Web App. In that case the content is loaded within the appropriate app context (such as permissions).

# <a name="Usage">Usage</a>

An `<iframe>` is turned into a browser frame by setting the `mozbrowser` attribute:

```html
<iframe src="http://hostname.tld" mozbrowser>`
```

In order to embed an Open Web App, the `mozapp` attribute must also be supplied, with the path to the app's manifest:

```html
<iframe src="http://hostname.tld" mozapp='http://path/to/manifest.webapp' mozbrowser>
```

At last the content of the `<iframe>` can be loaded in its own child process, separate to the page embedding this frame by using the remote attribute.

```html
<iframe src="http://hostname.tld" mozbrowser remote>
```

## <a name="WebIDL">WebIDL</a>

``` webidl
callback BrowserElementNextPaintEventCallback = void ();

enum BrowserFindCaseSensitivity { "case-sensitive", "case-insensitive" };
enum BrowserFindDirection { "forward", "backward" };

dictionary BrowserElementDownloadOptions {
  DOMString? filename;
  DOMString? referrer;
};

dictionary BrowserElementExecuteScriptOptions {
  DOMString? url;
  DOMString? origin;
};

[NoInterfaceObject]
interface BrowserElement {
};

BrowserElement implements BrowserElementCommon;
BrowserElement implements BrowserElementPrivileged;

[NoInterfaceObject]
interface BrowserElementCommon {
  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser embed-widgets"]
  void setVisible(boolean visible);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser embed-widgets"]
  DOMRequest getVisible();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser embed-widgets"]
  void setActive(boolean active);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser embed-widgets"]
  boolean getActive();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser embed-widgets"]
  void addNextPaintListener(BrowserElementNextPaintEventCallback listener);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser embed-widgets"]
  void removeNextPaintListener(BrowserElementNextPaintEventCallback listener);
};

[NoInterfaceObject]
interface BrowserElementPrivileged {
  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  void sendMouseEvent(DOMString type,
                      unsigned long x,
                      unsigned long y,
                      unsigned long button,
                      unsigned long clickCount,
                      unsigned long modifiers);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   Func="TouchEvent::PrefEnabled",
   CheckAnyPermissions="browser"]
  void sendTouchEvent(DOMString type,
                      sequence<unsigned long> identifiers,
                      sequence<long> x,
                      sequence<long> y,
                      sequence<unsigned long> rx,
                      sequence<unsigned long> ry,
                      sequence<float> rotationAngles,
                      sequence<float> forces,
                      unsigned long count,
                      unsigned long modifiers);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  void goBack();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  void goForward();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  void reload(optional boolean hardReload = false);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  void stop();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  DOMRequest download(DOMString url,
                      optional BrowserElementDownloadOptions options);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  DOMRequest purgeHistory();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  DOMRequest getScreenshot([EnforceRange] unsigned long width,
                           [EnforceRange] unsigned long height,
                           optional DOMString mimeType="");

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  void zoom(float zoom);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  DOMRequest getCanGoBack();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  DOMRequest getCanGoForward();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  DOMRequest getContentDimensions();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAllPermissions="browser input-manage"]
  DOMRequest setInputMethodActive(boolean isActive);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAllPermissions="browser nfc-manager"]
  void setNFCFocus(boolean isFocus);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  void findAll(DOMString searchString, BrowserFindCaseSensitivity caseSensitivity);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  void findNext(BrowserFindDirection direction);

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAnyPermissions="browser"]
  void clearMatch();

  [Throws,
   Pref="dom.mozBrowserFramesEnabled",
   CheckAllPermissions="browser browser:universalxss"]
  DOMRequest executeScript(DOMString script,
                           optional BrowserElementExecuteScriptOptions options);

};
```

## <a name="DOMRequest">Note about `DOMRequest`</a>

The returned [DOMRequest](https://developer.mozilla.org/en-US/docs/Web/API/DOMRequest) objects can also be used as [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise).

For example:

``` javascript
var req = iframe.getCanGoBack();

req.onsuccess = function() {
  if (req.result) {
    console.log("It's possible to navigate the history backward.");
  } else {
    console.log("It's not possible to navigate the history backward.");
  }
}

req.onerror = function() {
  console.error(error.name);
}
```

Is equivalent to:

``` javascript
iframe.getCanGoBack().then(result => {
  if (result) {
    console.log("It's possible to navigate the history backward.");
  } else {
    console.log("It's not possible to navigate the history backward.");
  }
}, error => console.error(error.name));
```

# <a name="Methods">Methods</a>

## <a name="goBack">The `goBack` method</a>

Requires permissions: **browser**.

``` webidl
void goBack();
```

``` javascript
iframe.goBack();
```

The [goBack](#goBack) method is used to navigate backwards in the browser iframe's history. By calling this method, the browser iframe changes its location for the previous location available in its navigation history, which sends the according events: [mozbrowserlocationchange](#mozbrowserlocationchange), [mozbrowserloadstart](#mozbrowserloadstart) and so on.

## <a name="goForward">The `goForward` method</a>

Requires permissions: **browser**.

``` webidl
void goForward();
```

``` javascript
iframe.goForward();
```

The [goForward](#goForward) method is used to navigate forward in the browser iframe's history. By calling this method, the browser iframe changes its location for the next location available in its navigation history, which sends the according events: [mozbrowserlocationchange](#mozbrowserlocationchange), [mozbrowserloadstart](#mozbrowserloadstart) and so on. 

## <a name="getCanGoBack">The `getCanGoBack` method</a>

Requires permissions: **browser**.

``` webidl
DOMRequest getCanGoBack();
```

``` javascript
var req = iframe.getCanGoBack();
req.onsuccess = function() {
  if (req.result) {
    console.log("It's possible to navigate the history backward.");
  } else {
    console.log("It's not possible to navigate the history backward.");
  }
}
```

The [getCanGoBack](#getCanGoBack) method is used to know if it's possible to go back in the navigation history of the browser iframe. Returns a [DOMRequest](https://developer.mozilla.org/en-US/docs/Web/API/DOMRequest) object to handle the request's success or error. If the request is successful, the request's result will be a boolean indicating if the history can be navigated backward (true) or not (false).

## <a name="getCanGoForward">The `getCanGoForward` method</a>

Requires permissions: **browser**.

``` webidl
DOMRequest getCanGoForward();
```

``` javascript
var req = iframe.getCanGoForward();
req.onsuccess = function() {
  if (req.result) {
    console.log("It's possible to navigate the history forward.");
  } else {
    console.log("It's not possible to navigate the history forward.");
  }
}
```

The [getCanGoForward](#getCanGoForward) method is used to know if it's possible to go forward in the navigation history of the browser iframe. Returns a [DOMRequest](https://developer.mozilla.org/en-US/docs/Web/API/DOMRequest) object to handle the request's success or error. If the request is successful, the request's result will be a boolean indicating if the history can be navigated forward (true) or not (false).

## <a name="reload">The `reload` method</a>

Requires permissions: **browser**.

``` webidl
void reload(optional boolean hardReload = false);
```

``` javascript
iframe.reload();
```

The [reload](#reload) method is used to reload the content of the iframe.

## <a name="stop">The `stop` method</a>

Requires permissions: **browser**.

``` webidl
void stop();
```

``` javascript
iframe.stop();
```

The [stop](#stop) method is used to stop loading the content of the iframe.

## <a name="purgeHistory">the `purgeHistory` method</a>

Requires permissions: **browser**.

``` webidl
DOMRequest purgeHistory();
```

``` javascript
iframe.purgeHistory().then(_ => console.log("History cleared"));
```

The [purgeHistory](#purgeHistory) method is used to clear the browsing history associated with the browser iframe. It doesn't delete cookies or offline web content; just the history data. It purges the older documents from history. Documents can be removed from session history for various reasons. For example to control memory usage of the browser, to prevent users from loading documents from history, to erase evidence of prior page loads etc…

## <a name="zoom">the `zoom` method</a>

Requires permissions: **browser**.

``` webidl
void zoom(float zoom);
```

``` javascript
iframe.zoom(1.2);
```

Change the zoom factor of the browser iframe's content.

## <a name="getContentDimensions">the `getContentDimensions` method</a>

Requires permissions: **browser**.

``` webidl
DOMRequest getContentDimensions();
```

``` javascript
var req = iframe.getContentDimensions();
req.onsuccess = function() {
  console.log("page size:", req.result.width + "x" + req.result.height);
}
```

Get the size of the content window. Equivalent of `document.body.scrollWidth` and `document.body.scrollHeight`.

## <a name="findAll">the `findAll` method</a>

Requires permissions: **browser**.

``` webidl
void findAll(DOMString searchString, BrowserFindCaseSensitivity caseSensitivity);
```

``` javascript
iframe.findAll("Foo", "case-sensitive");
```

Search for a string in a page. The first instance of the string, relative to the caret position, will be highlighted. This will be followed by a [mozbrowserfindchange](#mozbrowserfindchange) event, carrying details about the search results. [findNext](#findNext) can be used to highlight the next or previous result.

## <a name="findNext">the `findNext` method</a>

Requires permissions: **browser**.

``` webidl
void findNext(BrowserFindDirection direction);
```

``` javascript
iframe.findNext("backward");
```

Highlight the next or previous search result. Followed by a [mozbrowserfindchange](#mozbrowserfindchange)  event.

## <a name="clearMatch">the `clearMatch` method</a>

Requires permissions: **browser**.

``` webidl
void clearMatch();
```

``` javascript
iframe.clearMatch();
```

Clear text highlighted via the [findAll](#findAll) and [findNext](#findNext) methods. Followed by a [mozbrowserfindchange](#mozbrowserfindchange) event.

## <a name="setVisible">the `setVisible` method</a>

Requires permissions: **browser** or **embed-widgets**.

``` webidl
void setVisible(boolean visible);
```

``` javascript
iframe.setVisible(true);
```

Gecko implementation note: equivalent to `nsIDocShell.isActive`.

The [setVisible](#setVisible) method is used to change the visibility state of the browser iframe.

The visible state of a browser iframe has nothing to do with its actual visibility (which is handled through CSS). The visible state is used to define the level of resources required by the browser iframe. If the visible state is set to true, it means that the browser iframe has a high priority on the resources needed to render and handle its content. On the contrary, if its visible state is set to false, the browser has a low priority over the resources it needs.

As an example, if the content of a browser iframe uses the `window.requestAnimationFrame` method and if the visible state is set to true, `window.requestAnimationFrame` will be called as often as necessary. However, if the visible state is set to false, `window.requestAnimationFrame` will be called only when there are free resources to do it.

Changing the visibility of the iframe will trigger a [mozbrowservisibilitychange](#mozbrowservisibilitychange) event.

## <a name="getVisible">the `getVisible` method</a>

Requires permissions: **browser** or **embed-widgets**.

``` webidl
DOMRequest getVisible();
```

``` webidl
var req = iframe.getVisible();
req.onsuccess = function() {
  if (req.result) {
    console.log("Iframe visibility is set to true");
  } else {
    console.log("Iframe visibility is set to false");
  }
}
```

The [getVisible](#getVisible) method is used to request the current visibility state of the browser iframe. See [setVisible](#setVisible).

## <a name="setActive">the `setActive` method</a>

Requires permissions: **browser** or **embed-widgets**.

``` webidl
void setActive(boolean active);
```

Gecko implementation note: equivalent to setting `nsIFrameLoader.visible`.

This helps the process manager prioritize processes.

## <a name="getActive">the `getActive` method</a>

Requires permissions: **browser** or **embed-widgets**.

``` webidl
boolean getActive();
```

See [setActive](#setActive).

## <a name="addNextPaintListener">the `addNextPaintListener` method</a>

Requires permissions: **browser** or **embed-widgets**.

``` webidl
void addNextPaintListener(BrowserElementNextPaintEventCallback listener);
```

``` javascript
function onNextPaint() {
  console.log("paint");
}
iframe.addNextPaintListener(onNextPaint);
```

The listener is called when content of the iframe is repainted on the screen. The event provides information about what has been repainted. It is mainly used to investigate performance optimization.

## <a name="removeNextPaintListener">the `removeNextPaintListener` method</a>

Requires permissions: **browser** or **embed-widgets**.

``` webidl
void removeNextPaintListener(BrowserElementNextPaintEventCallback listener);
```

``` javascript
iframe.removeNextPaintListener(onNextPaint);
```

Remove previously added paint listener. See [addNextPaintListener](#addNextPaintListener).

## <a name="sendMouseEvent">The `sendMouseEvent` method</a>

Requires permissions: **browser**.

``` webidl
void sendMouseEvent(DOMString type, unsigned long x, unsigned long y, unsigned long button, unsigned long clickCount, unsigned long modifiers);
```

``` javascript
iframe.sendMouseEvent("mousemove", x, y, 0, 0, 0)
iframe.sendMouseEvent("mousedown", x, y, 0, 1, 0)
iframe.sendMouseEvent("mouseup", x, y, 0, 1, 0)
```

The sendMouseEvent method allows to fake a mouse event and send it to the browser iframe's content.

### Parameters:

- type: A string representing the event type. Possible values are mousedown, mouseup, mousemove, mouseover, mouseout, or contextmenu.
- x: A number representing the x position of the cursor relative to the browser iframe's visible area in CSS pixels.
- y: A number representing the y position of the cursor relative to the browser iframe's visible area in CSS pixels.
- button: A number representing which button has been pressed on the mouse: 0 (Left button), 1 (middle button), or 2 (right button).
- clickCount: Number of clicks that have been performed.
- modifiers: A number representing a key pressed at the same time the mouse button was clicked:
  - 1 : The ALT key
  - 2 : The CONTROL key
  - 4 : The SHIFT key
  - 8 : The META key
  - 16 : The ALTGRAPH key
  - 32 : The CAPSLOCK key
  - 64 : The FN key
  - 128 : The NUMLOCK key
  - 256 : The SCROLL key
  - 512 : The SYMBOLLOCK key
  - 1024 : The WIN key

For example, to send WIN and ALT modifiers, use `1 | 1014`.

## <a name="sendTouchEvent">The `sendTouchEvent` method</a>

Requires permissions: **browser**.

``` webidl
void sendTouchEvent(DOMString type, sequence<unsigned long> identifiers, sequence<long> x, sequence<long> y, sequence<unsigned long> rx, sequence<unsigned long> ry, sequence<float> rotationAngles, sequence<float> forces, unsigned long count, unsigned long modifiers);
```

``` javascript
iframe.sendTouchEvent("touchstart", [1], [x], [y], [2], [2], [20], [0.5], 1, 0);
```

The sendTouchEvent method allows to fake a touch event and send it to the browser iframe's content.

Note: This method is available for touch enable devices only.

Parameters:

- type: A string representing the event type. Possible values are `touchstart`, `touchend`, `touchmove`, or `touchcancel`.
- x: An array of numbers representing the x position of each touch point relative to the browser iframe's visible area in CSS pixels.
- y: An array of numbers representing the y position of each touch point relative to the browser iframe's visible area in CSS pixels.
- rx: An array of numbers representing the x radius of each touch point in CSS pixels.
- ry: An array of numbers representing the y radius of each touch point in CSS pixels.
- rotationAngles: An array of numbers representing the angle of each touch point in degrees.
- forces: An array of numbers representing the intensity of each touch in the range [0,1].
- count: Number of touches that have been performed.
- modifiers: A number representing a key pressed at the same time the touches were performed:
  - 1 : The ALT key
  - 2 : The CONTROL key
  - 4 : The SHIFT key
  - 8 : The META key
  - 16 : The ALTGRAPH key
  - 32 : The CAPSLOCK key
  - 64 : The FN key
  - 128 : The NUMLOCK key
  - 256 : The SCROLL key
  - 512 : The SYMBOLLOCK key
  - 1024 : The WIN key

For example, to send WIN and ALT modifiers, use `1 | 1014`.

## <a name="download">The `download` method</a>

Requires permissions: **browser**.

``` webidl
DOMRequest download(DOMString url, optional BrowserElementDownloadOptions options);
```

``` javascript
var req = iframe.download(fooURL, { filename: 'foo.bin' });
req.onsuccess = function() {
  console.log("File downladed");
}
req.onerror = function() {
  console.log("Download error");
}
```

Download specified URL to specified filename.

## <a name="getScreenshot">The `getScreenshot` method</a>

Requires permissions: **browser**.

``` webidl
DOMRequest getScreenshot([EnforceRange] unsigned long maxWidth, [EnforceRange] unsigned long maxHeight, optional DOMString mimeType="");
```

``` javascript
var req = iframe.getScreenshot(100, 100);
req.onsuccess = function() {
  var blob = req.result;
  var url = URL.createObjectURL(blob);
}
```

The [getScreenshot](#getScreenshot) method lets you request a screenshot of an iframe, scaled to fit within a specified maximum width and height; the image may be cropped if necessary but will not be squished vertically or horizontally. MIME type specifying the format of the image to be returned; the default is `image/jpeg`. Use `image/png` to capture the alpha channel of the rendering result by returning a PNG-format image. This lets you get the transparent background of the iframe.


[getScreenshot](#getScreenshot) waits for the event loop to go idle before it takes the screenshot. It won't wait more than 2000ms (Gecko implementation note: delay defined by the `dom.browserElement.maxScreenshotDelayMS` preference).

## <a name="executeScript">The `executeScript` method</a>

Requires permissions: **browser** and **browser:universalxss**.

``` webidl
DOMRequest executeScript(DOMString script, optional BrowserElementExecuteScriptOptions options);
```

``` javascript
var req1 = iframe.executeScript(`
  var a = 3;
  a + 3
`, {url: 'http://example.com/index.html'});

req1.onsuccess = function() {
  console.log(req1.result); // 6
}

var req2 = iframe.executeScript(`
  new Promise((resolve, reject) => {
    setTimeout(function() {
      resolve(6);
    }, 1000})
  )
`, {origin: 'http://example.com'});

req2.onsuccess = function() {
  console.log(req2.result); // 6
}

```

If the script value is a not a [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise), it is simply return as the request value. If the script value is a Promise, the result of the request will be the Promise resolved value.

The `url` or `origin` option is required to ensure the script is being executed in the expected context.

## <a name="setInputMethodActive">The `setInputMethodActive` method</a>

Requires permissions: **browser** and **input-manager**.

``` webidl
DOMRequest setInputMethodActive(boolean isActive);
```

``` javascript
iframe.setInputMethodActive(true);
```

Calling [setInputMethodActive](#setInputMethodActive) on an iframe will set it as active IME window and other iframes as non-active IME windows. Useful when a top level app wants to activate a window (one at a time) as a IME (Input Method Editor, like a keyboard).

## <a name="setNFCFocus">The `setNFCFocus` method</a>

Requires permissions: **browser** and **nfc-manager**.

``` webidl
void setNFCFocus(boolean isFocus);
```

``` javascript
iframe.setNFCFocus(true);
```

Sets whether an iframe will receive NFC events.

# <a name="Events">Events</a>

Most of the events come with with a `details` property which carry extra information about the event.

## <a name="mozbrowserloadstart">The `mozbrowserloadstart` event</a>

Sent when the browser iframe starts to load a new page. This is usually when the embedder wants to start spinning a loading indicator.

## <a name="mozbrowserloadend">The `mozbrowserloadend` event</a>

Sent when the browser iframe has finished loading all its assets, or failed to load. This is usually when the embedder wants to stop spinning a loading indicator.

``` javascript
event.details = {
  backgroundColor: String
}
```

## <a name="mozbrowsertitlechange">The `mozbrowsertitlechange` event</a>

Sent when the top level window title changed.

``` javascript
  event.details = String;
```

## <a name="mozbrowservisibilitychange">The `mozbrowservisibilitychange` event</a>

Sent when the visibility state of the iframe changed. See also [setVisible](#setVisible).

``` javascript
  event.details = {
    visible: Boolean
  }
```

## <a name="mozbrowserfirstpaint">The `mozbrowserfirstpaint` event</a>

Sent when the iframe paints content for the first time. Doesn't include the initial paint from *about:blank*. See also [mozbrowserdocumentfirstpaint](#mozbrowserdocumentfirstpaint) and [addNextPaintListener](#addNextPaintListener).

## <a name="mozbrowserdocumentfirstpaint">The `mozbrowserdocumentfirstpaint` event</a>

Sent for every document (across navigation). See also [mozbrowserfindchange](#mozbrowserfindchange) and [addNextPaintListener](#addNextPaintListener).

## <a name="mozbrowserclose">The `mozbrowserclose` event</a>

Window is now closed. Embedder wants to destroy the iframe.

## <a name="mozbrowsercontextmenu">The `mozbrowsercontextmenu` event</a>

User right clicked or long-pressed on page. The event can hold several menu descriptions. For example, if the user clicked on an image nested in a `<a>` tag, 2 menus are available. One with information related to the image, one for the link. The page can also describe its own menus with the <menu> tag.

``` javascript
  event.details = {

    clientX: Number, // Click coordinates
    clientY: Number,

    // Regular menu. One per possible target
    systemTargets: Array(MenuSystem),

    // if dom node has a `contextmenu` attribute pointing to a menu element
    // https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/contextmenu

    contextmenu: Menu
    contextMenuItemSelected: function(id)

  }

  MenuSystem = {

    documentURI: String,

    // link:
    uri: String, // href
    text: String, // textContent

    // image:
    uri: String, // src

    // video:
    url: String, // src
    hasVideo: Boolean, // true if video has metadata and is bigger than 0x0

    // input in a <form>:
    action: String, // from form
    method: String, // from form
    name: String // from input

  }

  Menu: {

    type: "menu" | "menuitem",
    label: String // label attribute from the dom node

    // if menuitem
    icon: String,
    id: String, // used with contextMenuItemSelected()

    // if menu
    items: Array(Menu)

  }
```

## <a name="mozbrowsererror">The `mozbrowsererror` event</a>

The page has encountered an error while loading, or crashed.

``` javascript
  event.details = {
    type: String
  }
```

Possible errors:

- `"fatal"` (crash)
- `"unknownProtocolFound"`
- `"fileNotFound"`
- `"dnsNotFound"`
- `"connectionFailure"`
- `"netInterrupt"`
- `"netTimeout"`
- `"cspBlocked"`
- `"phishingBlocked"`
- `"malwareBlocked"`
- `"unwantedBlocked"`
- `"offline"`
- `"malformedURI"`
- `"redirectLoop"`
- `"unknownSocketType"`
- `"netReset"`
- `"notCached"`
- `"isprinting"`
- `"deniedPortAccess"`
- `"proxyResolveFailure"`
- `"proxyConnectFailure"`
- `"contentEncodingFailure"`
- `"remoteXUL"`
- `"unsafeContentType"`
- `"corruptedContentError"`
- `"certerror"`
- `"other"`

## <a name="mozbrowsersecuritychange">The `mozbrowsersecuritychange` event</a>

Sent when the SSL and mixed content states change

``` javascript
  event.details = {

    // "insecure" indicates that the data corresponding to
    //   the request was received over an insecure channel.
    //
    // "broken" indicates an unknown security state.  This
    //   may mean that the request is being loaded as part
    //   of a page in which some content was received over
    //   an insecure channel.
    //
    // "secure" indicates that the data corresponding to the
    //   request was received over a secure channel.

    state: "insecure" | "broken" | "secure",

    // "loaded_tracking_content": tracking content has been loaded.
    // "blocked_tracking_content": tracking content has been blocked from loading.

    trackingState: "loaded_tracking_content" | "blocked_tracking_content",

    // "blocked_mixed_active_content": Mixed active content has been blocked from loading.
    // "loaded_mixed_active_content": Mixed active content has been loaded.
    mixedState: "blocked_mixed_active_content" | "loaded_mixed_active_content",
    extendedValidation: Boolean,
    trackingContent: Boolean,
    mixedContent: Boolean,
  }
```

## <a name="mozbrowserlocationchange">The `mozbrowserlocationchange` event</a>

Sent when the location change. Happens every time navigation occurs. It's usually the right time to call [getCanGoBack](#getCanGoBack) and [getCanGoForward](#getCanGoForward).

``` javascript
  event.details = String; // new url
```

## <a name="mozbrowsericonchange">The `mozbrowsericonchange` event</a>

Sent when a new icon (`<link rel=icon|apple-touch-icon>`) is available

``` javascript
  event.details = {
    href: url,
    sizes: String, // optional. "16x16" or "16x16 32x32" or "any"
    rel: String, // optional. "apple-touch-icon" "shortcut icon", "icon"
  }
```

## <a name="mozbrowserscrollareachanged">The `mozbrowserscrollareachanged` event</a>

Sent when the available scrolling area changes. Happens on resize and when the page size changes (while loading for example).

``` javascript
  event.details =  {
    width: Number,
    height: Number,
  }
```

## <a name="mozbrowseropensearch">The `mozbrowseropensearch` event</a>

Sent when page encounter `<link rel=search type=application/opensearchdescription+xml>`.

``` javascript
  event.details = {
    title,
    href
  }
```

## <a name="mozbrowsermanifestchange">The `mozbrowsermanifestchange` event</a>

Sent when the manifest url changes `<link rel=manifest href=…>`.

``` 
  event.details = {
    href: String,
  }
```

## <a name="mozbrowsermetachange">The `mozbrowsermetachange` event</a>

Sent when a `<meta>` tag is added, removed or changed.

``` javascript
  event.details = {
    name: 'viewmode' | 'theme-color' | 'theme-group' | 'application-name',
    content: String // <meta content=…>
    type: 'added' | 'changed' | 'removed', // optional
    lang: String, // optional
  }
```

## <a name="mozbrowserresize">The `mozbrowserresize` event</a>

Window size has changed.

``` javascript
  event.details = {
    width: Number,
    height: Number,
  }
```

## <a name="mozbrowseractivitydone">The `mozbrowseractivitydone` event</a>

FIXME

``` javascript
  event.details = {
    success: Boolean
  }
```

## <a name="mozbrowserscroll">The `mozbrowserscroll` event</a>

Content has scrolled.

``` javascript
  event.details = {
    top: Number,
    left: Number,
  }
```

## <a name="mozbrowserasyncscroll">The `mozbrowserasyncscroll` event</a>

Content has scrolled (apzc version).

``` javascript
  event.details = {
    left: Number,
    top: Number,
    width: Number,
    height: Number,
    scrollWidth: Number,
    scrollHeight: Number,
  }
```

## <a name="mozbrowseropentab">The `mozbrowseropentab` event</a>

User *middle|ctrl|cmd* clicked on a link

``` javascript
  event.details = {
    url: String,
  }
```

## <a name="mozbrowseropenwindow">The `mozbrowseropenwindow` event</a>

A new window is required. Usually happens when the user clicked on link with a unknown target. The embedder must use the passed `frameElement` DOM node.

``` javascript
  event.details = {
    url: String,
    name: String,

    // See https://developer.mozilla.org/en-US/docs/Web/API/Window/open
    features: String,

    // The event provides an iframe.
    // For a window sharing the same domain of its parent for example,
    // it will use the same process

    frameElement: HTMLIFrameElement
  }
```

## <a name="mozbrowseraudioplaybackchange">The `mozbrowseraudioplaybackchange` event</a>

Sent when audio starts or stops playing

``` javascript
  event.details = Boolean; // playing or not
```

## <a name="mozbrowserusernameandpasswordrequired">The `mozbrowserusernameandpasswordrequired` event</a>

Authentification required. The embedder is supposed retrieve credentials, usually via a dialog or a database of username/passwords, and then call `authenticate()` or `cancel()`.

``` javascript
  event.details = {
    host: String,
    realm: String,
    isProxy: Boolean,

    // Embedder should call once of these functions
    authenticate: function("user", "password"),
    cancel: function(),
  }
```

## <a name="mozbrowsershowmodalprompt">The `mozbrowsershowmodalprompt` event</a>

Embedder should show a dialog.

``` javascript
  event.details = {

    promptType: "alert" | "confirm"
    title: String,
    message: String,

    // Embedder should set returnValue
    returnValue: undefined,

    // If the embedder calls preventDefault() on this event,
    // the iframe is blocked until unblock() is called.
    unblock: function(),

  }

  // or:

  event.details = {

    promptType: "custom-prompt",
    // FIXME More things! See BrowserElementPromptService.jsm:confirmEx implementation

  }
```

## <a name="mozbrowserselectionstatechanged">The `mozbrowserselectionstatechanged` event</a>

Selection changed.

**Deprecated**. See [mozbrowsercaretstatechanged](#mozbrowsercaretstatechanged).

``` javascript
  event.details = {

    rect: {
      // Contains bounding rectangle of selection
      width: Number,
      height: Number,
      top: Number,
      bottom: Number,
      left: Number,
      right: Number,
    },

    commands: {
      // Describe what commands can be executed in child. Include canSelectAll,
      // canCut, canCopy and canPaste. For example: if we want to check if cut
      // command is available, using following code, if (event.commands.canCut) {}.
      canSelectAll: Boolean,
      canCut: Boolean,
      canCopy: Boolean,
      canPaste: Boolean
    },

    zoomFactor: Number, // Current zoom factor in child frame.
    isCollapsed: Boolean, // Indicate current selection is collapsed or not.
    visible: Boolean, // Indicate selection visibility
    states: FIXME,
  }
```

## <a name="mozbrowsercaretstatechanged">The `mozbrowsercaretstatechanged` event</a>

Sent when the user select content in the page. Used by the embedder to show a context menu for clipboard actions.

``` javascript
  event.details = {

    rect: {
      // Contains bounding rectangle of selection
      width: Number,
      height: Number,
      top: Number,
      bottom: Number,
      left: Number,
      right: Number,
    },

    commands: {
      // Describe what commands can be executed in child.
      // For example: if we want to check if cut command is available, using following code,
      // if (event.details.commands.canCut) {event.details.sendDoCommandMsg('cut')}.
      canSelectAll: Boolean,
      canCut: Boolean,
      canCopy: Boolean,
      canPaste: Boolean
    },

    // The reason causes the state changed. Include "visibilitychange",
    // "updateposition", "longpressonemptycontent", "taponcaret", "presscaret",
    // "releasecaret".
    reason: String,

    zoomFactor: Number, // Current zoom factor in child frame.
    collapsed: Boolean, // Indicate current selection is collapsed or not.
    caretVisible: Boolean, // Indicate the caret visiibility.
    selectionVisible: Boolean, // Indicate current selection is visible or not.
    selectionEditable: Boolean, // Indicate current selection is editable or not.
    selectedTextContent: String, // Contains current selected text content.

    // Possible commands: cut, copy, paste, selectall
    sendDoCommandMsg: function(command),

  }
```

## <a name="mozbrowserscrollviewchange">The `mozbrowserscrollviewchange` event</a>

Sent when async scroll starts and stops.

FIXME: I think this only happens if caret is enabled

``` javascript
  event.details = {
    state: "started" | "stopped"
  }
```

## <a name="mozbrowserfindchange">The `mozbrowserfindchange` event</a>

Sent after [findAll](#findAll), [findNext](#findNext) and [clearMatch](#clearMatch) are called.

``` javascript
  event.details = {

    // Search is active.
    active: true,

    // The current searched string
    searchString: String

    // Number of maximum possible results (0 -> no limit)
    searchLimit: Number,

    // Highlighted result position
    activeMatchOrdinal: Number,

    // Number of result
    numberOfMatches: Number,
  }

  // or

  event.details = {
    // Search is not active.
    active: false
  }
```
