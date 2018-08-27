Installation and deployment
===========================

Supported platforms
-------------------

INGInious is intended to run on Linux (kernel 3.10+), but can also be run on Windows and macOS thanks to
the Docker toolbox.

Dependencies setup
------------------

INGInious needs:

- Python_ (with pip) **3.5+**
- Docker_ 1.12+
- MongoDB_
- Libtidy
- LibZMQ

.. _Docker: https://www.docker.com
.. _Python: https://www.python.org/
.. _MongoDB: http://www.mongodb.org/

RHEL/Cent OS 7.0+, Fedora 24+
`````````````````````````````

The previously mentioned dependencies can be installed, for Cent OS 7.0+ :
::

    # curl -fsSL https://get.docker.com/ | sh #This will setup the Docker repo
    # yum install -y epel-release https://centos7.iuscommunity.org/ius-release.rpm
    # yum install -y git mongodb mongodb-server gcc libtidy python35u python35u-pip python35u-devel zeromq-devel

Or, for Fedora 24+:
::

    # curl -fsSL https://get.docker.com/ | sh #This will setup the Docker repo
    # dnf install -y git mongodb mongodb-server gcc libtidy python3 python3-pip python3-devel zeromq-devel

You may also add ``openldap-devel`` if you want to use the LDAP auth plugin and
``xmlsec1-openssl-devel libtool-ltdl-devel`` for the SAML2 auth plugin.

.. DANGER::
    Due to compatibility issues, it is recommended to disable SELinux on the target machine.

You can now start and enable the ``mongod`` and ``docker`` services:
::

    # systemctl start mongod
    # systemctl enable mongod
    # systemctl start docker
    # systemctl enable docker

Ubuntu 16.04+
`````````````

The previously mentioned dependencies can be installed, for Ubuntu 16.04+:
::

    # curl -fsSL https://get.docker.com/ | sh #This will setup the Docker repo
    # apt-get install git mongodb gcc tidy python3 python3-pip python3-dev libzmq-dev

You may also add ``libldap2-dev libsasl2-dev libssl-dev`` if you want to use the LDAP auth plugin and
``libxmlsec1-dev libltdl-dev`` for the SAML2 auth plugin

You can now start and enable the ``mongod`` and ``docker`` services:
::

    # systemctl start mongodb
    # systemctl enable mongodb
    # systemctl start docker
    # systemctl enable docker

OS X 10.9+
``````````

We use brew_ to install some packages. Packages are certainly available too via macPorts.

.. _brew: http://brew.sh/

::

    $ brew install mongodb
    $ brew install python3

Follow the instruction of brew to enable mongodb.

The next step is to install `Docker for Mac <https://docs.docker.com/docker-for-mac/>`_.

Windows
```````

.. DANGER::
    INGInious rely on Docker to run containers. While Docker is supported on Windows 10 (version 1607), INGInious does not
    provide support for Windows containers yet.

The recommended way to run INGInious under Windows is by using a Linux virtual machine, for much more simplicity. One can
also only run the Docker agent under a Linux virtual machine and run the backend and selected frontend under Windows.

In the later case, you'll need to install Python 3.5+, MongoDB, LibTidy and LibZMQ.

.. _Installpip:

Installing INGInious
--------------------

The recommended setup is to install INGInious via pip and the master branch of the INGInious git repository.
This allows you to use the latest development version. This version is currently the supported one for issues.
::

    $ pip3 install --upgrade git+https://github.com/UCL-INGI/INGInious.git@v0.5

This will automatically upgrade an existing version.

.. note::

   You may want to enable the LDAP/SAML2 plugin or use FCGI/UWSGI instead of the web.py default webserver.
   In this case, you have to install more packages: simply add ``[cgi]``, ``[uwgsi]``, ``[ldap]`` or ``[saml2]`` to the above command, depending on your needs:

   ::

       $ pip3 install --upgrade git+https://github.com/UCL-INGI/INGInious.git@v0.5#egg=INGInious[cgi,ldap]

.. _config:

Configuring INGInious
---------------------

INGInious comes with a mini-LMS web app that provides statistics, groups management, and the
INGInious studio, that allows to modify and test your tasks directly in your browser. It supports the LTI_ interface
that allows to interface with Learning Management System via the LTI_ specification. Any LMS supporting LTI_ is
compatible. This includes Moodle, edX, among many others.

.. _LTI: http://www.imsglobal.org/LTI/v1p1/ltiIMGv1p1.html

To configure the web app automatically, use the ``inginious-install`` CLI.

::

    $ inginious-install

This will help you create the configuration file in the current directory. For manual configuration and details, see
:ref:`ConfigReference`.

The detailed ``inginious-install`` reference can be found at :ref:`inginious-install`.

Running INGInious
-----------------

During the configuration step, you were asked to setup either a local or remote backend. In the former case, the frontend
will automatically start a local backend and grading agents.

With local backend/agent
````````````````````````
To run the frontend, please use the ``inginious-webapp`` CLI. This will open a small Python
web server and display the url on which it is bind in the console. Some parameters (configuration file, host, port)
can be specified. Details are available at :ref:`inginious-webapp`.

With remote backend/agent
`````````````````````````
To run INInious with a remote backend (and agents), do as follows:

