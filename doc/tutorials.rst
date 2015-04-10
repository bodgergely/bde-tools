=========
Tutorials
=========

.. _tutorials-build_bde:

Use waf to Build BDE
====================

.. note::

   The following instruction assumes that you are running a unix
   platform. Building on windows is almost equivalent, except you must use the
   equivalent commands in windows command prompt and call ``waf.bat`` instead
   ``waf``. For more details, please see :ref:`waf-windows`.

First, clone the bde and bde-tools repositores from `github
<https://github.com/bloomberg/bde>`_:

::

   $ git clone https://github.com/bloomberg/bde.git
   $ git clone https://github.com/bloomberg/bde-tools.git

Then, add the ``<bde-tools>/bin`` to the ``PATH`` environment variable:

::

   $ export PATH=<bde-tools>/bin:$PATH

.. note::

   Instead of adding ``bde-tools/bin`` to your ``PATH``, you can also execute
   the scripts in ``bde-tools/bin`` directly.

Now, go to the root of the bde repository and build the libraries in it:

::

    $ cd <bde>
    $ waf configure
    $ waf build --target bslstl  # build just the bslstl package
    $ waf build  # build all libraries
    $ waf build --test build  # build all test drivers
    $ waf build --test run  # run all test drivers

The linked libraries, test drivers, and other build artifacts should be in the
build output directory, which by default is just "build".

See :ref:`waf-top` for detailed reference.

.. _tutorials-setwafenv-bde:

Use bde_setwafenv.py to Build BDE
=================================

First, specify the compilers available on your system in a
``~/bdecompilerconfig``.  Here is an example:

::

   [
        {
            "hostname": ".*",
            "uplid": "unix-linux-",
            "compilers": [
                {
                    "type": "gcc",
                    "c_path": "/opt/swt/install/gcc-4.7.2/bin/gcc",
                    "cxx_path": "/opt/swt/install/gcc-4.7.2/bin/g++",
                    "version": "4.7.2"
                },
                {
                    "type": "gcc",
                    "c_path": "/opt/swt/install/gcc-4.3.5/bin/gcc",
                    "cxx_path": "/opt/swt/install/gcc-4.3.5/bin/g++",
                    "version": "4.3.5"
                },
                {
                    "type": "gcc",
                    "c_path": "/usr/bin/gcc",
                    "cxx_path": "/usr/bin/g++",
                    "version": "4.1.2"
                }
            ]
        }
   ]

See :ref:`setwafenv-compiler_config` for more details.

Then, follow the instructions from :ref:`tutorials-build_bde` to checkout bde
and bde-tools and add bde-tools/bin to your PATH.

Next, use ``bde_setwafenv.py`` to set up the environment variables:

::

   $ eval $(bde_setwafenv.py -i /tmp/bde-install -t dbg_mt_exc_64 -c gcc-4.7.2)

   using configuration: /home/che2/.bdecompilerconfig
   using compiler: gcc-4.7.2
   using ufid: dbg_exc_mt_64
   using install directory: /tmp/bde-install

.. note::

   Here we choose to use :ref:`ufid` to specify the build configuration.  You
   can also use the :ref:`qualified configuration options
   <waf-qualified_build_config>`.

The actual environment variables being set will depend on your machine's
platform :ref:`uplid`. On my machine, the following Bourne shell commands are
evaluated to set the environment variables:

::

   export BDE_WAF_UPLID=unix-linux-x86_64-3.2.0-gcc-4.7.2
   export BDE_WAF_UFID=dbg_exc_mt_64
   export BDE_WAF_BUILD_DIR="unix-linux-x86_64-3.2.0-gcc-4.7.2-dbg_exc_mt_64"
   export WAFLOCK=".lock-waf-unix-linux-x86_64-3.2.0-gcc-4.7.2-dbg_exc_mt_64"
   export CXX=/usr/bin/g++
   export CC=/usr/bin/gcc
   export PREFIX="/tmp/bde-install/unix-linux-x86_64-3.2.0-gcc-4.7.2-dbg_exc_mt_64"
   export PKG_CONFIG_PATH="/tmp/bde-install/unix-linux-x86_64-3.2.0-gcc-4.7.2-dbg_exc_mt_64/lib/pkgconfig"
   unset BDE_WAF_COMP_FLAGS

