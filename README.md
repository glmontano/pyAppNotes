# pyAppNotes
## Notes for deploying Python apps to the web

Assume that a dash app has been made and we wish to deploy it to the web, to be accessed by any individual.

Before going any further - we need to understand the *framework* in which browsers, servers and apps behave.

An app is a contained set of code that executes according to its Python script. However, there requires a link between something calling the app, and the app responding.

One may view WSGI has the middle of two sides:

1. The web server or gateway
2. A Python application

## Web Server Gateway Interface

The Web Server Gateway Interact or WSGI is a calling convention (or protocol) allowing web servers to forward requests to Python applications. It is built for Python applications and their deployment. It's calling convention is Pythonic, and located in a separate file that references the Python app created.

The separate file is conventionally called `wsgi.py`, and for our purpose is as sfollows

```python
import sys

project_home = u"/home/gmontano/pyDash"

if project_home not in sys.path:
    sys.path = [project_home] + sys.path

from dashapp import app

app = app.server
```

In the first instance - the system path is ensured to contain our apps location. Our Python app is located in `dashapp.py`, and thus `from dashapp import app` imports the application variable of our app. Finally WSGI provides the app to teh server through `app.server`, which we have, unfortunately given it the name `app` to maintain the conventions seen in various documentations.

With `app` now attached to WSGI - it may be passed to the web server.

## The Web Server - uWSGI

A notable Python capable web server is uWSGI (I know - it looks like WSGI - but it's a web server **not** the WSGI web protocol). Since it's built for Python, it can be installed using Python's pip: `pip install uswsgi`.

There are a large number of commands which you can see through `uwsgi --help`. For our purpose we're interested in the following

- `--socket <IP:PORT>`: Binds the app to respond to the specified UNIX/TCP socket using default protocol
- `--chdir <path>`: Working directory of app
- `--wsgi-file <wsgiFileName>`: Considering the `chdir` - the name of the wsgi file
- `--module <wsgiFileName:serverName>: Loads the WSGI module in file wsgiFileName, and the server in the file
- `--master`: Enables the master process; complicated but recommended when running apps in production
- `--virtualenv`: The path to the virtual Python directory
- `--processes`: The number of processes to be used
- `--threads`: The number of threads to be used

Primitively, these commands are followed by the `uwsgi` command in the shell terminal. This is clearly cruel. Luckily uwsgi can real an `ini` file with these commands. An example for our purposes is shown below

```shl
socket=0.0.0.0:8050
chdir=/home/gmontano/pyDash/
wsgi-file=wsgi.py
module=wsgi:app
virtualenv=/home/gmontano/env/pyDash/
processes = 1
threads = 4
protocol=http
```

This is contained within the file in the app's directory as `dashapp.ini` The uWSGI server is then called via `uwsgi dashapp.ini`. You may then visit `yourIPAddress:socket` to view your Python app!

