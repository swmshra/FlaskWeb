---
title: Creating web apps with Flask in Azure
description: A tutorial that introduces you to running a Flask Python web app on Azure.
services: app-service\web
documentationcenter: python
tags: python
author: Cody Farmer
manager:
editor: ''

ms.assetid: b7f4ca3a-0b52-4108-90a7-f7e07ac73da0
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: python
ms.topic: article
ms.date: 07/18/2017
ms.author: cofarm

---
# Creating web apps with Flask in Azure
This tutorial describes how to get started running Python with Flask in [Azure App Service Web Apps]. Web Apps provides limited free hosting and rapid deployment, and you can use Python!  As your app grows, you can switch to paid hosting, and easily integrate with other Azure service offerings.

You will create an application named FlaskWeb using Flask,  a popular lightweight framework for Python web development. Other popular choices include Django and Bottle. We suggest reading our comprehensive guide on [Managing Python on Azure App Service], but this tutorial will cover the required portions to get your Flask application running. This application will be named FlaskWeb, and be deployed from your local Git repository.

[!INCLUDE [create-account-and-websites-note](../../includes/create-account-and-websites-note.md)]

> [!NOTE]
> If you want to get started with Azure App Service before signing up for an Azure account, go to [Try App Service](https://azure.microsoft.com/try/app-service/), where you can immediately create a short-lived starter web app in App Service. No credit cards required; no commitments.
>
>

## Prerequisites
* Windows, Mac or Linux
* Git
* Text/Code Editor (Notebook, atom.io, Visual Studio Code, etc.)
* [Python Tools for Visual Studio] (PTVS) - Note: Optional, only for Visual Studio users

For Git on Windows, we recommend [Git for Windows] or [GitHub for Windows].  If you use Visual Studio, you can also use the integrated Git support. You will need to create a local Git repository that we will later use to deploy your Web App. For this example, we're naming the repository **FlaskWeb**.

We also recommend installing [Python Tools for Visual Studio], which may have been included with your Visual Studio installation.  This is optional, but if you have [Visual Studio], including the free Visual Studio Community 2016 or Visual Studio Express 2016 for Web, then this will give you a great Python IDE.


## Create a Web App in the Azure Portal
The first step in creating your app is to create the web app via the [Azure Portal](https://portal.azure.com). This can also be done with Azure CLI, but we won't be covering that here.

1. Log into the Azure Portal and click the **NEW** button in the bottom left corner.
2. Click **Web + Mobile**.
3. Click **Web App**.
4. Configure the Web App, such as creating a new App Service plan and a new resource group for it. Then, click **Create**.
5. Configure Git publishing for your newly created web app by following the instructions at [Local Git Deployment to Azure App Service](app-service-deploy-local-git.md).

## Python Configuration
Now that your web application is launched, we can begin customizing it for Flask. Our first task is to install the Python version of your choice. You can view the details of this in the [managing Python documentation](https://docs.microsoft.com/en-us/visualstudio/python/managing-python-on-azure-app-service#choosing-a-python-version-through-the-azure-portal).

1. On the overview page, scroll down to **Development Tools**.
2. Select **Extensions > Add**.
3. Scroll through the list to find the version of Python you want. For this tutorial, we will be going with **Python 3.6.1 x64**.

## Environment
### Web Application Configuration Files
Our Flask Deployment will use Fast CGI to interface with the web application server. In order to accomplish this, we will need to add two files. First, we'll create a file that is required for every deployment type, `web.config`. This file tells the Web Application Server how we want it to actually run our program.

 **web.config** file:
```xml
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <appSettings>
      	<add key="PYTHONPATH" value="D:\home\site\wwwroot"/>
  	    <add key="WSGI_HANDLER" value="FlaskWeb.wsgi_app"/>
  	    <add key="WSGI_LOG" value="D:\home\LogFiles\wfastcgi.log"/>
      </appSettings>
      <system.webServer>
    	<handlers>
      		<add name="PythonHandler" path="*" verb="*" modules="FastCgiModule" scriptProcessor="D:\home\Python361x64\python.exe|D:\home\Python361x64\wfastcgi.py" resourceType="Unspecified" requireAccess="Script"/>
    	</handlers>
      </system.webServer>
    </configuration>
```

You'll note that we declare `appSettings`, provide a `PythonPath`, and a location for the WSGI handler. Additionally, we provide a path to the `scriptProcessor` which depends on the version of Python you installed. If you are using Python361x64, you don't need to change anything here.

Next, we'll create a FastCGI interface file. We've already referenced this file's location in the config. We will create this file and name it **FlaskWeb.wsgi\_app**. Copy the following code into FlaskWeb.wsgi_app within your local Github repository for this project:

```python
def wsgi_app(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    response_body = 'Hello World'
    yield response_body.encode()
if __name__ == '__main__':
    from wsgiref.simple_server import make_server
    httpd = make_server('localhost', 5555, wsgi_app)
    httpd.serve_forever()
```

In order to easily configure the environment on the Web Application service to contain our dependencies, we will use a `requirements.txt` file to have PIP which is installed with the Python extension take care of the packages our application needs. The below code should be copied into a requirements.txt file in your repository:
```XML
    click==6.7
    Flask==0.12.2
    itsdangerous==0.24
    Jinja2==2.9.6
    MarkupSafe==1.0
    Werkzeug==0.12.2
```

With these files, we will be able to quickly configure the Azure Web App instance once we have pushed our Git Repository to Azure by running a `pip install requirements.txt`.

## Flask Development
Now we're ready to create our Flask application! For the purposes of our example, we're going to make a very simple Flask application.

Flask can be setup in a lot of different ways, and here we'll be making a very minimal application that can be built upon easily. This application doesn't utilize a database or any other external functionality, but all of that can be easily added later!

In the root of your Git repository (along with your configuration files), create a *FlaskWeb* folder. Underneath this folder, you should create *static* and *templates* folders to look like this:
```XML
    /FlaskWeb
    	/Static
    	/Templates
    web.config
    requirements.txt
    FlaskWeb.wsgi_app
```

In the **FlaskWeb** folder we're going to create two files - one to initialize our app and one to define how to handle specific URLs - called *routing*. Create **\_\_init__.py ** (two underscores on each side) and **views.py**.

*\_\_init__.py*
```python
"""
The flask application package. This is where we initialize Flask
"""
from flask import Flask
app = Flask(__name__)
wsgi_app = app.wsgi_app #Registering with IIS
import FlaskWeb.views
```
*views.py*
```python
"""
Routes and views for the flask application.
"""

from datetime import datetime
from flask import render_template
from FlaskWeb import app

@app.route('/')
@app.route('/home')
def home():
    """Renders the home page."""
    return render_template(
        'index.html',
        title='Home Page',
        year=datetime.now().year,
    )

@app.route('/contact')
def contact():
    """Renders the contact page."""
    return render_template(
        'contact.html',
        title='Contact',
        year=datetime.now().year,
        message='Your contact page.'
    )

@app.route('/about')
def about():
    """Renders the about page."""
    return render_template(
        'about.html',
        title='About',
        year=datetime.now().year,
        message='Your application description page.'
    )
```

> [!NOTE]
> Indentation is important in Python!

These two files establish the package **FlaskWeb** so we can import it, and provide routing for our requests to render templates using [Jinja2].

As you may have guessed from the above code, we're going to create three pages - all based off one shared template. These four files - **layout.html**, **index.html**, **about.html**, and **contact.html** - will be placed into the *templates/* folder.

*layout.html*

```xml
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }} - My Flask Application</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u"
     crossorigin="anonymous">
     <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap-theme.min.css"
     integrity="sha384-rHyoN1iRsVXV4nD0JutlnGaslCJuC7uwjduW9SVrLvRYooPp2bWYgmgJQIXwl/Sp" crossorigin="anonymous">

</head>

<body>
    <div class="navbar navbar-inverse navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                <a href="/" class="navbar-brand">Application name</a>
            </div>
            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li><a href="{{ url_for('home') }}">Home</a></li>
                    <li><a href="{{ url_for('about') }}">About</a></li>
                    <li><a href="{{ url_for('contact') }}">Contact</a></li>
                </ul>
            </div>
        </div>
    </div>

    <div class="container body-content">
        {% block content %}{% endblock %}
        <hr />
        <footer>
            <p>&copy; {{ year }} - My Flask Application</p>
        </footer>
    </div>

    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa"
     crossorigin="anonymous"></script>
    {% block scripts %}{% endblock %}

</body>
</html>
```

Inheriting from this layout template, we create three more templates that just define our basic website.

*index.html*
```XML
{% extends "layout.html" %}

{% block content %}

<div class="jumbotron">
    <h1>Flask</h1>
    <p class="lead">Flask is a free web framework for building great Web sites and Web applications using HTML, CSS and JavaScript.</p>
    <p><a href="http://flask.pocoo.org/" class="btn btn-primary btn-large">Learn more &raquo;</a></p>
</div>

<div class="row">
    <div class="col-md-4">
        <h2>Getting started</h2>
        <p>
            Flask gives you a powerful, patterns-based way to build dynamic websites that
            enables a clean separation of concerns and gives you full control over markup
            for enjoyable, agile development.
        </p>
        <p><a class="btn btn-default" href="http://flask.pocoo.org/docs/">Learn more &raquo;</a></p>
    </div>
    <div class="col-md-4">
        <h2>Get more libraries</h2>
        <p>The Python Package Index is a repository of software for the Python programming language.</p>
        <p><a class="btn btn-default" href="https://pypi.python.org/pypi">Learn more &raquo;</a></p>
    </div>
    <div class="col-md-4">
        <h2>Microsoft Azure</h2>
        <p>You can easily publish to Microsoft Azure using Python Tools for Visual Studio. Find out how you can host your application using a free trial today.</p>
        <p><a class="btn btn-default" href="http://azure.microsoft.com">Learn more &raquo;</a></p>
    </div>
</div>

{% endblock %}
```

*about.html*
```xml
{% extends "layout.html" %}

{% block content %}

<h2>{{ title }}.</h2>
<h3>{{ message }}</h3>

<p>Use this area to provide additional information.</p>

{% endblock %}
```

*contact.html*
```XML
{% extends "layout.html" %}

{% block content %}

<h2>{{ title }}.</h2>
<h3>{{ message }}</h3>

<address>
    One Microsoft Way<br />
    Redmond, WA 98052-6399<br />
    <abbr title="Phone">P:</abbr>
    425.555.0100
</address>

<address>
    <strong>Support:</strong>   <a href="mailto:Support@example.com">Support@example.com</a><br />
    <strong>Marketing:</strong> <a href="mailto:Marketing@example.com">Marketing@example.com</a>
</address>

{% endblock %}
```
Congratulations! You've now created all the necessary files for deploying Flask to Azure.

## Deployment
We'll be using Git to deploy to Azure. You will need to use the Git Shell if you're on Windows to add the remote repository that the application is deployed. You should follow our [wonderful guide](app-service-deploy-local-git.md) to get this all setup. Once your local Git repository is linked to the Azure Web App's, we can push our content. Once we push the repository, the Web App will deploy automatically.

### Preparing the Remote Environment
Now that we have files in the remote environment, we can install our Python packages. The easiest way to do this is through Kudu, which is located at `http://<your-app>.scm.azurewebsites.net/`. You can then open up the a console by selecting **Debug Console** --> **CMD**. From here, we can run PIP using the Python extension we installed.

1. Navigate to the folder of the Python installation where the extension was installed, which was `d:\home\python361x64` for our selected extension.
2. Use `python.exe -m pip install --upgrade -r d:\home\site\wwwroot\requirements.txt` to install all of the packages we need.

Now our Web App environment will have all the required Python packages for running Flask.

With everything setup, we can restart our application from the **Overview** blade. Once this completes, we should be able to navigate to our Web App's URL and find our completed Flask Application! The page is using [Bootstrap] to get styled, but you can add your own CSS in the *static* folder.

## Next Steps

With your first web application deployed, you are probably thinking about ways to tinker with this project. In this tutorial, we didn't do any local deployment. Developing an application locally is a much quicker way to inspect your changes, and ensure you don't disrupt a functioning website.

The suggested method for this is [Virtual Environments]. With PIP and a Virtual Environment, we can create the requirements.txt and use it to keep the remote Azure Application running the same dependencies as the local one. You can then run Flask's [Development Server] to preview your web application locally before updating it in Azure.

Follow these links to learn more about Flask and Python Tools for Visual Studio:

* [Flask Documentation]
* [Python Tools for Visual Studio Documentation]

For more information, see also the [Python Developer Center](/develop/python/).

## Troubleshooting
[Troubleshooting Web Applications](web-sites-enable-diagnostic-log.md)

[Managing Python on Azure](https://docs.microsoft.com/en-us/visualstudio/python/managing-python-on-azure-app-service#configuring-your-site)

<!--External Link references-->
[python.org]: http://www.python.org/
[Git for Windows]: http://msysgit.github.io/
[GitHub for Windows]: https://windows.github.com/
[Python Tools for Visual Studio]: http://aka.ms/ptvs
[Visual Studio]: http://www.visualstudio.com/
[Python Tools for Visual Studio Documentation]: http://aka.ms/ptvsdocs
[Flask Documentation]: http://flask.pocoo.org/
[Azure App Service Web Apps]: https://docs.microsoft.com/en-us/azure/app-service-web/app-service-web-overview
[Managing Python on Azure App Service]: https://docs.microsoft.com/visualstudio/python/managing-python-on-azure-app-service#configuring-your-site
[Virtual Environments]: https://virtualenv.pypa.io/en/stable/
[Development Server]: http://flask.pocoo.org/docs/0.12/server/]
[Jinja2]: http://jinja.pocoo.org/docs/2.9/
[Bootstrap]: http://getbootstrap.com/
