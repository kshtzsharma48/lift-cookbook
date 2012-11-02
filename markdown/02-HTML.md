HTML
====




Testing and debugging CSS selectors
===================================

Problem
-------

You want to explore or debug CSS selectors interactively.

Solution
--------

Use the Scala REPL to run your CSS selector:

```scala
> console             
[info] Starting scala interpreter...
[info] 
Welcome to Scala version 2.9.1.final 
Type in expressions to have them evaluated.
Type :help for more information.

scala> import net.liftweb.util.Helpers._ 
import net.liftweb.util.Helpers._

scala> val in = <a>click me</a>
in: scala.xml.Elem = <a>click me</a>

scala> val f = "a [href]" #> "http://example.org"
f: net.liftweb.util.CssSel = 
  (Full(a [href]), Full(ElemSelector(a,Full(AttrSubNode(href)))))

scala> f(in)
res0: scala.xml.NodeSeq = 
  NodeSeq(<a href="http://example.org">click me</a>)
```

Discussion
----------

The CSS selector functionality in Lift gives you a `CssSel` function which is `NodeSeq => NodeSeq`.  Knowing this, you can construct an input `NodeSeq` (called `in` above), create your CSS function (called `f`) and then apply it to get the `NodeSeq` output (called `res0`, above). 

See Also
--------