#. On the backend host, launch the backend (see :ref:`inginious-backend`) :
   ::

        inginious-backend tcp://backend-host:2001 tcp://backend-host:2000

   The agents will connect on ``tcp://backend-host:2001`` and clients on ``tcp://backend-host:2000``
#. Possibly on different hosts, launch the Docker and MCQ agents (see :ref:`inginious-agent-docker`
   and :ref:`inginious-agent-mcq`) :
   ::

        inginious-agent-docker tcp://backend-host:2001
        inginious-agent-mcq tcp://backend-host:2001
#. In your INGInious frontend configuration file (see :ref:`ConfigReference`), set ``backend`` to :
   ::

        backend: tcp://backend-host:2000
#. Run the frontend using :ref:`inginious-webapp`.

.. _webterm_setup:

Webterm setup
-------------

An optional web terminal can be used with INGInious to load the remote SSH debug session. This rely on an external tool.

To install this tool :
::

    $ git clone https://github.com/UCL-INGI/INGInious-xterm
    $ cd INGInious-xterm && npm install

You can then launch the tool by running:
::

    $ npm start bind_hostname bind_port debug_host:debug_ports

This will launch the app on ``http://bind_hostname:bind_port``. The ``debug_host`` and ``debug_ports`` parameters are
the debug paramaters on the local (see :ref:`ConfigReference`) or remote (see :ref:`inginious-agent-docker`) Docker agent.

To make the INGInious frontend aware of that application, update your configuration file by setting the ``webterm``
field to ``http://bind_hostname:bind_port`` (see :ref:`ConfigReference`).

For more information on this tool, please see `INGInious-xterm <https://github.com/UCL-INGI/INGInious-xterm>`_. Please
note that INGInious-xterm must be launched using SSL if the frontend is launched using SSL.

.. _production:

Webserver configuration
-----------------------

.. _lighttpd:

.. WARNING::
    In configurations below, environment variables accessible to the application must be explicitly repeated.
    **If you use a local backend with remote Docker daemon**, you may need to set the ``DOCKER_HOST`` variable.
    To know the value to set, start a terminal that has access to the docker daemon (the terminal should be able to run
    ``docker info``), and write ``echo $DOCKER_HOST``. If it returns nothing, just ignore this comment. It is possible
    that you may need to do the same for the env variable ``DOCKER_CERT_PATH`` and ``DOCKER_TLS_VERIFY`` too.

Using lighttpd
``````````````

In production environments, you can use lighttpd in replacement of the built-in Python server.
This guide is made for CentOS 7.x.

Install lighttpd with fastcgi:

::

    # yum install lighttpd lighttpd-fastcgi

Add the ``lighttpd`` user in the necessary groups, to allow it to launch new containers and to connect to mongodb:

::

    # usermod -aG docker lighttpd
    # usermod -aG mongodb lighttpd

Create a folder for INGInious, for example ``/var/www/INGInious``, and change the directory owner to ``lighttpd``:

::

    # mkdir -p /var/www/INGInious
    # chown -R lighttpd:lighttpd /var/www/INGInious

Put your configuration file in that folder, as well as your tasks, backup, download, and temporary (if local backend)
directories (see :ref:`config` for more details on these folders).

