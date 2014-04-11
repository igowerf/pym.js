# responsiveiframe

A simplified NPR Visuals fork of [NPR Tech's responsiveiframe](http://npr.github.com/responsiveiframe/). Requires jQuery on both the parent and child pages.

Released under the MIT open source license. See `LICENSE` for details.

## What is it?

A library that allows iframes to be embedded in a way that allows them to resize responsively within their parent and bypasses the usual cross-domain issues.

The typical use-case for this is embedding small custom bits of code (charts, maps, etc.) inside a CMS without override CSS or JavaScript conflicts.

See an example of this in action on [NPR.org](http://www.npr.org/2014/03/25/293870089/maze-of-college-costs-and-aid-programs-trap-some-families).

## Examples

Make your browser window wider or narrower to see test responsiveness in these examples:

* [Simple](examples/simple/)
* [Dynamic](examples/dynamic/)
* [HTML table](examples/table/)
* [D3 line graph](examples/graphic/)
* [Quiz](examples/quiz/)
* [Multiple D3 line graphs](examples/multiple/)

## Usage

### On the parent page

* Name your containing div with a unique ID name, and include a `data-iframe-target` attribute with the path to your `child.html` file.
* Include `responsiveIframe.js`. **Only once per page.**
* Call `window.responsiveParent();` to set up all responsive iframes on the page. **Only once per page.**
* You can optionally pass in a regex to filter incoming message domains: `$('iframe').responsiveIframe({ xdomain: '*\.npr\.org' });`.

For example:
```
<div id="example" data-iframe-target="PATH-TO-THIS-FOLDER/child.html"></div>
<script type="text/javascript" src="PATH-TO-THIS-FOLDER/js/responsiveIframe.js"></script>
<script>window.responsiveParent();</script>
```

(Replace "<strong>PATH-TO-THIS-FOLDER</strong>" with the actual published path to your files.)


### On the child (embedded) page

* Include `child.js`.
* Invoke `setupResponsiveChild();`.
* If the contents of your iframe are dynamic you will want to pass in a rendering function, like this: `setupResponsiveChild({ renderCallback: myFunc });` This function will be called once on load and then again anytime the window is resized.
* You can optionally pass in a number of milliseconds to enable automaticaly updating the height at that rate (in addition to on load and resize events). Like this `setupResponsiveChild({ polling: 500 });`.

#### Basic example

In our [simplest example](examples/table/), we have an HTML table that changes format (via CSS media queries). The height of the iframe adjusts to the height of the content onload and onresize.

```
<script src="../../src/responsiveIframe.js" type="text/javascript"></script>
<script>window.responsiveChild();</script>
```

#### Call a function when the window resizes

This might be useful in cases where sections of your child page need to be redrawn based on the new width. ([For example, a graphic generated by D3](examples/graphic/), which would not stretch or reflow on its own.)

```
function drawGraphic(width) {
...
}
window.responsiveChild({renderCallback: drawGraphic});
```

#### Manual resize events

If you have dynamic content and need finer control over resize events, you can invoke `sendHeightToParent()` in the child window at any time to force the iframe to update its size.

[For example, say you have a quiz](example/quiz/), and the content of the page changes when someone selects and answer, affecting the page's height:

```
function check_answer(e) {
    // highlight the correct answer
    window.responsiveChild.sendHeightToParent();
}
$('.question').find('li').on('click', check_answer);
window.responsiveChild();
```

## Assumptions / requirements

* If you're pasting the iframe embed code into a CMS or blogging software, make sure that it allows you to embed HTML and JavaScript. Some CMSes (such as WordPress) may strip it out.

* The parent and child pages do not necessarily need to be on the same domain. (You can restrict this with xdomain.)

* The responsive iframe library can stand alone and does not require jQuery. (Some of the examples use jQuery for unrelated features.)

### Browsers

This has been tested in:

* Internet Explorer 9 and 10 (Windows 7)
* Chrome 32 (Mac 10.9)
* Firefox 26 (Mac 10.9)
* Safari 7 (Mac 10.9)
* iOS 7 Safari
* Android 4.4 Chrome

Internet Explorer versions earlier than **9** are not supported.


## How it works

The `parent.js` library and a small bit of javascript are injected onto the parent page. This
code writes an iframe to the page in a container of your choice. The request for the iframe's contents includes querystring parameters for the `initialWidth` and `childId` of the child page. The `initialWidth` allows the child to know its size immediately on load. (In iOS the child frame can not determine its own width accurately.) The `childId` allows multiple children to be embedded on the same page, each with its own communication to the parent.

The child page includes `child.js` and its own javascript. It invokes the `setupResponsiveIframe` function, which initializes cross-iframe communication to the parent, renders the any dynamic contents and then sends the computed height of the child to the parent via [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage). Upon receiving this message the parent resizes the containing iframe, thus ensuring the contents of the child are always visible.

The parent page also registers for resize events. Any time one is received, the parent sends the new container width to each child via `postMessage`. The child rerenders its content and sends back the new height.


## Credits

Rewritten by [@nprapps](http://github.com/nprapps).

Originally built by [@NPR](http://github.com/npr/).

Based on an original prototype by [Ioseb Dzmanashvili](https://github.com/ioseb).
