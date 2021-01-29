# pyAppNotes
## Notes for deploying Python apps to the web

Assume that a dash app has been made. We wish to deploy it to the web, to be accessed by any individual.

Before going any further - we need to understand the *framework* in which browsers, servers and apps behave.

An app is simply a contained set of code that executes according to its Python script. However, there requires a link between something calling the app, and the app responding.

## Web Server Gateway Interface

The Web Server Gateway Interact or WSGI is a calling convention (or protocol) allowing web servers to forward requests to Python applications. It is the first step in deplying a Python app to the web.

One may view WSGI has the middle of two sides:

1. The web server or gateway
2. A Python application


