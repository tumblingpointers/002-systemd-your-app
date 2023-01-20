# Systemd your code that needs to start at boot (Linux)

We come accross a lot of methods to "start my python code on boot". I find systemd to be elegant and use it all the time.

### The subjective beauty in using Systemd

Beauty is subjective. However, if you want some process to run at startup there are a few ways Linux systems will let you do it. Very few ways
will allow you to define a configuration file that is statically typed, and fully-reliable to run processes during boot. These files are called unit files. They have a standard syntax that cover the definitions of how systemd must manage the process. Consequently, there is no scripting at all for common tasks when creating startup scripts, for example waiting for the network connection, executing as a specific user, or even setting up environment variables in config files. The beauty in it for me is that I don't need to do this work myself, and I can make a very clean and simple configuration that can perform complex functionalities.

Let's take the example project I am building here. It is going to be an HTTP API server using Python and FastAPI. Let's say my server needs to start at system boot. I will configure systemd to enable the service and it will run on every boot. Next, let's say our server needs an internet connection. I will add a single line in the configuration to wait for internet connection. I will also want to run the server as one of the users on the machine. I will add a single line for defining the user that should be assumed when running the process. All of this simplicity is thanks to systemd!

I can start from creating a sample Python project that hosts an HTTP API, and work my way into making a unit file to launch this application on system startup. Whenever I start my computer, this API server would be online!

### The Python project

There are a couple of main components that make up our Python application.

#### 1. FastAPI application

The `/` route is implemented in [main.py](example/example/main.py), which simply responds to a GET request with a dummy JSON.

#### 2. Uvicorn

In the [repo](example/example), I have a FastAPI application. The unicorn app is launched using [`run.py`](example/example/run.py).

#### 3. Application Config

This one is critical. I generate an executable for my project using configurations in [pyproject.toml](example/pyproject.toml) and [setup.cfg](example/setup.cfg). While the chunk of them is bolierplate, the section below in [setup.cfg](example/setup.cfg) is what creates application-specific executable:

```
[options.entry_points]
console_scripts =
  my_example_app=example.run:main
```

This creates a executable `my_example_app` that can be installed using `pip install -e .`. Run the command in example directory.

```bash
~002-systemd-your-app$ cd example
~002-systemd-your-app/example$ pip install -e .
...
...
Successfully installed example-0.0.1

~002-systemd-your-app/example$ my_example_app
INFO:     Started server process [35024]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

Now, we just need to create a systemd unit file that runs this executable. We will configure systemd to run this Python app to run on system boot up.
