.. include:: links.rst

.. _quickstart-releaseprocess:

Quickstart: uploading, testing, pushing releases 
------------------------------------------------

This quickstart document walks you through setting up a self-contained
pypi release upload, testing and staging system for your Python packages.

Installing devpi client and server
++++++++++++++++++++++++++++++++++

We want to run the full devpi system on our laptop::

    pip install -U devpi-web devpi-client

Note that the ``devpi-web`` package will pull in the core
``devpi-server`` package.  If you don't want a web interface you 
can just install the latter only.

initializing a basic server and index
+++++++++++++++++++++++++++++++++++++

We need to perform a couple of steps to get an index
where we can upload and test packages:

- start a background devpi-server at ``http://localhost:3141``

- configure the client-side tool ``devpi`` to connect to the newly
  started server

- create and login a user, using as defaults your current login name 
  and an empty password.

- create an index and directly use it.

So let's first start a background server::

    $ devpi-server --start --init
    2017-11-23 14:32:00,289 INFO  NOCTX Loading node info from /tmp/home/.devpi/server/.nodeinfo
    2017-11-23 14:32:00,290 INFO  NOCTX generated uuid: f68e55fb507b4811a2b581747fc686f2
    2017-11-23 14:32:00,291 INFO  NOCTX wrote nodeinfo to: /tmp/home/.devpi/server/.nodeinfo
    2017-11-23 14:32:00,293 INFO  NOCTX DB: Creating schema
    2017-11-23 14:32:00,302 INFO  [Wtx-1] setting password for user 'root'
    2017-11-23 14:32:00,302 INFO  [Wtx-1] created user 'root' with email None
    2017-11-23 14:32:00,302 INFO  [Wtx-1] created root user
    2017-11-23 14:32:00,302 INFO  [Wtx-1] created root/pypi index
    2017-11-23 14:32:00,305 INFO  [Wtx-1] fswriter0: committed: keys: 'root/.config','.config'
    starting background devpi-server at http://localhost:3141
    /tmp/home/.devpi/server/.xproc/devpi-server$ /home/devpi/devpi/bin/devpi-server
    process 'devpi-server' started pid=43735
    devpi-server process startup detected
    logfile is at /tmp/home/.devpi/server/.xproc/devpi-server/xprocess.log

Then we point the devpi client to it::

    $ devpi use http://localhost:3141
    using server: http://localhost:3141/ (not logged in)
    no current index: type 'devpi use -l' to discover indices
    ~/.pydistutils.cfg     : no config file exists
    ~/.pip/pip.conf        : no config file exists
    ~/.buildout/default.cfg: no config file exists
    always-set-cfg: no

Then we add our own "testuser"::

    $ devpi user -c testuser password=123
    user created: testuser

Then we login::

    $ devpi login testuser --password=123
    logged in 'testuser', credentials valid for 10.00 hours

And create a "dev" index, telling it to use the ``root/pypi`` cache as a base
so that all of pypi.python.org packages will appear on that index::

    $ devpi index -c dev bases=root/pypi
    http://localhost:3141/testuser/dev:
      type=stage
      bases=root/pypi
      volatile=True
      acl_upload=testuser
      mirror_whitelist=
      pypi_whitelist=

Finally we use the new index::

    $ devpi use testuser/dev
    current devpi index: http://localhost:3141/testuser/dev (logged in as testuser)
    ~/.pydistutils.cfg     : no config file exists
    ~/.pip/pip.conf        : no config file exists
    ~/.buildout/default.cfg: no config file exists
    always-set-cfg: no

We are now ready to go for uploading and testing packages.

.. _`quickstart_release_steps`:

devpi install: installing a package
+++++++++++++++++++++++++++++++++++