* [CSS Selector Transforms](http://simply.liftweb.net/index-7.10.html#toc-Section-7.10) in _Simply Lift_.
* Section 6.1.2 of _Lift in Action_.


Sequencing CSS selector operations 
==================================

Problem
-------

You want your CSS selector binding to apply to the results of earlier binding expressions.

Solution
--------

Use `andThen` rather than `&` to compose your selector expressions.
For example, suppose we want to replace `<div id="foo"/>` with `<div id="bar">bar content</div>` but for some reason we needed to generate the `bar` div as a separate step in the selector expression:

```scala
sbt> console
[info] Starting scala interpreter...
[info] 
Welcome to Scala version 2.9.1.final (Java 1.7.0_05).
Type in expressions to have them evaluated.
Type :help for more information.

scala> import net.liftweb.util.Helpers._
import net.liftweb.util.Helpers._

scala> def render = "#foo" #> <div id="bar"/> andThen "#bar *" #> "bar content"
render: scala.xml.NodeSeq => scala.xml.NodeSeq

scala> render(<div id="foo"/>)
res0: scala.xml.NodeSeq = NodeSeq(<div id="bar">bar content</div>)
```

Discussion
----------

When using `&` think of the CSS selectors as always applying to the original template, no matter what other expressions you are combining.

Compare the example above with the change of replacing `andThen` with `&`:

```scala
scala> def render = "#foo" #> <div id="bar" /> & "#bar *" #> "bar content"
render: net.liftweb.util.CssSel

scala> render(<div id="foo"/>)
res1: scala.xml.NodeSeq = NodeSeq(<div id="bar"></div>)           
```

The second expression will not match as it is applied to the original input of `<div id="foo"/>`.


See Also
--------

* Mailing list discussion on: [CSS Selector in render on element ID created in the same render](https://groups.google.com/forum/?fromgroups#!topic/liftweb/fz3Pmlhzhfg).
* [CSS Selector Transforms](http://simply.liftweb.net/index-7.10.html#toc-Section-7.10) in _Simply Lift_.
* Wiki page on [Binding via CSS Selectors](http://www.assembla.com/spaces/liftweb/wiki/Binding_via_CSS_Selectors).


Setting meta tag contents
==========================

Problem
-------

You want to set the content of an HTML meta tag using Lift.

Solution
--------

Use the `@` CSS binding name selector.  For example, given:

```scala
<meta name="keywords" content="some words go here" />
```

The following snippet code will update the value of the contents attribute:

```scala
"@keywords [content]" #> "other words to use" 
```


Discussion
----------

The `@` selector selects all elements with the given name.  The `[content]` part matches on the attribute of `content`.  These are general mechanisms, not specific to meta tags.

See Also
--------

* [CSS Selector Transforms](http://simply.liftweb.net/index-7.10.html#toc-Section-7.10) in _Simply Lift_.


Setting the page title
=======================

Problem
-------

You want to set the `<title>` of the page from a Lift snippet.

Solution
--------

Select all the elements of the `title` element and replace them with the text you want:

```scala
"title *" #> "I am different"
```

Assuming you have a `<title>` tag in your template, the above will result in:

```scala
<title>I am different</title>
```

Discussion
----------

It is also possible to set the page title from the contents of `SiteMap, meaning the title used will be the title you've assigned to the page in the site map:

```scala
<title class="lift:Menu.title"></title>
```

The `lift:Menu.title` code prepends to any existing text in the title.  This means the following will have the phrase "Site Title - " in the title followed by the page title:

```scala
<title class="lift:Menu.title">Site Title - </title>
```

If you need more control, you can of course bind on title using a regular snippet. This example uses a custom snippet to put the site title after the page title:

```scala
<title class="lift:MyTitle"></title>

object MyTitle {
  def render = <title><lift:Menu.title /> - Site Title</title>
}
```


See Also
--------

* [CSS Selector Transforms](http://simply.liftweb.net/index-7.10.html#toc-Section-7.10) in _Simply Lift_.
* [Wiki page for SiteMap](http://www.assembla.com/spaces/liftweb/wiki/SiteMap)
* [Using <lift:Menu>](http://exploring.liftweb.net/master/index-7.html#toc-Subsection-7.2.3) in _Exploring Lift_.
* Mailing list discussion of [dynamic titles on sitemap](http://groups.google.com/group/liftweb/browse_thread/thread/e19bd2dda2b3159d).


Including HTML5 Shiv 
====================

Problem
-------

You want to include HTML5 Shiv (a.k.a. HTML5 Shim) so you can use HTML5 elements with legacy IE browsers.

Solution
--------

Put the markup in a snippet and include the snippet in your page or template.

```scala
package code.snippet

import scala.xml.Unparsed

object Html5Shiv {        
  def render = Unparsed("""<!--[if lt IE 9]>
  <script src="http://html5shim.googlecode.com/svn/trunk/html5.js">
  </script><![endif]-->""")
}
```

Reference the snippet in the `<head>` of your `templates-hidden/default.html`, e.g.,:

```html
<script class="lift:Html5Shiv"></script>
```

Discussion
----------

The HTML5 parser used by Lift does not carry comments from the source through to the rendered page.  If you're looking at `Unparsed` and worried, your instincts are correct, but in this case we do want unparsed XML content (the comment tag) included in the output.

See Also
--------

* [How to incorporate html5shiv](https://groups.google.com/forum/?fromgroups#!topic/liftweb/kLzcJwfIqHQ), from the mailing list.
* [html5shim Google code page](http://code.google.com/p/html5shim/).

Adding a Google +1 button
==========================

Problem
-------

You want to include a Google +1 button on a page.

Solution
--------

Put the markup into a snippet and invoke the snippet. For example:

```scala
object PlusOne {

 import net.liftweb.http.js.JsCmds.{Script,Run}

 def render = Script(Run("""(function() {
   var po = document.createElement('script'); 
   po.type = 'text/javascript'; po.async = true;
   po.src = 'https://apis.google.com/js/plusone.js';
   var s = document.getElementsByTagName('script')[0]; 
   s.parentNode.insertBefore(po, s);
 })();"""))

}
```

Reference the snippet to make the button show by including the script...

```html
<script class="lift:PlusOne"></script>
```

...and including the code Google ask you to include:

```html
<div class="g-plusone" data-size="medium" data-annotation="bubble"
  data-href="http://www.example.org/"></div>
```


See Also
--------

* [Google +1 Documentation](http://www.google.com/intl/en/webmasters/+1/button/index.html).



Returning snippet markup unchanged
==================================

Problem
-------

You want a snippet to return the original markup associated with the snippet invocation.

Solution
--------

Use the `PassThru` transform that does not change the nodes.  For example, you have a snippet which performs a transforms when some condition is met, but if the condition is not met, you want the snippet return the original markup:

```scala
if (somethingOK)
  ".myclass *" #> <p>Everything is OK</p>
else
  PassThru
```

Discussion
----------

`PassThru` is a `NodeSeq => NodeSeq` function that returns the input it is given (an identity function).


See Also
--------

* Mailing list discussion: [How to return the original markup associated with snippet invocation](https://groups.google.com/forum/?fromgroups#!topic/liftweb/A69tyIBBSdg).
* [BindHelpers.scala](https://github.com/lift/framework/blob/master/core/util/src/main/scala/net/liftweb/util/BindHelpers.scala) source where `PassThru` is defined.
* _Simply Lift_ section [7.10 CSS Selector Transforms](http://simply.liftweb.net/index-7.10.html#toc-Section-7.10).
Snippet not found when using HTML5
==================================

Problem
-------

You're using Lift with the HTML5 parser and one of your snippets, perhaps `<lift:HelloWorld.howdy />`, is rendering with a "Class Not Found" error.

Solution
--------

Switch to the designer-friendly snippet invocation mechanism.  E.g.,

```scala
<div class="lift:HellowWorld.howdy">...</div>
```

Discussion
----------

The HTML5 parser and the traditional Lift XHTML parser have different behaviours, in particular converting elements and attributes to lower case when looking up snippets.  The two links in the _See Also_ section gives a more complete description. 

See Also
--------

* [Html5 and XHTML are different](https://groups.google.com/forum/?fromgroups#!topic/liftweb/H-xe1uRLW1c) important notes from the mailing list.
* Wiki page on [HtmlProperties, XHTML and HTML5](http://www.assembla.com/wiki/show/liftweb/HtmlProperties_XHTML_and_HTML5).
Avoiding CSS and JavaScript caching
===================================

Problem
-------

You've modified CSS or JavaScript in your application, but web browsers have cached your resources and are using the older versions. You'd like to avoid this browser caching.

Solution
--------

Add the `lift:with-resource-id` class attribute to script or link tags:

```html
<script class="lift:with-resource-id" src="/myscript.js" 
 type="text/javascript"></script>
```

The addition of this class will cause Lift to append a "resource id" to your `src` (or `href`), and as this resource id changes each time Lift starts, it defeats browser caching.

The resultant HTML might be:

```html
<script src="/myscript.js?F619732897824GUCAAN=_" 
  type="text/javascript" ></script>
```

Discussion
----------

If you need some other behaviour from `with-resource-id` you can assign a new function of type `String => String` to `LiftRules.attachResourceId`.  The default implementation, shown above, takes the resource name ("/myscript.js" in the example) and returns the resource name with an id appended.  See the `LiftRules` source for additional notes.

Note that some proxies may choose not to cache resources with query parameters at all.

You can also wrap a number of tags inside a `<lift:with-resource-id>...<lift:with-resource-id>` block.  However, avoid doing this in the `<head>` of your page as the HTML5 parser will move the tags to be outside of the head section.

See Also
--------

* Chapter 6 of _Lift in Action_.
* Mailing list discussion of [lift:with-resource-id and html5](https://groups.google.com/forum/?fromgroups#!msg/liftweb/93U-7GY0FuY/Y-T7BESuOwAJ).
* [LiftRules.scala](https://github.com/lift/framework/blob/master/web/webkit/src/main/scala/net/liftweb/http/LiftRules.scala).
* [Optimize caching](https://developers.google.com/speed/docs/best-practices/caching) notes from Google.
* [Custom attachReourceId example](https://gist.github.com/491a86b5da2d3161e774).

Adding to the head of a page
============================

Problem
-------

You use a template for layout, but on one specific page you need to add something to the `<head>` section.


Solution
--------

Use the `lift:head` snippet or CSS class so Lift knows to merge the contents with the `<head>` of your page.  For example, suppose you have the following contents in `templates-hidden/default.html`:

```html
<html lang="en" xmlns:lift="http://liftweb.net/"> 
  <head> 
    <meta charset="utf-8"></meta> 
    <title class="lift:Menu.title">App: </title>
    <script id="jquery" src="/classpath/jquery.js" 
      type="text/javascript"></script>
    <script id="json" src="/classpath/json.js" 
      type="text/javascript"></script>
 </head>
 <body>
 	 <div id="content">The main content will get bound here</div>
 </body>
</html>
```

Also suppose you have `index.html` on which you want to include `my.css` just for that page.  Do so by including the CSS in the part of the page that will get processed and mark it for the head with `lift:head`:

```html
<!DOCTYPE html>
<html>
 <head>
   <title>Special</title>
 </head>
 <body class="lift:content_id=main">
  <div id="main" class="lift:surround?with=default;at=content">
   <link class="lift:head" rel="stylesheet" href="/my.css" type='text/css'>
   <h2>Hello</h2>
  </div>
 </body>
</html>
```

Note that this `index.html` page is validated HTML5, and will produce a result with the custom CSS inside the `<head>` tag, something like this:

```html
<!DOCTYPE html>
<html lang="en">
 <head> 
  <meta charset="utf-8"> 
  <title>App:  Home</title>
  <script type="text/javascript" 
    src="/classpath/jquery.js" id="jquery"></script>
  <script type="text/javascript" 
    src="/classpath/json.js" id="json"></script>
  <link rel="stylesheet" href="/my.css" type="text/css">
 </head>
 <body>
   <div id="main">
     <h2>Hello</h2>
   </div>
  <script type="text/javascript" src="/ajax_request/liftAjax.js"></script>
  <script type="text/javascript"> 
  // <![CDATA[
  var lift_page = "F557573613430HI02U4";
  // ]]>
  </script>
 </body>
</html>
```

Discussion
----------

If you find your tags not appearing the the `<head>` section, check that the HTML in your template and page is valid HTML5. 

You can also use `<lift:head>...</lift:head>` to wrap a number of expressions, and will see `<head_merge>...</head_merge>` used in code example as an alternative to `<lift:head>`.

You may also see `data-lift="head"` is also used as an alternative to `class="lift:head"`.


See Also
--------

* Wiki page on [HtmlProperties XHTML and HTML5](http://www.assembla.com/spaces/liftweb/wiki/HtmlProperties_XHTML_and_HTML5).
* Mailing list discussion on a [designer friendly way of head merge. 
](https://groups.google.com/forum/?fromgroups#!topic/liftweb/rG_pOXdp4Ew).
* [W3C HTML validator](http://validator.w3.org/).
Custom 404 page
===============

Problem
-------

You want to show a customised "404" (page not found) page.

Solution
--------

In `Boot.scala` add the following:

```scala
LiftRules.uriNotFound.prepend(NamedPF("404handler"){
  case (req,failure) => 
    NotFoundAsTemplate(ParsePath(List("404"),"html",false,false))
})
```

The file `src/main/webapp/404.html` will now be served for requests to unknown resources.


Discussion
----------

The `uriNotFound` Lift rule needs to return a `NotFound` in reply to a `Req` (request) and optional `Failure`. This allows you to customise the response based on the type of failure or the request that was made.

There are three types of `NotFound`:

* `NotFoundAsTemplate` is useful to invoke the Lift template processing facilities from a `ParsePath`. 

* `NotFoundAsResponse` allows you to return a specific `LiftResponse`.

* `NotFoundAsNode` wrappers a `NodeSeq` for Lift to translate into a 404 response.

In case you're wondering, the two `false` arguments to `ParsePath` indicates the path we've given isn't absolute, and doesn't end in a slash.


See Also
--------

* [Lift Wiki entry for this topic](http://www.assembla.com/spaces/liftweb/wiki/Custom_404_-_URI_not_found_page)


Other custom status pages
=========================

Problem
-------

You want to show a customised page for certain HTTP status codes.

Solution
--------

Use `LiftRules.responseTransformers` to match against the response and supply an alternative.

For example, suppose we want to provide a customised page for 403 ("Forbidden") statuses created in your Lift application.  In `Boot.scala` we could add the following:

```scala
LiftRules.responseTransformers.append {
  case r if r.toResponse.code == 403 => RedirectResponse("/403.html")
  case r => r
}
```

The file `src/main/webapp/403.html` will now be served for requests that generate 403 status codes.  Other requests are passed through.


Discussion
----------

`LiftRules.responseTransformers` allows you to supply `LiftResponse => LiftResponse` functions to change a response at the end of the HTTP processing cycle.  This is a very general mechanism: in this example we are matching on a status code, but we could match on anything exposed by `LiftResponse`.  We've shown a `RedirectResponse` being returned but there are many different kinds of `LiftResponse` we could send to the client.

One way to test the above example is to add the following to Boot to make all requests to `/secret` return a 403:

```scala
val Protected = If(() => false, () => ForbiddenResponse("no way"))

val entries = List(
  Menu.i("Home") / "index", 
  Menu.i("secret") / "secret" >> Protected,
  Menu.i("403") / "403" >> Hidden 
  // rest of your site map here...
)
```


See Also
--------

* _The Request/Response Lifecycle_ in [Exploring Lift](http://exploring.liftweb.net/master/index-9.html#toc-Section-9.2).
* Mailing list discussion of [custom error 403 page](https://groups.google.com/forum/?fromgroups#!topic/liftweb/9wU0hzQ0wgs%5B1-25%5D).


Links in notices
================

Problem
-------

You want to include a clickable link in your `S.error`, `S.notice` or `S.warning` messages.

Solution
--------

Include a `NodeSeq` containing a link in your notice:

```scala
S.error("checkPrivacyPolicy", 
  <span>See our <a href="/policy">privacy policy</a></span>)
```

You might pair this with the following in your template:

```html
<div class="lift:Msg?id=checkPrivacyPolicy"></div>
```


See Also
--------

* [Lift Notices and Auto Fadeout](http://www.assembla.com/spaces/liftweb/wiki/Lift_Notices_and_Auto_Fadeout) wiki page.
* Mailing list question: [Is there a way for the display of the S.errror to have a clickable URL in it?](https://groups.google.com/forum/?fromgroups#!topic/liftweb/Q6ToHnebOB0)


Rendering Textile markup
========================

Problem
-------

You want to render Textile markup in your web app.

Solution
--------

Install the Lift Textile module in your `build.sbt` file by adding the following to the list of dependencies:

```scala
"net.liftweb" %% "lift-textile" % liftVersion % "compile->default", 
```
You can then render Textile using `toHtml` method:

```scala
scala> import net.liftweb.textile._                   
import net.liftweb.textile._

scala> TextileParser.toHtml("""h1. Hi!              
 | 
 | The module in "Lift":http://www.liftweb.net for turning Textile markup 
 | into HTML is pretty easy to use.
 | 
 | * As you can see
 | * in this example
 |""")
res0: scala.xml.NodeSeq = 
NodeSeq(<h1>Hi!</h1>, 
, <p>The module in <a href="http://www.liftweb.net">Lift</a> for turning 
Textile markup into HTML is pretty easy to use.</p>, 
, <ul><li> As you can see</li>
<li> In this example</li>
</ul>, 
, )
```

Discussion
----------

Textile is one of many [lightweight markup language](http://en.wikipedia.org/wiki/Lightweight_markup_language), but stands out for Lift users as being easy to install and use.


See Also
--------

* [A Textile Reference](http://redcloth.org/hobix.com/textile/).
* [An online Textile to HTML tool](http://textile.thresholdstate.com/) from Threshold State.
* _Lift in Action_,  chapter 7 contains a wiki example that uses the Textile plugin.
* [Lift Source code for Textile](https://github.com/lift/modules/blob/master/textile/src/main/scala/net/liftweb/textile/TextileParser.scala).
* [Lift tests for the Textile plugin](https://github.com/lift/modules/blob/master/textile/src/test/scala/net/liftweb/textile/TextileSpec.scala).
Access restriction by HTTP header
=================================

Problem
-------

You need to control access to a page based on the value of a HTTP header.


Solution
--------

Use a custom `If` in SiteMap:

```scala
val HeaderRequired = If(  
  () => S.request.map(_.header("ALLOWED") == Full("YES")) openOr false,
  "Access not allowed" 
)

// Build SiteMap
val entries = List(
      Menu.i("Restricted") / "restricted" >> HeaderRequired
)
```

In this example  `restricted.html` can only be viewed if the request includes a HTTP header called `ALLOWED` with a value of `Yes`.  Any other request for the page will be redirected with a Lift error notice of "Access not allowed".

This can be tested from the command line using a tool like cURL:

```scala
\$ curl  http://127.0.0.1:8080/restricted.html -H "ALLOWED:YES"
```

Discussion
----------

The `If` test ensures the `() => Boolean` function you supply as a first argument returns `true` before the page it applies to is shown. The second argument is what Lift does if the test isn't true, and should be a `() => LiftResponse` function, meaning you can return whatever you like, including redirects to other pages.

In the example we are making use of a convenient implicit conversation from a `String` ("Access not allowed") to a redirection that will take the user to the home page (actually `LiftRules.siteMapFailRedirectLocation`) with a notice which shows the string.


See Also
--------

* Mailing list thread on [testing a Loc for a HTTP Header Value for Access Control](https://groups.google.com/forum/?fromgroups#!topic/liftweb/CtSGkPbgEVw).
* Source for [Loc.scala](https://github.com/lift/framework/blob/master/web/webkit/src/main/scala/net/liftweb/sitemap/Loc.scala) where `If` and other tests are defined.
* Chapter 7, "SiteMap and access control" in _Lift in Action_.
* [Site map in _Exploring Lift_](http://exploring.liftweb.net/onepage/index.html#toc-Chapter-7).

----

Comments? Questions? Corrections? [Discuss this recipe on the Lift Web mailing list](mailto:liftweb@googlegroups.com?subject=Cookbook%20-%20Access%20restriction%20by%20HTTP%20header).