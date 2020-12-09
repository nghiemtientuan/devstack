Open edX Devstack |Build Status|
================================

Get up and running quickly with Open edX services.

This project replaces the older Vagrant-based devstack with a
multi-container approach driven by `Docker Compose`_.

A Devstack installation includes the following Open edX components:

* The Learning Management System (LMS)
* Open edX Studio
* Discussion Forums
* Open Response Assessments (ORA)
* E-Commerce
* Credentials
* Notes
* Course Discovery
* XQueue
* Open edX Search
* A demonstration Open edX course

It also includes the following extra components:

* XQueue
* The components needed to run the Open edX Analytics Pipeline. This is the primary extract, transform, and load (ETL) tool that extracts and analyzes data from the other Open edX services.
* The Program Console micro-frontend
* edX Registrar service.

Prerequisites
-------------

You will need to have the following installed:

- make
- python 3
- docker

This project requires **Docker 17.06+ CE**.  We recommend Docker Stable, but
Docker Edge should work as well.

**NOTE:** Switching between Docker Stable and Docker Edge will remove all images and
settings.  Don't forget to restore your memory setting and be prepared to
provision.

For macOS users, please use `Docker for Mac`_. Previous Mac-based tools (e.g.
boot2docker) are *not* supported.

Since a Docker-based devstack runs many containers,
you should configure Docker with a sufficient
amount of resources. We find that `configuring Docker for Mac`_ with
a minimum of 2 CPUs and 8GB of memory does work.

`Docker for Windows`_ may work but has not been tested and is *not* supported.

If you are using Linux, use the ``overlay2`` storage driver, kernel version
4.0+ and *not* ``overlay``. To check which storage driver your
``docker-daemon`` uses, run the following command.

.. code:: sh

   docker info | grep -i 'storage driver'

Getting Started (Base)
---------------

All of the services can be run by following the steps below. For analyticstack, follow `Getting Started on Analytics`_.

1. Going to folder that include OpenEdx resources (Example: /root/user/edx):

2. Clone custom devstack repo:

    .. code:: sh

       git clone https://github.com/nghiemtientuan/devstack.git

3. Going to folder devstack:

    .. code:: sh

       cd devstack

4. Set OPENEDX_RELEASE param global:

    .. code:: sh

       export OPENEDX_RELEASE=juniper.master

5. Checkout branch OPENEDX_RELEASE:

    .. code:: sh

       git checkout open-release/$OPENEDX_RELEASE

6. Install the requirements inside of a `Python virtualenv`_ (recommend)
    .. code:: sh

        pip install virtualenv
        virtualenv venv
        virtualenv -p python3 venv
        source venv/bin/activate

   .. code:: sh

        make requirements

   This will install docker-compose and other utilities into your virtualenv.

    If you are done working in the virtual environment for the moment, you can deactivate it:
    .. code:: sh

        deactivate

7. The Docker Compose file mounts a host volume for each service's executing code. The host directory defaults to be a sibling of this directory. Run following command to clone basic repos.

   .. code:: sh

       make dev.clone https

8. Pull any changes made to the various images on which the devstack depends.

   .. code:: sh

       make dev.pull

9. Run the provision command

   The username and password for the superusers are both ``edx``. You can access
   the services directly via Django admin at the ``/admin/`` path, or login via
   single sign-on at ``/login/``.

   .. code:: sh

       make dev.provision

   This is expected to take a while, produce a lot of output from a bunch of steps, and finally end with ``Provisioning complete!``

10. Start the services. This command will mount the repositories

   **NOTE:** it may take up to 60 seconds for the LMS to start, even after the ``make dev.up`` command outputs ``done``.

   .. code:: sh

       make dev.up

11. Done! Setup Successfully

Getting Started on Analytics
----------------------------

Analyticstack can be run by following the steps below.

1. Follow steps `1` to `10` from `Getting Started`_ section.

2. Before running the provision command, make sure to pull the relevant
   docker images from dockerhub by running the following commands:

   .. code:: sh

       make pull.analytics_pipeline

3. Run the provision command to configure the analyticstack.

   .. code:: sh

       make dev.provision.analytics_pipeline