We can now use the ``devpi`` command line client to trigger a ``pip
install`` of a pypi package using the index from our already running server::

    $ devpi install pytest
    -->  /home/devpi/devpi/doc$ /tmp/docenv/bin/pip install -U -i http://localhost:3141/testuser/dev/+simple/ pytest  [PIP_USE_WHEEL=1,PIP_PRE=1]
    Collecting pytest
      Downloading http://localhost:3141/root/pypi/+f/9c8/ab8b299cdaa6a/pytest-3.2.5-py2.py3-none-any.whl (188kB)
    Requirement already up-to-date: py>=1.4.33 in /private/tmp/docenv/lib/python3.4/site-packages (from pytest)
    Requirement already up-to-date: setuptools in /private/tmp/docenv/lib/python3.4/site-packages (from pytest)
    Installing collected packages: pytest
    Successfully installed pytest-3.2.5

The ``devpi install`` command configured a pip call, using the
pypi-compatible ``+simple/`` page on our ``testuser/dev`` index for
finding and downloading packages.  The ``pip`` executable was searched
in the ``PATH`` and found in ``docenv/bin/pip``.

Let's check that ``pytest`` was installed correctly::

    $ py.test --version
    This is pytest version 3.2.5, imported from /tmp/docenv/lib/python3.4/site-packages/pytest.py

You may invoke the ``devpi install`` command a second time which will
even work when you have no network.

.. _`devpi upload`:

devpi upload: uploading one or more packages
++++++++++++++++++++++++++++++++++++++++++++

We are going to use ``devpi`` command line tool facilities for 
performing uploads (you can also 
:ref:`use plain setup.py <configure pypirc>`).

Let's verify we are logged in to the correct index::

    $ devpi use
    current devpi index: http://localhost:3141/testuser/dev (logged in as testuser)
    ~/.pydistutils.cfg     : no config file exists
    ~/.pip/pip.conf        : no config file exists
    ~/.buildout/default.cfg: no config file exists
    always-set-cfg: no

Now go to the directory of a ``setup.py`` file of one of your projects  
(we assume it is named ``example``) to build and upload your package
to our ``testuser/dev`` index::

    example $ devpi upload
    using workdir /tmp/devpi0
    copied repo /home/devpi/devpi/.git to /tmp/devpi0/upload/devel/.git
    pre-build: cleaning /home/devpi/devpi/doc/example/dist
    -->  /tmp/devpi0/upload/devel/doc/example$ /tmp/docenv/bin/python setup.py sdist --formats gztar 
    built: /home/devpi/devpi/doc/example/dist/example-1.0.tar.gz [SDIST.TGZ] 0.709kb
    register example-1.0 to http://localhost:3141/testuser/dev/
    file_upload of example-1.0.tar.gz to http://localhost:3141/testuser/dev/

There are three triggered actions:

- detection of a mercurial repository, leading to copying all versioned
  files to a temporary work dir.  If you are not using mercurial,
  the copy-step is skipped and the upload operates directly on your source
  tree.

- registering the ``example`` release as defined in ``setup.py`` to 
  our current index

- building and uploading a ``gztar`` formatted release file from the
  workdir to the current index (using a ``setup.py`` invocation under
  the hood).

We can now install the freshly uploaded package::

    $ devpi install example
    -->  /home/devpi/devpi/doc$ /tmp/docenv/bin/pip install -U -i http://localhost:3141/testuser/dev/+simple/ example  [PIP_PRE=1,PIP_USE_WHEEL=1]
    Collecting example
      Downloading http://localhost:3141/testuser/dev/+f/469/9bae162d0b05b/example-1.0.tar.gz
    Building wheels for collected packages: example
      Running setup.py bdist_wheel for example: started
      Running setup.py bdist_wheel for example: finished with status 'done'
      Stored in directory: /tmp/home/Library/Caches/pip/wheels/b1/df/a7/096e13a08929e6947b6e4b4f5cabf149dfc962dcb83c63f0b9
    Successfully built example
    Installing collected packages: example
    Successfully installed example-1.0

This installed your just uploaded package from the ``testuser/dev``
index where we previously uploaded the package.

.. note::

    ``devpi upload`` allows to simultanously upload multiple different 
    formats of your release files such as ``sdist.zip`` or ``bdist_egg``.
    The default is ``sdist.tgz``.



devpi test: testing an uploaded package
+++++++++++++++++++++++++++++++++++++++

