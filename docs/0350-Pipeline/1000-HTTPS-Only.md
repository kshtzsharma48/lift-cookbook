Secure requests only 
===================

Problem
-------

You want to ensure clients are using https. 

Solution
--------

Add an `earlyResponse` function in `Boot.scala` redirecting insecure requests.  For example:

```scala
      LiftRules.earlyResponse.append((req: Req) ⇒ if (req.request.scheme != "https") {
        Full(PermRedirectResponse("https://%s%s".format(req.request.serverName, req.request.uri), req, req.cookies: _*))
      } else { Empty })

```

Discussion
----------

The `earlyResponse` call is called early in the Lift pipeline. It is used to excute code before a request is handled and if required exit the pipeline and return a response.  The function signature it expects is `Req ⇒ Box[LiftResponse]`.

The ideal method to ensure requests are via web server configuration, such as apache or nginx. This isn't possibly in some cases, such as when your application is deployed to a PaaS, for instance cloudbees.


See Also
--------

* [Wiki entry](https://www.assembla.com/spaces/liftweb/wiki/SiteMap)
* Source code for [LiftRules.scala](https://github.com/lift/framework/blob/master/web/webkit/src/main/scala/net/liftweb/http/LiftRules.scala).
* [Req.scala](https://github.com/lift/framework/blob/master/web/webkit/src/main/scala/net/liftweb/http/Req.scala) source.
* [Lift pipeline](https://www.assembla.com/spaces/liftweb/wiki/HTTP_Pipeline)