4. Start the analytics service. This command will mount the repositories under the
   DEVSTACK\_WORKSPACE directory.

   **NOTE:** it may take up to 60 seconds for Hadoop services to start.

   .. code:: sh

       make dev.up.analytics_pipeline

5. Done! Setup Successfully

Useful Commands
----------------------------
- To access the analytics pipeline shell, run the following command. All analytics
pipeline job/workflows should be executed after accessing the shell.

    .. code:: sh

        make analytics-pipeline-shell

- To see logs from containers running in detached mode, you can either use
"Kitematic" (available from the "Docker for Mac" menu), or by running the
following command:

    .. code:: sh

        make logs

- To view the logs of a specific service container run ``make <service>-logs``.
For example, to access the logs for Hadoop's namenode, you can run:

    .. code:: sh

        make namenode-logs

- To reset your environment and start provisioning from scratch, you can run:

    .. code:: sh

        make destroy

 **NOTE:** Be warned! This will remove all the containers and volumes
 initiated by this repository and all the data ( in these docker containers )
 will be lost.

- For information on all the available ``make`` commands, you can run:

    .. code:: sh

        make help

- After the services have started, if you need shell access to one of the
services, run ``make <service>-shell``. For example to access the
Catalog/Course Discovery Service, you can run:

    .. code:: sh

        make discovery-shell

- To see logs from containers running in detached mode, you can either use
"Kitematic" (available from the "Docker for Mac" menu), or by running the
following:

    .. code:: sh

        make logs

- To view the logs of a specific service container run ``make <service>-logs``.
For example, to access the logs for Ecommerce, you can run:

    .. code:: sh

        make ecommerce-logs

- To reset your environment and start provisioning from scratch, you can run:

    .. code:: sh

        make destroy

- For information on all the available ``make`` commands, you can run:

    .. code:: sh

        make help

Usernames and Passwords
-----------------------

The provisioning script creates a Django superuser for every service.

::

    Email: edx@example.com
    Username: edx
    Password: edx

The LMS also includes demo accounts. The passwords for each of these accounts
is ``edx``.

  .. list-table::
   :widths: 20 60
   :header-rows: 1

   * - Account
     - Description
   * - ``staff@example.com``
     - An LMS and Studio user with course creation and editing permissions.
       This user is a course team member with the Admin role, which gives
       rights to work with the demonstration course in Studio, the LMS, and
       Insights.
   * - ``verified@example.com``
     - A student account that you can use to access the LMS for testing
       verified certificates.
   * - ``audit@example.com``
     - A student account that you can use to access the LMS for testing course
       auditing.
   * - ``honor@example.com``
     - A student account that you can use to access the LMS for testing honor
       code certificates.

Service List
------------

The services marked as ``Default`` are provisioned/pulled/run whenever you run
``make dev.provision`` / ``make dev.pull`` / ``make dev.up``, respectively.

The extra services are provisioned/pulled/run when specifically requested (e.g.,
``make dev.provision.xqueue`` / ``make dev.pull.xqueue`` / ``make dev.up.xqueue``).

+---------------------------+-------------------------------------+----------------+------------+
| Service                   | URL                                 | Type           | Role       |
+===========================+=====================================+================+============+
| `lms`_                    | http://localhost:18000/             | Python/Django  | Default    |
+---------------------------+-------------------------------------+----------------+------------+
| `studio`_                 | http://localhost:18010/             | Python/Django  | Default    |
+---------------------------+-------------------------------------+----------------+------------+
| `forum`_                  | http://localhost:44567/api/v1/      | Ruby/Sinatra   | Default    |
+---------------------------+-------------------------------------+----------------+------------+
| `discovery`_              | http://localhost:18381/api-docs/    | Python/Django  | Default    |
+---------------------------+-------------------------------------+----------------+------------+
| `ecommerce`_              | http://localhost:18130/dashboard/   | Python/Django  | Default    |
+---------------------------+-------------------------------------+----------------+------------+
| `credentials`_            | http://localhost:18150/api/v2/      | Python/Django  | Default    |
+---------------------------+-------------------------------------+----------------+------------+
| `edx_notes_api`_          | http://localhost:18120/api/v1/      | Python/Django  | Default    |
+---------------------------+-------------------------------------+----------------+------------+
| `frontend-app-publisher`_ | http://localhost:18400/             | MFE (React.js) | Default    |
+---------------------------+-------------------------------------+----------------+------------+
| `gradebook`_              | http://localhost:1994/              | MFE (React.js) | Default    |
+---------------------------+-------------------------------------+----------------+------------+
| `registrar`_              | http://localhost:18734/api-docs/    | Python/Django  | Extra      |
+---------------------------+-------------------------------------+----------------+------------+
| `program-console`_        | http://localhost:1976/              | MFE (React.js) | Extra      |
+---------------------------+-------------------------------------+----------------+------------+
| `xqueue`_                 | http://localhost:18040/api/v1/      | Python/Django  | Extra      |
+---------------------------+-------------------------------------+----------------+------------+
| `analyticspipeline`_      | http://localhost:4040/              | Python         | Extra      |
+---------------------------+-------------------------------------+----------------+------------+
| `marketing`_              | http://localhost:8080/              | PHP/Drupal     | Extra      |
+---------------------------+-------------------------------------+----------------+------------+