If you have a package which uses tox_ for testing you may now invoke::

    $ devpi test example  # package needs to contain tox.ini
    received http://localhost:3141/testuser/dev/+f/469/9bae162d0b05b/example-1.0.tar.gz
    unpacking /tmp/devpi-test0/downloads/example-1.0.tar.gz to /tmp/devpi-test0/targz
    /tmp/devpi-test0/targz/example-1.0$ tox --installpkg /tmp/devpi-test0/downloads/example-1.0.tar.gz -i ALL=http://localhost:3141/testuser/dev/+simple/ --recreate --result-json /tmp/devpi-test0/targz/toxreport.json -c /tmp/devpi-test0/targz/example-1.0/tox.ini
    python create: /tmp/devpi-test0/targz/example-1.0/.tox/python
    python installdeps: pytest
    python inst: /tmp/devpi-test0/downloads/example-1.0.tar.gz
    python installed: example==1.0,py==1.5.2,pytest==3.2.5
    python runtests: PYTHONHASHSEED='1246855891'
    python runtests: commands[0] | py.test
    ============================= test session starts ==============================
    platform darwin -- Python 3.4.6, pytest-3.2.5, py-1.5.2, pluggy-0.4.0
    rootdir: /tmp/devpi-test0/targz/example-1.0, inifile:
    collected 1 item
    
    test_example.py .
    
    =========================== 1 passed in 0.01 seconds ===========================
    ___________________________________ summary ____________________________________
      python: commands succeeded
      congratulations :)
    wrote json report at: /tmp/devpi-test0/targz/toxreport.json
    posting tox result data to http://localhost:3141/testuser/dev/+f/469/9bae162d0b05b/example-1.0.tar.gz
    successfully posted tox result data

Here is what happened:

- devpi got the latest available version of ``example`` from the current
  index

- it unpacked it to a temp dir, found the ``tox.ini`` and then invoked
  tox, pointing it to our ``example-1.0.tar.gz``, forcing all installations
  to go through our current ``testuser/dev/+simple/`` index and instructing
  it to create a ``json`` report.

- after all tests ran, we send the ``toxreport.json`` to the devpi server
  where it will be attached precisely to our release file.
 
We can verify that the test status was recorded via::

    $ devpi list example
    http://localhost:3141/testuser/dev/+f/469/9bae162d0b05b/example-1.0.tar.gz
    trubel.local darwin  python     3.4.6 tests passed

.. versionadded:: 2.6

With ``--index`` you can get the release from another index. Full URLs to
another devpi-server are also supported.

.. note::

    Since version 2.2.0 testing of universal wheels is supported if there
    also is an sdist which contains the neccessary tox.ini and tests files.
    Wheels typically don't contain them as they are a pure installation
    package.

devpi push: staging a release to another index
++++++++++++++++++++++++++++++++++++++++++++++

Once you are happy with a release file you can push it either
to another devpi-managed index or to an outside pypi index server.

Let's create another ``staging`` index::

    $ devpi index -c staging volatile=False
    http://localhost:3141/testuser/staging:
      type=stage
      bases=
      volatile=False
      acl_upload=testuser
      mirror_whitelist=
      pypi_whitelist=

We created a non-volatile index which means that one can not 
overwrite or delete release files. See :ref:`non_volatile_indexes` for more info
on this setting.

We can now push the ``example-1.0.tar.gz`` from above to
our ``staging`` index::

    $ devpi push example==1.0 testuser/staging
       200 register example 1.0 -> testuser/staging
       200 store_releasefile testuser/staging/+f/469/9bae162d0b05b/example-1.0.tar.gz
       200 store_toxresult testuser/staging/+f/469/9bae162d0b05b/example-1.0.tar.gz.toxresult0

This will determine all files on our ``testuser/dev`` index belonging to
the specified ``example==1.0`` release and copy them to the
``testuser/staging`` index. 

devpi push: releasing to an external index
++++++++++++++++++++++++++++++++++++++++++

