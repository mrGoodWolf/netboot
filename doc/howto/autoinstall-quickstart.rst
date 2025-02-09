.. _autoinstall_quickstart:

Autoinstall quick start
***********************

The intent of this page is to provide simple instructions to perform an
autoinstall in a VM on your machine.

This page assumes that you are installing a recent Ubuntu release. However,
for older releases, you can substitute the name of the ISO image but the
instructions should otherwise be the same.

This page also assumes you are on the AMD64 architecture. There is a
:ref:`version for s390x<autoinstall-quickstart-s390x>` too.

Providing the autoinstall data over the network
===============================================

This method is the one that generalises most easily to doing an entirely
network-based installation where a machine boots over a network and then is automatically
installed.

Download the ISO
----------------

Go to the `Ubuntu ISO download page`_ and download the latest Ubuntu
live-server ISO.

Mount the ISO
-------------

.. code-block:: bash

    sudo mount -r ~/Downloads/ubuntu-<release-number>-live-server-amd64.iso /mnt

Where you should change `<release-number>` to match the number of the LTS or
release you have downloaded (e.g., `22.04.3` for Jammy or `23.04` for the Lunar
interim release).

Write your autoinstall configuration
------------------------------------

This means creating cloud-init configuration as follows:

.. code-block:: bash

    mkdir -p ~/www
    cd ~/www
    cat > user-data << 'EOF'
    #cloud-config
    autoinstall:
    version: 1
    identity:
        hostname: ubuntu-server
        password: "$6$exDY1mhS4KUYCE/2$zmn9ToZwTKLhCw.b4/b.ZRTIZM30JZ4QrOQ2aOXJ8yk96xpcCof0kxKwuX1kqLG/ygbJ1f8wxED22bTL4F46P0"
        username: ubuntu
    EOF
    touch meta-data

The encrypted password is ``ubuntu``.

Serve the cloud-init configuration over HTTP
--------------------------------------------

Leave this running in one terminal window:

.. code-block:: bash

    cd ~/www
    python3 -m http.server 3003

Create a target disk
--------------------

.. code-block::

    truncate -s 10G image.img

Run the installation
--------------------

As before, you will need to change `<release-number>` in the following command
to match the release ISO you downloaded.

.. code-block:: bash

    kvm -no-reboot -m 2048 \
        -drive file=image.img,format=raw,cache=none,if=virtio \
        -cdrom ~/Downloads/ubuntu-<release-number>-live-server-amd64.iso \
        -kernel /mnt/casper/vmlinuz \
        -initrd /mnt/casper/initrd \
        -append 'autoinstall ds=nocloud-net;s=http://_gateway:3003/'

This will boot, download the configuration from the server (set up in the previous
step) and run the installation. The installer reboots at the end but the
``-no-reboot`` flag to ``kvm`` means that ``kvm`` will exit when this happens.
It should take about 5 minutes.

Boot the installed system
-------------------------

.. code-block:: bash

    kvm -no-reboot -m 2048 \
        -drive file=image.img,format=raw,cache=none,if=virtio

This will boot into the freshly installed system and you should be able to log
in as ``ubuntu/ubuntu``.

Using another volume to provide the autoinstall configuration
=============================================================

This is the method to use when you want to create media that you can just plug
into a system to have it be installed.

Download the live-server ISO
----------------------------

Go to the `Ubuntu ISO download page`_ and download the latest Ubuntu
live-server ISO.

Create your user-data and meta-data files
-----------------------------------------

.. code-block:: bash

    mkdir -p ~/cidata
    cd ~/cidata
    cat > user-data << 'EOF'
    #cloud-config
    autoinstall:
    version: 1
    identity:
        hostname: ubuntu-server
        password: "$6$exDY1mhS4KUYCE/2$zmn9ToZwTKLhCw.b4/b.ZRTIZM30JZ4QrOQ2aOXJ8yk96xpcCof0kxKwuX1kqLG/ygbJ1f8wxED22bTL4F46P0"
        username: ubuntu
    EOF
    touch meta-data

The encrypted password is ``ubuntu``.

Create an ISO to use as a cloud-init data source
------------------------------------------------

.. code-block:: bash

    sudo apt install cloud-image-utils
    cloud-localds ~/seed.iso user-data meta-data

Create a target disk
--------------------

.. code-block:: bash

    truncate -s 10G image.img

Run the installation
--------------------

As before, you will need to change `<release-number>` in the following command
to match the release ISO you downloaded.

.. code-block:: bash

    kvm -no-reboot -m 2048 \
        -drive file=image.img,format=raw,cache=none,if=virtio \
        -drive file=~/seed.iso,format=raw,cache=none,if=virtio \
        -cdrom ~/Downloads/ubuntu-<release-number>-live-server-amd64.iso

This boots the system and runs the installation. Unless you interrupt boot to add
``autoinstall`` to the kernel command line, the installer prompts for
confirmation before touching the disk.

The installer reboots at the end but the ``-no-reboot`` flag to ``kvm`` means
that ``kvm`` will exit when this happens.

The whole process should take about 5 minutes.

Boot the installed system
-------------------------

.. code-block:: bash

    kvm -no-reboot -m 2048 \
        -drive file=image.img,format=raw,cache=none,if=virtio

This will boot into the freshly installed system and you should be able to log
in as ``ubuntu/ubuntu``.

.. LINKS

.. _Ubuntu ISO download page: https://releases.ubuntu.com/