.. _credentials: https://github.com/edx/credentials
.. _discovery: https://github.com/edx/course-discovery
.. _ecommerce: https://github.com/edx/ecommerce
.. _edx_notes_api: https://github.com/edx/edx-notes-api
.. _forum: https://github.com/edx/cs_comments_service
.. _frontend-app-publisher: https://github.com/edx/frontend-app-publisher
.. _gradebook: https://github.com/edx/frontend-app-gradebook
.. _lms: https://github.com/edx/edx-platform
.. _program-console: https://github.com/edx/frontend-app-program-console
.. _registrar: https://github.com/edx/registrar
.. _studio: https://github.com/edx/edx-platform
.. _lms: https://github.com/edx/edx-platform
.. _analyticspipeline: https://github.com/edx/edx-analytics-pipeline
.. _marketing: https://github.com/edx/edx-mktg
.. _xqueue: https://github.com/edx/xqueue

Questions & Troubleshooting – Multiple Named Open edX Releases on Same Machine
------------------------------------------------------------------------------

How do I create relational database dumps?
------------------------------------------
We use relational database dumps to spend less time running relational database
migrations and to speed up the provisioning of a devstack. These dumps are saved
as .sql scripts in the root directory of this git repository and they should be
updated occasionally - when relational database migrations take a prolonged amount
of time *or* we want to incorporate database schema changes which were done manually.

To update the relational database dumps:

1. Backup the data of your existing devstack if needed
2. If you are unsure whether the django_migrations tables (which keeps which migrations
were already applied) in each database are consistent with the existing database dumps,
disable the loading of these database dumps during provisioning by commenting out
the calls to ``load-db.sh`` in the provision-*.sh scripts. This ensures a start with a
completely fresh database and incorporates any changes that may have required some form
of manual intervention for existing installations (e.g. drop/move tables).
3. Run the shell script which destroys any existing devstack, creates a new one
and updates the relational database dumps:

.. code:: sh

   ./update-dbs-init-sql-scripts.sh

How do I keep my database up to date?
-------------------------------------

You can run Django migrations as normal to apply any changes recently made
to the database schema for a particular service.  For example, to run
migrations for LMS, enter a shell via ``make lms-shell`` and then run:

.. code:: sh

   paver update_db

Alternatively, you can discard and rebuild the entire database for all
devstack services by re-running ``make dev.provision`` or
``make dev.sync.provision`` as appropriate for your configuration.  Note that
if your branch has fallen significantly behind master, it may not include all
of the migrations included in the database dump used by provisioning.  In these
cases, it's usually best to first rebase the branch onto master to
get the missing migrations.

How do I access a database shell?
---------------------------------

To access a MySQL or Mongo shell, run the following commands, respectively:

.. code:: sh

   make mysql-shell
   mysql

.. code:: sh

   make mongo-shell
   mongo

How do I make migrations?
-------------------------

Log into the LMS shell, source the ``edxapp`` virtualenv, and run the
``makemigrations`` command with the ``devstack_docker`` settings:

.. code:: sh

   make lms-shell
   source /edx/app/edxapp/edxapp_env
   cd /edx/app/edxapp/edx-platform
   ./manage.py <lms/cms> makemigrations <appname> --settings=devstack_docker

