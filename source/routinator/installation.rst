.. _doc_routinator_installation:

Installation
============

Getting started with Routinator is really easy using either `Cargo
<https://crates.io/crates/routinator>`_, `Docker
<https://hub.docker.com/r/nlnetlabs/routinator/>`_, or building from the
`source <https://github.com/NLnetLabs/routinator>`_. Binary packages are
`coming soon <https://www.nlnetlabs.nl/pipermail/rpki/2019-May/000049.html>`_.

Quick Start
-----------

Assuming you have rsync and the C toolchain but not yet Rust 1.34 or newer,
here’s how you get Routinator to run as an RTR server listening on 192.0.2.13
port 3323 and an HTTP server listening on port 9556:

.. code-block:: bash

   curl https://sh.rustup.rs -sSf | sh
   source ~/.cargo/env
   cargo install routinator
   routinator init
   # Follow instructions provided
   routinator server --rtr 192.0.2.13:3323 --http 192.0.2.13:9556

If you have an older version of Routinator, you can update via:

.. code-block:: bash

   cargo install -f routinator

If you want to try the master branch from the repository instead of a
release version, you can run:

.. code-block:: bash

   cargo install --git https://github.com/NLnetLabs/routinator.git

Quick Start with Docker
-----------------------

Due to the impracticality of complying with the ARIN TAL distribution terms
in an unsupervised Docker environment, before launching the container it
is necessary to first review and agree to the `ARIN Relying Party Agreement
(RPA) <https://www.arin.net/resources/manage/rpki/tal/>`_. If you
agree to the terms, you can let the Routinator Docker image install the TALs
into a mounted volume that is later reused for the server:

.. code-block:: bash

   # Create a local directory for the RPKI cache
   sudo mkdir -p /etc/routinator/tals
   # Review the ARIN terms.
   # Run a disposable container to install TALs.
   sudo docker run --rm -v /etc/routinator/tals:/root/.rpki-cache/tals \
      nlnetlabs/routinator init -f --accept-arin-rpa
   # Launch the final detached container named 'routinator' exposing RTR on
   # port 3323 and HTTP on port 9556
   sudo docker run -d --name routinator -p 3323:3323 -p 9556:9556 \
      -v /etc/routinator/tals:/root/.rpki-cache/tals nlnetlabs/routinator

System Requirements
-------------------

At this time, the size of the global RPKI data set is about 300MB. Cryptographic
validation of it takes Routinator about 2 seconds on a quad-core i7. 

When choosing a system to run Routinator on, make sure you have 512MB of 
available memory and 1GB of disk space. This will give you ample margin for
the RPKI repositories to grow over time, as adoption increases.

Getting Started
---------------

There are three things you need to install and run Routinator: rsync, a C
toolchain and Rust. You can install Routinator on any system where you can
fulfil these requirements.

You need rsync because most RPKI repositories currently use it as its main
means of distribution. Some of the cryptographic primitives used by
Routinator require a C toolchain. Lastly, you need Rust because that’s the
programming language that Routinator has been written in.

rsync
"""""

Currently, Routinator requires the ``rsync`` executable to be in your path.
Due to the nature of rsync, it is unclear which particular version you need at
the very least, but whatever is being shipped with current Linux and \*BSD
distributions and macOS should be fine. Alternatively, you can download rsync
from `its website <https://rsync.samba.org/>`_.

On Windows, Routinator requires the ``rsync`` version that comes with
`Cygwin <https://www.cygwin.com/>`_ – make sure to select rsync during the
installation phase. 

C Toolchain
"""""""""""

Some of the libraries Routinator depends on require a C toolchain to be
present. Your system probably has some easy way to install the minimum
set of packages to build from C sources. For example, 
``apt-get install gcc`` will install the C toolchain on Debian/Ubuntu.

If you are unsure, try to run ``cc`` on a command line and if there’s a 
complaint about missing input files, you are probably good to go. 

Rust
""""

The Rust compiler runs on, and compiles to, a great number of platforms,
though not all of them are equally supported. The official `Rust 
Platform Support <https://forge.rust-lang.org/platform-support.html>`_
page provides an overview of the various support levels.

While some system distributions include Rust as system packages, 
Routinator relies on a relatively new version of Rust, currently 1.34 or 
newer. We therefore suggest to use the canonical Rust installation via a
tool called ``rustup``.

To install ``rustup`` and Rust, simply do:

.. code-block:: bash

   curl https://sh.rustup.rs -sSf | sh

Alternatively, visit the `official Rust website <https://www.rust-lang.org/tools/install>`_ for other installation methods.

You can update your Rust installation later by running:

.. code-block:: bash

   rustup update

For some platforms, ``rustup`` cannot provide binary releases to install
directly. The `Rust Platform Support
<https://forge.rust-lang.org/platform-support.html>`_ page lists
several platforms where official binary releases are not available,
but Rust is still guaranteed to build. For these platforms, automated 
tests are not run so it’s not guaranteed to produce a working build, but 
they often work to quite a good degree.

One such example that is especially relevant for the routing community
is OpenBSD. On this platform, `patches
<https://github.com/openbsd/ports/tree/master/lang/rust/patches>`_ are 
required to get Rust running correctly, but these are well maintained 
and offer the latest version of Rust quite quickly. 

Rust can be installed on OpenBSD by running:

.. code-block:: bash

   pkg_add rust

Another example where the standard installation method does not work is
CentOS 6, where you will end up with a long list of error messages about 
missing assembler instructions. This is because the assembler shipped with 
CentOS 6 is too old.

You can get the necessary version by installing the `Developer Toolset 
6 <https://www.softwarecollections.org/en/scls/rhscl/devtoolset-6/>`_
from the `Software Collections <https://wiki.centos.org/AdditionalResources/Repositories/SCL>`_ 
repository. On a virgin system, you can install Rust using these steps:

.. code-block:: bash

   sudo yum install centos-release-scl
   sudo yum install devtoolset-6
   scl enable devtoolset-6 bash
   curl https://sh.rustup.rs -sSf | sh
   source $HOME/.cargo/env

Building
--------

The easiest way to get Routinator is to leave it to cargo by saying:

.. code-block:: bash

   cargo install routinator

If you want to try the master branch from the repository instead of a
release version, you can run:

.. code-block:: bash

   cargo install --git https://github.com/NLnetLabs/routinator.git

If you want to update an installed version, you run the same command but
add the ``-f`` flag, a.k.a. force, to approve overwriting the installed
version.

The command will build Routinator and install it in the same directory
that cargo itself lives in, likely ``$HOME/.cargo/bin``. This means 
Routinator will be in your path, too.

Building a Statically Linked Routinator
"""""""""""""""""""""""""""""""""""""""

While Rust binaries are mostly statically linked, they depend on ``libc``
which, as least as ``glibc`` that is standard on Linux systems, is somewhat
difficult to link statically. This is why Routinator binaries are actually
dynamically linked on ``glibc`` systems and can only be transferred between
systems with the same ``glibc`` versions.

However, Rust can build binaries based on the alternative implementation
named musl that can easily be statically linked. Building such binaries is
easy with ``rustup``. You need to install musl and the correct musl target
such as ``x86_64-unknown-linux-musl`` for x86\_64 Linux systems. Then you
can just build Routinator for that target.

On a Debian (and presumably Ubuntu) system, enter the following:

.. code-block:: bash

   sudo apt-get install musl-tools
   rustup target add x86_64-unknown-linux-musl
   cargo build --target=x86_64-unknown-linux-musl --release