Let's check again our current index::

    $ devpi use
    current devpi index: http://localhost:3141/testuser/dev (logged in as testuser)
    ~/.pydistutils.cfg     : no config file exists
    ~/.pip/pip.conf        : no config file exists
    ~/.buildout/default.cfg: no config file exists
    always-set-cfg: no

Let's now use our ``testuser/staging`` index::

    $ devpi use testuser/staging
    current devpi index: http://localhost:3141/testuser/staging (logged in as testuser)
    ~/.pydistutils.cfg     : no config file exists
    ~/.pip/pip.conf        : no config file exists
    ~/.buildout/default.cfg: no config file exists
    always-set-cfg: no

and check the test result status again::

    $ devpi list example
    http://localhost:3141/testuser/staging/+f/469/9bae162d0b05b/example-1.0.tar.gz
    trubel.local darwin  python     3.4.6 tests passed

Good, the test result status is still available after the push
from the last step.

We may now decide to push this release to an external
pypi-style index which we have configured in the ``.pypirc`` file::

    $ devpi push example-1.0 pypi:testrun
    no pypirc file found at: /tmp/home/.pypirc

this will push all release files of the ``example-1.0`` release
to the external ``testrun`` index server, using credentials
and the URL found in the ``pypi`` section in your
``.pypirc``.

index inheritance re-configuration
++++++++++++++++++++++++++++++++++

At this point we have the ``example-1.0`` release and release file
on both the ``testuser/dev`` and ``testuser/staging`` indices.
If we rather want to always use staging packages in our development
index, we can reconfigure the inheritance 
``bases`` for ``testuser/dev``::

    $ devpi index testuser/dev bases=testuser/staging
    /testuser/dev changing bases: ['testuser/staging']
    http://localhost:3141/testuser/dev:
      type=stage
      bases=testuser/staging
      volatile=True
      acl_upload=testuser
      mirror_whitelist=
      pypi_whitelist=

If we now switch back to using ``testuser/dev``::

    $ devpi use testuser/dev
    current devpi index: http://localhost:3141/testuser/dev (logged in as testuser)
    ~/.pydistutils.cfg     : no config file exists
    ~/.pip/pip.conf        : no config file exists
    ~/.buildout/default.cfg: no config file exists
    always-set-cfg: no

and look at our example release files::

    $ devpi list example
    http://localhost:3141/testuser/dev/+f/469/9bae162d0b05b/example-1.0.tar.gz
    trubel.local darwin  python     3.4.6 tests passed
    http://localhost:3141/testuser/staging/+f/469/9bae162d0b05b/example-1.0.tar.gz
    trubel.local darwin  python     3.4.6 tests passed

we'll see that ``example-1.0.tar.gz`` is contained in both
indices.  Let's remove the ``testuser/dev`` ``example`` release::

    $ devpi remove -y example
    About to remove the following releases and distributions
    version: 1.0
      - http://localhost:3141/testuser/dev/+f/469/9bae162d0b05b/example-1.0.tar.gz
      - http://localhost:3141/testuser/dev/+f/469/9bae162d0b05b/example-1.0.tar.gz.toxresult0
    Are you sure (yes/no)? yes (autoset from -y option)
    deleting release 1.0 of example

If you don't specify the ``-y`` option you will be asked to confirm
the delete operation interactively.

The ``example-1.0`` release remains accessible through ``testuser/dev``
because it inherits all releases from its ``testuser/staging`` base::

    $ devpi list example
    http://localhost:3141/testuser/staging/+f/469/9bae162d0b05b/example-1.0.tar.gz
    trubel.local darwin  python     3.4.6 tests passed

::

    $ devpi-server --stop
    2017-11-23 14:32:29,166 INFO  NOCTX Loading node info from /tmp/home/.devpi/server/.nodeinfo
    2017-11-23 14:32:29,166 INFO  NOCTX wrote nodeinfo to: /tmp/home/.devpi/server/.nodeinfo
    killed server pid=43735

running devpi-server permanently
+++++++++++++++++++++++++++++++++

If you want to configure a permanent devpi-server install,
you can go to :ref:`quickstart-server` to get some help.