Also, make sure you are aware of the `Django Migration Don'ts`_ as the
edx-platform is deployed using the red-black method.


How do I upgrade Node.JS packages?
----------------------------------

JavaScript packages for Node.js are installed into the ``node_modules``
directory of the local git repository checkout which is synced into the
corresponding Docker container.  Hence these can be upgraded via any of the
usual methods for that service (``npm install``,
``paver install_node_prereqs``, etc.), and the changes will persist between
container restarts.

How do I upgrade Python packages?
---------------------------------

Unlike the ``node_modules`` directory, the ``virtualenv`` used to run Python
code in a Docker container only exists inside that container.  Changes made to
a container's filesystem are not saved when the container exits, so if you
manually install or upgrade Python packages in a container (via
``pip install``, ``paver install_python_prereqs``, etc.), they will no
longer be present if you restart the container.  (Devstack Docker containers
lose changes made to the filesystem when you reboot your computer, run
``make down``, restart or upgrade Docker itself, etc.) If you want to ensure
that your new or upgraded packages are present in the container every time it
starts, you have a few options:

* Merge your updated requirements files and wait for a new `edxops Docker image`_
  for that service to be built and uploaded to `Docker Hub`_.  You can
  then download and use the updated image (for example, via ``make dev.pull.<service>``).
  The discovery and edxapp images are built automatically via a Jenkins job. All other
  images are currently built as needed by edX employees, but will soon be built
  automatically on a regular basis. See `How do I build images?`_
  for more information.
* You can update your requirements files as appropriate and then build your
  own updated image for the service as described above, tagging it such that
  ``docker-compose`` will use it instead of the last image you downloaded.
  (Alternatively, you can temporarily edit ``docker-compose.yml`` to replace
  the ``image`` entry for that service with the ID of your new image.) You
  should be sure to modify the variable override for the version of the
  application code used for building the image. See `How do I build images?`_.
  for more information.
* You can temporarily modify the main service command in
  ``docker-compose.yml`` to first install your new package(s) each time the
  container is started.  For example, the part of the studio command which
  reads ``...&& while true; do...`` could be changed to
  ``...&& pip install my-new-package && while true; do...``.
* In order to work on locally pip-installed repos like edx-ora2, first clone
  them into ``../src`` (relative to this directory). Then, inside your lms shell,
  you can ``pip install -e /edx/src/edx-ora2``. If you want to keep this code
  installed across stop/starts, modify ``docker-compose.yml`` as mentioned
  above.

Changing LMS/CMS settings
-------------------------
The LMS and CMS read many configuration settings from the container filesystem
in the following locations:

- ``/edx/app/edxapp/lms.env.json``
- ``/edx/app/edxapp/lms.auth.json``
- ``/edx/app/edxapp/cms.env.json``
- ``/edx/app/edxapp/cms.auth.json``

Changes to these files will *not* persist over a container restart, as they
are part of the layered container filesystem and not a mounted volume. However, you
may need to change these settings and then have the LMS or CMS pick up the changes.

To restart the LMS/CMS process without restarting the container, kill the LMS or CMS
process and the watcher process will restart the process within the container. You can
kill the needed processes from a shell within the LMS/CMS container with a single line of bash script:

LMS:

.. code:: sh

    kill -9 $(ps aux | grep 'manage.py lms' | egrep -v 'while|grep' | awk '{print $2}')

CMS:

.. code:: sh

    kill -9 $(ps aux | grep 'manage.py cms' | egrep -v 'while|grep' | awk '{print $2}')

From your host machine, you can also run ``make lms-restart`` or
``make studio-restart`` which run those commands in the containers for you.

PyCharm Integration
-------------------

See the `Pycharm Integration documentation`_.

devpi Caching
-------------

LMS and Studio use a devpi container to cache PyPI dependencies, which speeds up several Devstack operations.
See the `devpi documentation`_.

Debugging using PDB
-------------------

It's possible to debug any of the containers' Python services using PDB. To do so,
start up the containers as usual with:

.. code:: sh

    make dev.up

This command starts each relevant container with the equivalent of the '--it' option,
allowing a developer to attach to the process once the process is up and running.

To attach to a container and its process, use ``make <service>-attach``. For example:

.. code:: sh

    make lms-attach

Set a PDB breakpoint anywhere in the code using:

