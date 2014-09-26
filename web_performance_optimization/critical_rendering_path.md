#Web Optimizing Performance

##Critical Rendering Path

Optimizing the critical rendering path is critical for improving performance of the pages: The developer goal is to prioritize and display the content that relates to the primary action of the user wants to take on a page.

Delivering a fast web experience requires a lot of work by the browser. Most of this work is hidden from the web developers: how the browser rendering the HTML, CSS, Javascript on the screen.

Optimizing for performance is about understanding all the happens in these intermediate step between receiving the HTML, CSS, and JavaScript bytes and the required processing to turn them into rendered pixels - that is the critical **rendering path**.

![](progressive-rendering.png)

By optimizing the critical rendering path we can significantly improve the time to first render of our pages. Further, understanding the critical rendering path will also serve as a foundation for building well performing interactive applications.
First, let’s take a quick, ground-up overview of how the browser goes about displaying a simple page.

###Constructing the Object Model

Before the browser can render the page it needs to construct the DOM and CSSOM trees. As a result, HTML and CSS need to be delivered to the browser as quickly as possible.

**TL;DR**
- Bytes -> characters -> tokens -> nodes -> object model,

- HTML markup is transformed into a Document Object Model (DOM), CSS markup is transformed into a CSS Object Model (CSSOM),

- DOM and CSSOM are independent data structures,

- Chrome DevTools Timeline allows us to capture and inspect the construction and processing costs of DOM and CSSOM.

####Document Object Model (DOM)

```html
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link href="style.css" rel="stylesheet">
    <title>Critical Path</title>
  </head>
  <body>
    <p>Hello <span>web performance</span> students!</p>
    <div><img src="awesome-photo.jpg"></div>
  </body>
</html>
```

To see how the browser process the rendering page, we consider the simplest case: a plain HTML page with some text and a single image

![Image of full-process](full-process.png)

1. **Conversion:** the browser reads the raw bytes of the HTML off the disk or network and translates them to individual characters based on specified encoding of the file (e.g. UTF-8).

2. **Tokenizing:** the browser converts strings of characters into distinct tokens specified by the W3C HTML5 standard - e.g. `<html>`, `<body>` and other strings within the "angle brackets". Each token has a special meaning and a set of rules.

3. **Lexing:** the emitted tokens are converted into "objects" which define their properties and rules.

4. **DOM construction:** Finally, because the HTML markup defines relationships between different tags (some tags are contained within tags) the created objects are linked in a tree data structure that also captures the parent-child relationships defined in the original markup: HTML object is a parent of the body object, the body is a parent of the paragraph object, and so on.

![](dom-tree.png)

**The final output of this entire process is the Document Object Model, or the "DOM" of a page, which the browser uses for all further processing of the page.**

Every time the browser has to process HTML markup it has to step through all of the steps above: convert bytes to characters, identify tokens, convert tokens to nodes, and build the DOM tree. This entire process can take some time, especially if we have a large amount of HTML to process.

The DOM tree captures the properties and relationships of the document markup, but it does not tell anything about how the element should look when rendered. That is the responsibility of the CSSOM.

####CSS Object Model (CSSOM)

In the previous simple page, while the browser was constructing the DOM, it will encountered a link tag in the head section of the document referencing an external CSS stylesheet call: `style.css`. The browser will dispatches a request for this resource, to get the following content:

```css
  body { font-size: 16px }
  p { font-weight: bold }
  span { color: red }
  p span { display: none }
  img { float: right }
```

Just as with HTML, CSS recieved need to be converted into something that the browser can understand and work with. And the process are very similiar to processing HTML:

![](cssom-construction.png)

The CSS bytes are converted into characters, then to tokens and nodes, and finally are linked into a tree structure known as the “CSS Object Model”, or CSSOM for short:

![](cssom-tree.png)

When computing the final set of styles for any object on the page, the browser starts with the most general rule applicabel to that node (Example: if the object is a child of body element, then all body styles apply) and then recursively refines the computed styles by applying more specific rules - i.e the rules "cascade down"

Consider the CSSOM tree above. Any text contained within the `span` tag placed within the body element will have a font size of 16 pixels and have red text - the `font-size` directive cascade down from body to the span. However, if a span tag is child of a paragraph `p` tag, then the content are not displayed.

The above CSSOM tree is the tree of the styles override in the example `style.css` file. It is not the complete CSSOM tree of the web page. Every browser provides a default set of styles also known as "user agent style". The css style which user create will override these default styles of the "agent".

To see how long the CSS processing took, using tool like Chrome DevTool to record a timeline and look for "Recaculate Style" event.

The CSSOM and the DOM are independent data structure. The browser need the render tree to links the CSSOM and DOM together to render a simple web page.

###Render-tree construction, Layout, and Paint

The CSSOM and DOM trees are combined into a render tree, which is then used to compute the layout of each visible element and serves as an input to the paint process which is the browser renders pixels to screen.

Optimizing each of these steps is critical to achieve optimal rendering performance.

**TL;DR**
- The DOM and CSSOM trees are combined to form the render tree,

- Render tree contains only the nodes required to render the page,

- Layout computes the exact position and size of each object,

- Paint is the last step that takes in the final render tree and renders the pixels to the screen

The first step is for the browser to combine the DOM and CSSOM into a "render tree" that captures all the visible DOM content on the page, plus all the CSSOM style information for each node.

![](render-tree-construction.png)

To construct the render tree, the browser roughly does the following:

1. Starting at the root of the DOM tree, traverse each visible node:
  - Some nodes are not visible (eg. script tags, meta tags...), and are ommited because they are not refelct in the rendered output,
  - Some nodes are hidden via CSS (`display: none;`) and are also omitted from the render tree.

2. For each visible node find the appropriate matching CSSOM rules and apply them.

3. Emit visible nodes with content and their computed styles.

**Note**

> `visibility: hidden` is different from `display: none`. The former makes the
> element invisible, but the element is still occupies space in the layout
> (i.e empty box), whereas the later removes the element entirely from the render
> tree.

The final output is a render that contains both the content and the style information of all the visible content on the screen. And with the render tree in place, now the browser are ready for the "layout" stage.

At this point, the browser has know which nodes should be visible and their computed styles, but it have not calculated their exact position and size within the _viewport_ of the device, which is the work from the "layout" stage, also known as "reflow"

To figure out the exact size and position of each object, the browser begins at the root of the render tree and traverses it to compute the geometry of each object on the page. Lets start with a simple example:

```html
    <html>
      <head>
        <meta name="viewport" content="width=device-width,initial-scale=1.0">
        <title>Critial Path: Hello world!</title>
      </head>
      <body>
        <div style="width: 50%">
          <div style="width: 50%">Hello world!</div>
        </div>
      </body>
    </html>
```

In this example, the body contains two nested div: first, the parent div which has the display width is 50% of the viewport width, and the second - child div - has the display width is 25% of the viewport width or 50% of its parent div width.

![](layout-viewport.png)

The output of the layout process is a "box model" which precisely captures the exact position and size of each element within the viewport; all of the relative measure (e.g 50%) are converted to absolute pixels positions on the screen.

Finally, now the browser know every thing about nodes attributes: visibility, computed styles and geometry. The browser then can process the final stage which will convert each node in the render tree with these information to actual pixels on the screen - this step is often referred to as "painting" or "rasterizing".


The time required to perform render tree construction, layout and paint will vary based on the size of the document, the applied styles, and of course, the device it is running on: the larger the document the more work the browser will have to do; the more complicated the styles are the more time will be consumed for painting also.

Once all is done, the page is finally visible inthe viewport.

**Recap**:

1. Process HTML markup and build the DOM tree,

2. Process CSS markup and build the CSSOM tree,

3. Combine the DOM and CSSOM into a render tree,

4. Run layout on the render tree to compute geometry of each node,

5. Paint the individual nodes to the screen.

If the DOM and CSSOM is modified (e.g by javascript), the browser would have to repeat the same process over again to figure out which pixels need to be re-rendered on the screen.

**Optimizing the critical rendering path is the process of minimizing the total amount of time spent in steps 1 through 5 in the above sequence**. Doing so enables us to render content to the screen as soon as possible and also to reduces the amount of time between screen updates after the initial render - i.e. achieve higher refresh rate for interactive content.

###Rendering block CSS

Render blocking resource is when the browser hold any other rendering process for the rendering process of the current resource. CSS is treated as one of a render blocking resource which mean the others rendering process might continue if the CSSDOM is constructed. So to improve performance, make sure to keep the CSS lean, deliver it as quickly as possible along with using `media types` and `media queries` to unblock rendering.

**TL;DR**

- CSS is treated as a render blocking resource by default.

- `Media types`, and `media queries` allow to _mark_ some _CSS resource_ as _non render blocking_

- All CSS resource (block or non block) are downloaded by the browser.

Because CSS and HTML are render blocking resources, the browser will block rendering until it retrive both the DOM and CSSOM. That explain when the network is slow and a webpage is load there is a blank page with reloading icon in the title.

Nowadays, there are a lot of devices with different screen size, or maybe the webpage os used to print, and the style for each of those are different. We need a mechanism for loading each of the CSS for each of the devices, instead of loading it altogether at once.

CSS `media types` and `media queries` can be used to separate the rendering CSS for each specific devices.

```css
<link href="style.css" rel="stylesheet">
<link href="print.css" rel="stylesheet" media="print">
<link href="other.css" rel="stylesheet" media="(min-width: 40em)">
```

A media query consists of a media type (like "print") and 0 or more expression for represent conditions of s particular devices.

Example: the first stylesheet declaration does not provide any media type or query -> it applied in all case. Which mean it always render blocking. The second one will only apply when the content is being printed, hence this stylesheet does not block the rendering of the page when it is first loaded. The third stylesheet provides a `media query` which is executed by the browser: if the projection device has match the condition (has width > 40em), the browser of the device will block rendering until the stylesheet is downloaded and processed, else is does not block rendered.

Media query can be used on many characteristics of the device like: display vs. print, screen orientation changes, resize event...

When declaring stylesheet assets, pay close attention to the media type and queries, as they will have big performance impact on the critical rendering path!

Other example:

```css
<link href="style.css"    rel="stylesheet" media="screen">
<link href="portrait.css" rel="stylesheet" media="orientation:portrait">
```

- The first declaration is render blocking and its the same with the declaration without media query because "screen" is the default type if the media is not specify.

- The second declaration has a dynamic media query which will be evaluated when the page is being loaded. Depending on the orientation of the device when the page is loaded, so it may (when the device in portrait mode) or may not (when the devce in other mode) be render blocking.

Because render blocking only refers to whether the browser will have to hold the initial rendering of the page on theresource, it is not about whether the resource is being downloaded or not. In both case, the CSS assets is still downloaded by the browser, but only those which match media query condition will be render blocking.