Once this is done, we can configure lighttpd. First, the file ``/etc/lighttpd/modules.conf``, to load these modules:
::

    server.modules = (
        "mod_access",
        "mod_alias"
    )

    include "conf.d/compress.conf"
    include "conf.d/fastcgi.conf"

You can then replace the content of fastcgi.conf with:
::

    server.modules   += ( "mod_fastcgi" )
    server.modules   += ( "mod_rewrite" )

    alias.url = (
        "/static/" => "/usr/lib/python3.5/site-packages/inginious/frontend/static/"
    )

    fastcgi.server = ( "/inginious-webapp" =>
        (( "socket" => "/tmp/fastcgi.socket",
            "bin-path" => "/usr/bin/inginious-webapp",
            "max-procs" => 1,
            "bin-environment" => (
                "INGINIOUS_WEBAPP_HOST" => "0.0.0.0",
                "INGINIOUS_WEBAPP_PORT" => "80",
                "INGINIOUS_WEBAPP_CONFIG" => "/var/www/INGInious/configuration.yaml",
                "REAL_SCRIPT_NAME" => ""
            ),
            "check-local" => "disable"
        ))
    )

    url.rewrite-once = (
        "^/favicon.ico$" => "/static/icons/favicon.ico",
        "^/static/(.*)$" => "/static/$1",
        "^/(.*)$" => "/inginious-webapp/$1"
    )

The ``INGINIOUS_WEBAPP`` prefixed environment variables are used to replace the default command line parameters.
See :ref:`inginious-webapp` for more details.

The ``REAL_SCRIPT_NAME`` environment variable must be specified under lighttpd if you plan to access the application
from another path than the specified one. In this case, lighttpd forces to set a non-root path ``/inginious-webapp``,
while a root access if wanted, in order to serve static files correctly. Therefore, this environment variable is set
to an empty string in addition to the rewrite rule.

Finally, start the server:

::

    # systemctl enable lighttpd
    # systemctl start lighttpd

.. _apache:

Using Apache
````````````

You may also want to use Apache. You should install `mod_wsgi`. WSGI interfaces are supported through the
`inginious-webapp` script. This guide is made for CentOS 7.x.

Install the following packages (please note that the Python3.5+ version of *mod_wsgi* is required):
::

    # yum install httpd httpd-devel
    # pip3.5 install mod_wsgi

Add the ``apache`` user in the necessary groups, to allow it to launch new containers and to connect to mongodb:
::

    # usermod -aG docker apache
    # usermod -aG mongodb apache

Create a folder for INGInious, for example ``/var/www/INGInious``, and change the directory owner to ``apache``:
::

    # mkdir -p /var/www/INGInious
    # chown -R apache:apache /var/www/INGInious

Put your configuration file in that folder, as well as your tasks, backup, download, and temporary (if local backend)
directories (see :ref:`config` for more details on these folders).

Set the environment variables used by the INGInious CLI scripts in the Apache service environment file
(see lighttpd_ for more details):
::

    # cat  << EOF >> /etc/sysconfig/httpd
    INGINIOUS_WEBAPP_CONFIG="/var/www/INGInious/configuration.yaml"
    INGINIOUS_WEBAPP_HOST="0.0.0.0"
    INGINIOUS_WEBAPP_PORT="80"
    EOF
    # rm /etc/httpd/conf.d/welcome.conf

Please note that the service environment file ``/etc/sysconfig/httpd`` may differ from your distribution and wether it
uses *systemd* or *init*.

You can then modify your ``/etc/httpd/conf/httpd.conf`` file to apply the following rules:
::

    LoadModule wsgi_module /usr/lib64/python3.5/site-packages/mod_wsgi/server/mod_wsgi-py35.cpython-35m-x86_64-linux-gnu.so

    WSGIScriptAlias / "/usr/bin/inginious-webapp"
    WSGIScriptReloading On

    Alias /static /usr/lib/python3.5/site-packages/inginious/frontend/static

    <Directory "/usr/bin">
        <Files "inginious-webapp">
            Require all granted
        </Files>
    </Directory>

    <DirectoryMatch "/usr/lib/python3.5/site-packages/inginious/frontend/static">
        Require all granted
    </DirectoryMatch>

Please note that the compiled *wsgi* module path may differ according to the exact Python version you are running.