.. code:: sh

    import pdb;pdb.set_trace()

and your attached session will offer an interactive PDB prompt when the breakpoint is hit.

To detach from the container, you'll need to stop the container with:

.. code:: sh

    make stop

or a manual Docker command to bring down the container:

.. code:: sh

   docker kill $(docker ps -a -q --filter="name=edx.devstack.<container name>")

Alternatively, some terminals allow detachment from a running container with the
``Ctrl-P, Ctrl-Q`` key sequence.

Running LMS and Studio Tests
----------------------------

After entering a shell for the appropriate service via ``make lms-shell`` or
``make studio-shell``, you can run any of the usual paver commands from the
`edx-platform testing documentation`_.  Examples:

.. code:: sh

    paver run_quality
    paver test_a11y
    paver test_bokchoy
    paver test_js
    paver test_lib
    paver test_python

Tests can also be run individually. Example:

.. code:: sh

    pytest openedx/core/djangoapps/user_api

Tests can also be easily run with a shortcut from the host machine,
so that you maintain your command history:

.. code:: sh

    ./in lms pytest openedx/core/djangoapps/user_api

Connecting to Browser
~~~~~~~~~~~~~~~~~~~~~

If you want to see the browser being automated for JavaScript or bok-choy tests,
you can connect to the container running it via VNC.

+------------------------+----------------------+
| Browser                | VNC connection       |
+========================+======================+
| Firefox (Default)      | vnc://0.0.0.0:25900  |
+------------------------+----------------------+
| Chrome (via Selenium)  | vnc://0.0.0.0:15900  |
+------------------------+----------------------+

On macOS, enter the VNC connection string in the address bar in Safari to
connect via VNC. The VNC passwords for both browsers are randomly generated and
logged at container startup, and can be found by running ``make vnc-passwords``.

Most tests are run in Firefox by default.  To use Chrome for tests that normally
use Firefox instead, prefix the test command with
``SELENIUM_BROWSER=chrome SELENIUM_HOST=edx.devstack.chrome``.

Running End-to-End Tests
------------------------

To run the base set of end-to-end tests for edx-platform, run the following
make target:

.. code:: sh

   make e2e-tests

If you want to use some of the other testing options described in the
`edx-e2e-tests README`_, you can instead start a shell for the e2e container
and run the tests manually via paver:

.. code:: sh

    make e2e-shell
    paver e2e_test --exclude="whitelabel\|enterprise"

The browser running the tests can be seen and interacted with via VNC as
described above (Firefox is used by default).

Troubleshooting: General Tips
-----------------------------

If you are having trouble with your containers, this sections contains some troubleshooting tips.

Check the logs
~~~~~~~~~~~~~~

If a container stops unexpectedly, you can look at its logs for clues::

    docker-compose logs lms

Update the code and images
~~~~~~~~~~~~~~~~~~~~~~~~~~

Make sure you have the latest code and Docker images.

Pull the latest Docker images by running the following command from the devstack
directory:

.. code:: sh

   make dev.pull

Pull the latest Docker Compose configuration and provisioning scripts by running
the following command from the devstack directory:

.. code:: sh

   git pull

Lastly, the images are built from the master branches of the application
repositories (e.g. edx-platform, ecommerce, etc.). Make sure you are using the
latest code from the master branches, or have rebased your branches on master.

Clean the containers
~~~~~~~~~~~~~~~~~~~~

Sometimes containers end up in strange states and need to be rebuilt. Run
``make down`` to remove all containers and networks. This will **NOT** remove your
data volumes.

Reset
~~~~~

Sometimes you just aren't sure what's wrong, if you would like to hit the reset button
run ``make dev.reset``.

Running this command will perform the following steps:

* Bring down all containers
* Reset all git repositories to the HEAD of master
* Pull new images for all services
* Compile static assets for all services
* Run migrations for all services

It's good to run this before asking for help.

Start over
~~~~~~~~~~

If you want to completely start over, run ``make destroy``. This will remove
all containers, networks, AND data volumes.

Resetting a database
~~~~~~~~~~~~~~~~~~~~

In case you botched a migration or just want to start with a clean database.

1. Open up the mysql shell and drop the database for the desired service::

    make mysql-shell
    mysql
    DROP DATABASE (insert database here)