Then, build bde using waf:

::

   $ cd <bde>
   $ waf configure build

See :ref:`setwafenv-top` for detailed reference.

.. _tutorials-setwafenv-bde-app:

Use bde_setwafenv.py to Build an Application on Top of BDE
==========================================================

First, follow :ref:`tutorials-setwafenv-bde` to create ``~/bdecompilerconfig``,
set up the environment variables using bde_setwafenv.py, and build BDE.

Then, install bde:

::

   $ cd <bde>
   $ waf install

On my machine, the headers, libraries, and pkg-config files are installed to
``/tmp/bde-install/unix-linux-x86_64-3.2.0-gcc-4.7.2-dbg_exc_mt_64``:

::

   /tmp/bde-install/unix-linux-x86_64-3.2.0-gcc-4.7.2-dbg_exc_mt_64
   |
   |-- include
   |   |
   |   `-- ...  <-- header files
   |
   `-- lib
    |
    |-- libbdl.a
    |-- libbsl.a
    |-- libdecnumber.a
    |-- libinteldfp.a
    `-- pkgconfig
        |
        |-- bdl.pc
        |-- bsl.pc
        |-- decnumber.pc
        `-- inteldfp.pc

Next, create a new repository containing the application that we are going to
be building.

::

   $ mkdir testrepo
   $ cd testrepo
   $ cp <bde-tools>/etc/wscript .  # wscript is required for using waf

Then, create the following directory and file structure in the repo
(see :ref:`bde_repo-physical_layout` for more details):

::

   testrepo
   |
   |-- wscript
   `-- applications
      |
      `-- myapp
          |
          |-- myapp.m.cpp
          `-- package
              |
              |-- myapp.dep
              `-- myapp.mem

Contents of myapp.m.cpp:

::

    #include <bsl_vector.h>
    #include <bsl_iostream.h>

    int main(int, char *[])
    {
        bsl::vector<int> v;

        v.push_back(3);
        v.push_back(2);
        v.push_back(5);

        for (bsl::vector<int>::const_iterator iter = v.begin();
            iter != v.end();
            ++iter) {
            bsl::cout << *iter << bsl::endl;
        }

        return 0;
    }

Contents of myapp.dep:

::

   bsl # we depend on bsl

``myapp.mem`` should be empty because myapp doesn't contain any components
except the ``.m.cpp``, which is implicitly included in an application package.

Now, we can build this application using waf:

::

   $ cd <testrepo>
   $ waf configure
   $ waf build

.. _tutorials-setwafenv-bde-windows:

Use bde_setwafenv.py to Build BDE on Windows
============================================

bde_setwafenv.py can be used on Windows through cygwin.

**Prerequisites**:

- `cygwin <https://www.cygwin.com/>`_
- Windows version of Python 2.6, 2.7, or 3.3+

First, make sure you have cloned the bde and bde-tools repositories, and that
you have added ``bde-tools/bin`` to your system's PATH.

Next, in cygwin, run the following command to set the environment variables for waf:

::

   $ bde_setwafenv.py list  # list available compilers on windows
   $ eval $(bde_setwafenv.py -i ~/tmp/bde-install -c cl-18.00) # use visual studio 2013

.. note::

   On Windows, bde_setwafenv.py does not use ``~/.bdecompilerconfig``. Instead
   it uses a list of hard coded available compilers on windows and do not check
   those compilers are available. It is your job to make sure that you are
   using a installed Visual Studio compiler.


Now, you can build bde using ``waf.bat`` in cygwin:

::

   $ cd <bde>
   $ waf.bat configure
   $ waf.bat build

.. important::

   Even though bde_setwafenv.py must be invoked on cygwin on Windows, cygwin
   itself is not a supported build platform by :ref:`waf-top`.  Once
   bde_setwafenv.py is call in cygwin, ``bde-tools/bin/waf.bat`` must be used
   instead of calling ``waf`` directly. ``waf.bat`` will invoke ``waf`` using
   the windows version of Python and build using the Visual Studio C/C++
   compiler.  You can download a free version of Visual Studio Express from
   `Microsoft
   <https://www.visualstudio.com/en-us/products/visual-studio-express-vs.aspx>`_.

.. TODO: Building an Library That Does Not Depend on BDE
.. TODO: Building an Application That Does Not Depend on BDE