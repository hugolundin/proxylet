proxylet:  lightweight HTTP reverse proxy built on eventlet
===

Originally written by: Ryan Kelly <https://github.com/rfk> (2007)
Updated by: Marcus Breese (2013)


This module implements a lightweight reverse proxy for HTTP, using non-blocking
IO based on the eventlet module.  It aims to do as little as possible while
supporting simple request/response rewriting and being compatible with HTTP
keep-alive.

Basic operation is via the 'serve' function, which will bind to the
specified host and port and start accepting incoming HTTP requests:

    proxylet.serve(host,port,mapper)

Here 'mapper' is a function taking a proxylet.streams.HTTPRequest object,
and returning either None (for '404 Not Found') or a 3-tuple giving the
destination host, destination port, and a rewriter object.

The rewriter can be any callable that takes request and response streams
as arguments and returns wrapped versions of them, but it will most likely
be a subclass of proxylet.relocate.Relocator.  This class has the necessary
logic to rewrite the request for proxying.

As an example of the available functionality, this mapping function will
proxy requests to /svn to a private subversion server, requests to /files
to a private fileserver, and return 404 for any other paths:

Example code to get this running...

    def _demo_mapper(req):
        """Simple demonstration mapper function, also for testing purposes.
        Proxies the following:
            * /rfk/      :   my personal website
            * /g/        :   google website
            * /morph/    :   morph SVN repo
        """
        from relocate import DrupalRelocator, DAVRelocator, SVNRelocator, Relocator
        rfk = Relocator("http://localhost:8080/rfk","http://www.rfk.id.au/")
        goog = Relocator("http://localhost:8080/g","http://www.google.com/")
        svn = SVNRelocator("http://localhost:8080/svn","http://sphericalmatrix.com/svn/morph")
        if svn.matchesLocal(req.reqURI):
          return svn.mapping
        if rfk.matchesLocal(req.reqURI):
          return rfk.mapping
        if goog.matchesLocal(req.reqURI):
          return goog.mapping
        return None

    def _demo():
        serve('',8080,_demo_mapper)
