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
socket = 0.0.0.0:8050
chdir = /home/gmontano/pyDash/
wsgi-file = wsgi.py
module = wsgi:app
master = true
virtualenv =/home/gmontano/env/pyDash/
processes = 5
threads = 4
protocol = http
```

This is contained within the file in the app's directory as `dashapp.ini` The uWSGI server is then called via `uwsgi dashapp.ini`. You may then visit `yourIPAddress:socket` to view your Python app!

## Wait! We need something a little more robust - NGINX!

While uWSGI serves clients with the results of a Python application - there are complex interactions between client and server. This includes requesting for static resources, CSS and the alike. This can significantly reduce server and network load. As such - a robust, industry aged web server is required. 

We'll be using NGINX, and setting it up to call the uWSGI server, which in turns calls our Python application.

The `wsgi.py` file needs to changed as NGINX will be providing it with socket information. Hence the file changes form to become

```shl
chdir=/home/gmontano/pyDash/
wsgi-file=wsgi.py
module=wsgi:app
master = true
virtualenv=/home/gmontano/env/pyDash/
processes = 5
threads = 4

socket = dashapp.sock
chmod-socket = 660
vacuum = true
die-on-term = true
```

The differences are on the last four lines. Socket information will be provided by NGINX, and written into a file called `dashapp.sock`, which is an empty file. The file will have permission 660 so that NGINX can write to it. `vacuum = true` will remove the socket when the process stops. Finally we have `die-on-term = true`, so that uWSGI stops the app instead of reloading it upon closure.

NGINX needs to be used in conjunction with Supervisor. Supervisor is an application which calls upon a configuration file and executes it. We also have a Vapor application working in the background. The configuration file is located in `/etc/supervisor/conf.d`. Therein is a file called `gmBlog.conf`, where `gmBlog` is the name of my application and the parent hierarchy for my Vapor app. Within the configuration file is

```shl
[program:gmBlog]
;command=/home/gmontano/gmBlog/.build/release/Run serve --env production --port 8080 --hostname 0.0.0.0
command=vapor run serve --env production --port 8080 --hostname 0.0.0.0
directory=/home/gmontano/gmBlog/
autostart=true
aurorestart=true
user=root
stdout_logfile=/var/log/supervisor/%(program_name)-stdout.log
stderr_logfile=/var/log/supervisor/%(program_name)-stderr.log
```

Upon using `sudo supervisorctl start gmBlog` - Supervisor will look for configuration files in the aforementioned location and execute the contents. My Vapor app is executed through the `command`, which build the application and is available to anyone requesting port `8080`.

Note that there are two `command`s with the former being commented out `;`. It appears that the original does not recompile the project when a change has been made. 

Finally - we will use NGINX to forward calls from one port to another. Every call for the parent website through port `80`. This too is performed by ammending NGINX's configuration file locatedin `/etc/nginx/sites-enabled/default`. For my Vapor app I have

```shl
server {
        
        listen 80 default_server;
        server_name 157.245.247.123;
        root /home/gmBlog/Public/;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        location / {
                try_files $uri @proxy;
        }

        location @proxy {
                proxy_pass http://127.0.0.1:8080;
                proxy_pass_header Server;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass_header Server;
                proxy_connect_timeout 3s;
                proxy_read_timeout 10s;
        }
}
```

All calls for port 8080 will be directed to `http://127.0.0.1:8080`, which by Supervisor - is the location of my Vapor app.

Next we need to do something similar for our Python app. 


