.. _user-guide:

op-test User Guide
==================

.. _commandline:

Command Line Options
--------------------

All configuration options can be specified via the command line or in a
configuration file (see :ref:`config-file`).

.. argparse::
   :module: OpTestConfiguration
   :func: get_parser
   :prog: op-test

.. _config-file:

Configuration Files and Command Line Options
--------------------------------------------

When working with ``op-test``, you can specify every configuration option on
the command line, or you can save some in a configuration file. Typically,
you may keep a configuration file per-machine you test against, to save on
typing out all the IP addresses and login credentials.

For example, this ``witherspoon.conf`` file will connect to a Witherspoon
machine:

.. code-block:: ini

  [op-test]
  bmc_type=OpenBMC
  bmc_ip=10.0.0.1
  bmc_username=root
  bmc_password=0penBmc
  host_ip=10.0.0.2
  host_user=root
  host_password=abc123

It can be used as such:

.. code-block:: bash

  ./op-test --config-file witherspoon.conf

Other options can also be specified on the command line and in configuration
files as needed for your environment.

There is also a *per user* configuration file, at ``~/.op-test-framework.conf``
where you can store global options, for example:

.. code-block:: ini

  [op-test]
  os_cdrom=~/ubuntu-18.04-ppc64el.iso

Configuration file keys use the long option name with ``-`` replaced by ``_``.
For example, ``--os-cdrom`` becomes ``os_cdrom`` and ``--host-pnor`` becomes
``host_pnor``.

This per user configuration file will be loaded in *addition* to the config
file provided on the command line.

Common workflows
^^^^^^^^^^^^^^^^

The parser-generated command line reference above is the authoritative list of
supported options. In practice, the most commonly used groups are:

* test selection: ``--run-suite``, ``--run``, ``--list-suites``,
  ``--list-tests``, ``--list-all-tests``, ``--failfast``, ``--quiet``
* output and logging: ``--output``, ``--logdir``, ``--suffix``
* machine and BMC selection: ``--bmc-type``, ``--machine-state``,
  ``--bmc-ip``, ``--bmc-username``, ``--bmc-password``,
  ``--bmc-usernameipmi``, ``--bmc-passwordipmi``
* host access: ``--host-ip``, ``--host-user``, ``--host-password``,
  ``--host-serial-console-command``
* install and OS media: ``--os-cdrom``, ``--os-repo``, ``--no-os-reinstall``
* SSH handling: ``--check-ssh-keys``, ``--known-hosts-file``

Additional option groups exist for HMC/LPAR systems, secure/trusted boot,
kernel command line manipulation, kdump, git-based kernel workflows, and
machine-specific automation.

Secure and trusted boot options
-------------------------------

For secure and trusted boot validation, ``op-test`` also supports:

* ``--un-signed-pnor``
* ``--signed-pnor``
* ``--signed-to-pnor``
* ``--key-transition-pnor``
* ``--test-container PART FILE``
* ``--secure-mode``
* ``--trusted-mode``

HMC, PHYP, and LPAR options
---------------------------

For ``FSP_PHYP`` and ``EBMC_PHYP`` environments, additional HMC and LPAR
options are available, including:

* ``--hmc-ip``, ``--hmc-username``, ``--hmc-password``
* ``--system-name``, ``--lpar-name``, ``--lpar-prof``, ``--lpar-vios``
* ``--target-system-name``, ``--target-lpar-name``

These options are typically used when managing partitions through an HMC rather
than interacting with a standalone OpenPOWER host.

op-test and Qemu
----------------

You can use the ``qemu`` BMC type to run many tests using the qemu simulator.
This can be useful for test development/debug as well as testing the qemu
simulator itself.

It may be useful to keep a configuration file with your qemu configuration
in it for running tests. An example of such a configuration file is below:

.. code-block:: ini

  [op-test]
  bmc_type=qemu
  qemu_binary=~/qemu/ppc64-softmmu/qemu-system-ppc64
  host_pnor=palmetto.pnor
  flash_skiboot=~/skiboot/skiboot.lid
  flash_kernel=zImage.epapr
  flash_initramfs=rootfs.cpio
  host_user=ubuntu
  host_password=abc123
  os_cdrom=osimages/ubuntu-17.10-server-ppc64el.iso

Note that for ``qemu`` we want the *uncompressed* ``skiboot.lid`` for qemu to
load, and while it's not *required*, using the uncompressed ``rootfs.cpio``
does *significantly* improve boot time to Petitboot.

In this configuration file example, we point to a qemu development tree
rather than using the system default ``qemu-system-ppc64`` binary. We also
specify a ``--host-pnor`` file so that qemu can mount an NVRAM partition.

To run the "boot to petitboot" test in qemu with the above configuration file,
you can do so like this:

.. code-block:: bash

  ./op-test --config-file qemu.conf \
  --run testcases.BasicIPL.BootToPetitbootShell

Not all tests currently pass in ``qemu``, and running tests in ``qemu`` should
be considered somewhat experimental.

Other supported environments
----------------------------

In addition to ``AMI``, ``SMC``, ``FSP``, ``OpenBMC``, and ``qemu``, the parser
also supports ``FSP_PHYP``, ``EBMC_PHYP``, and ``mambo`` environments. These
are primarily intended for partitioned systems, enterprise BMC flows, and
simulator-based development.

Advanced workflows
------------------

Several advanced workflows are supported and are documented primarily through
the parser-generated option reference:

* git-based kernel workflows:
  ``--git-repo``, ``--git-repo-reference``, ``--git-repoconfigpath``,
  ``--git-repoconfig``, ``--git-branch``, ``--git-home``, ``--git-patch``,
  ``--use-kexec``, ``--append-kernel-cmdline``, ``--use-reboot``,
  ``--bad-commit``, ``--good-commit``, ``--bisect-script``,
  ``--bisect-category``, ``--bisect-flag``
* host command execution:
  ``--host-cmd``, ``--host-cmd-file``, ``--host-cmd-timeout``,
  ``--host-cmd-resultpath``, ``--machine-config``
* kernel command line manipulation:
  ``--add-kernel-args``, ``--remove-kernel-args``
* kdump and crash collection:
  ``--dump-server-ip``, ``--dump-server-pw``, ``--dump-path``
When in doubt, prefer the parser-generated command line reference at the top of
this document, since it is derived directly from ``OpTestConfiguration``.