2. From your devstack directory, run the provision script for the service. The
   provision script should handle populating data such as Oauth clients and
   Open edX users and running migrations::

    ./provision-(service_name)


Troubleshooting: Common issues
------------------------------

Docker is using lots of CPU time when it should be idle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On the Mac, this often manifests as the ``hyperkit`` process using a high
percentage of available CPU resources.  To identify the container(s)
responsible for the CPU usage:

.. code:: sh

    make stats

Once you've identified a container using too much CPU time, check its logs;
for example:

.. code:: sh

    make lms-logs

The most common culprit is an infinite restart loop where an error during
service startup causes the process to exit, but we've configured
``docker-compose`` to immediately try starting it again (so the container will
stay running long enough for you to use a shell to investigate and fix the
problem).  Make sure the set of packages installed in the container matches
what your current code branch expects; you may need to rerun ``pip`` on a
requirements file or pull new container images that already have the required
package versions installed.

Docker Sync Troubleshooting tips
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Check your version and make sure you are running 0.4.6 or above:

.. code:: sh

    docker-sync --version

If not, upgrade to the latest version:

.. code:: sh

    gem update docker-sync

If you are having issues with docker sync, try the following:

.. code:: sh

    make stop
    docker-sync stop
    docker-sync clean

Cached Consistency Mode
~~~~~~~~~~~~~~~~~~~~~~~

The performance improvements provided by `cached consistency mode for volume
mounts`_ introduced in Docker CE Edge 17.04 are still not good enough. It's
possible that the "delegated" consistency mode will be enough to no longer need
docker-sync, but this feature hasn't been fully implemented yet (as of
Docker 17.12.0-ce, "delegated" behaves the same as "cached").  There is a
GitHub issue which explains the `current status of implementing delegated consistency mode`_.

.. _Docker Compose: https://docs.docker.com/compose/
.. _Docker for Mac: https://docs.docker.com/docker-for-mac/
.. _Docker for Windows: https://docs.docker.com/docker-for-windows/
.. _Docker Sync: https://github.com/EugenMayer/docker-sync/wiki
.. _Docker Sync installation instructions: https://github.com/EugenMayer/docker-sync/wiki/1.-Installation
.. _cached consistency mode for volume mounts: https://docs.docker.com/docker-for-mac/osxfs-caching/
.. _current status of implementing delegated consistency mode: https://github.com/docker/for-mac/issues/1592
.. _configuring Docker for Mac: https://docs.docker.com/docker-for-mac/#/advanced
.. _feature added in Docker 17.05: https://github.com/edx/configuration/pull/3864
.. _edx-e2e-tests README: https://github.com/edx/edx-e2e-tests/#how-to-run-lms-and-studio-tests
.. _edxops Docker image: https://hub.docker.com/r/edxops/
.. _Docker Hub: https://hub.docker.com/
.. _Pycharm Integration documentation: docs/pycharm_integration.rst
.. _devpi documentation: docs/devpi.rst
.. _edx-platform testing documentation: https://github.com/edx/edx-platform/blob/master/docs/guides/testing/testing.rst#running-python-unit-tests
.. _docker-sync: #improve-mac-osx-performance-with-docker-sync
.. |Build Status| image:: https://travis-ci.org/edx/devstack.svg?branch=master
    :target: https://travis-ci.org/edx/devstack
    :alt: Travis
.. _Docker CI Jenkins Jobs: https://tools-edx-jenkins.edx.org/job/DockerCI
.. _How do I build images?: https://github.com/edx/devstack/tree/master#how-do-i-build-images
   :target: https://travis-ci.org/edx/devstack
.. _Django Migration Don'ts: https://engineering.edx.org/django-migration-donts-f4588fd11b64
.. _Python virtualenv: http://docs.python-guide.org/en/latest/dev/virtualenvs/#lower-level-virtualenv
.. _Running analytics acceptance tests in docker: http://edx-analytics-pipeline-reference.readthedocs.io/en/latest/running_acceptance_tests_in_docker.html
.. _Troubleshooting docker analyticstack: http://edx-analytics-pipeline-reference.readthedocs.io/en/latest/troubleshooting_docker_analyticstack.html
.. _Community: https://open.edx.org/community/connect/
