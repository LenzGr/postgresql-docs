.. _major-upgrade:

====================================================
Upgrading |pdp| from |previous-version| to |version|
====================================================

This document describes the in-place upgrade of |pdp| using the ``pg_upgrade``
tool.
The in-place upgrade means installing a new version without removing the old version and keeping the data files on the server.

.. note::

   ``pg_upgrade`` Documentation:
       https://www.postgresql.org/docs/13/pgupgrade.html


Similar to installing, we recommend you to upgrade |pdp| from |percona| repositories.

.. note::

   A major upgrade is a risky process because of many changes between versions and issues that might occur during or after the upgrade. Therefore, make sure to back up your data first. The backup tools are out of scope of this document. Use the backup tool of your choice.

The general in-place upgrade flow for |pdp| is the following:

1. Install |pdp| |version| packages.
#. Stop the |postgresql| service.
#. Check the upgrade without modifying the data.
#. Upgrade |pdp|.
#. Start |postgresql| service.
#. Execute the  :program:`analyze_new_cluster.sh` script to generate statistics
   so the system is usable.
#. Delete old packages and configuration files.

The exact steps may differ depending on the package manager of your operating system.

.. contents::
   :local:
   :depth: 1

On Debian and Ubuntu using ``apt``
=======================================================

.. note::

   Run **all** commands as root or via |sudo|.

1. Install |pdp| |version| packages.

   * Enable |percona| repository using the |percona-release| utility:

     .. code-block:: bash

        $ sudo percona-release setup ppg-13

   * Install |pdp| |version| package:

     .. code-block:: bash

        $ sudo apt-get install percona-postgresql-13

   * Install the components:

     .. code-block:: bash

        $ sudo apt-get install percona-postgresql-13-repack
        $ sudo apt-get install percona-postgresql-13-pgaudit
        $ sudo apt-get install percona-pgbackrest
        $ sudo apt-get install percona-patroni
        $ sudo apt-get install percona-pg-stat-monitor13
        $ sudo apt-get install percona-pgbadger
        $ sudo apt-get install percona-pgaudit13-set-user
        $ sudo apt-get install percona-pgbadger
        $ sudo apt-get install percona-postgresql-13-wal2json
        $ sudo apt-get install percona-postgresql-contrib

   .. note::

      |percona| Documentation:
         - `Percona Software Repositories Documentation <https://www.percona.com/doc/percona-repo-config/index.html>`_
         - :ref:`pdp.installing`

#. Stop the ``postgresql`` service.

   .. code-block:: bash

      $ sudo systemctl stop postgresql.service

   This stops both |pdp| |previous-version| and |version|.

#. Run the database upgrade.

   * Log in as the ``postgres`` user.

     .. code-block:: bash

        $ sudo su postgres

   * Change the current directory to the :file:`tmp` directory where logs and some scripts will be recorded: :command:`cd tmp/`.

   * Check the ability to upgrade |pdp| from |previous-version| to |version|:

     .. include:: .res/pdp-major-upgrade-check.txt

     .. note:: Sample output

        .. code-block:: text

           Performing Consistency Checks
           -----------------------------
           Checking cluster versions                                   ok
           Checking database user is the install user                  ok
           Checking database connection settings                       ok
           Checking for prepared transactions                          ok
           Checking for reg* data types in user tables                 ok
           Checking for contrib/isn with bigint-passing mismatch       ok
           Checking for tables WITH OIDS                               ok
           Checking for invalid "sql_identifier" user columns          ok
           Checking for presence of required libraries                 ok
           Checking database user is the install user                  ok
           Checking for prepared transactions                          ok

           *Clusters are compatible*

   * Upgrade the |pdp|

     .. include:: .res/pdp-major-upgrade.txt

   * Go back to the regular user: :command:`exit`

   * The |pdp| |previous-version| uses the ``5432`` port while the |pdp| |version| is set up to use the ``5433`` port by default. To start the |pdp| |version|, swap ports in the configuration files of both versions.

     .. include:: .res/pdp-major-ugrade-swap-ports.txt

#. Start the ``postgreqsl`` service.

   .. code-block:: bash

      $ sudo systemctl start postgresql.service

#. Check the ``postgresql`` version.

   .. code-block:: bash

      $ #Log in as a postgres user
      $ sudo su postgres
      $ #Check the database version
      $ psql -c "SELECT version();"

#. Run the :program:`analyze_new_cluster.sh` script

   .. code-block:: bash

      $ tmp/analyze_new_cluster.sh
      $ #Logout
      $ exit

#. Delete |pdp| |previous-version| packages and configuration files

   .. code-block:: bash

      $ #Remove packages
      $ sudo apt-get remove percona-postgresql-12* percona-pgbackrest percona-patroni percona-pg-stat-monitor12 percona-pgaudit12-set-user percona-pgbadger percona-pgbouncer percona-postgresql-12-wal2json
      $ #Remove old files
      $ rm -rf /etc/postgresql/12/main


On |rhel| and |centos| using ``yum``
=======================================================

.. note::

   Run **all** commands as root or via |sudo|.

1. Install |pdp| |version| packages

   * Enable |percona| repository using the |percona-release| utility:

     .. code-block:: bash

        $ sudo percona-release setup ppg-13

   * Install |pdp| |version|:

     .. code-block:: bash


        $ sudo yum install percona-postgresql13-server  
   
   * Install components:
    
     .. code-block:: bash
     
        $ sudo yum install percona-pgaudit
        $ sudo yum install percona-pgbackrest
        $ sudo yum install percona-pg_repack13
        $ sudo yum install percona-patroni
        $ sudo yum install percona-pg-stat-monitor13
        $ sudo yum install percona-pgbadger
        $ sudo yum install percona-pgaudit13_set_user
        $ sudo yum install percona-pgbadger
        $ sudo yum install percona-wal2json13
        $ sudo yum install percona-postgresql13-contrib

     .. note::

      |percona| Documentation:
         - `Percona Software Repositories Documentation <https://www.percona.com/doc/percona-repo-config/index.html>`_
         - :ref:`pdp.installing`

#. Set up |pdp| |version| cluster

   .. code-block:: bash

      $ #Log is as the postgres user
      $ sudo su postgres
      $ #Set up locale settings
      $ export LC_ALL="en_US.UTF-8"
      $ export LC_CTYPE="en_US.UTF-8"
      $ #Initialize cluster with the new data directory
      $ /usr/pgsql-13/bin/initdb -D /var/lib/pgsql/13/data

#. Stop the ``postgresql`` |previous-version| service

   .. code-block:: bash

      $ systemctl stop postgresql-12

#. Run the database upgrade.

   * Log in as the ``postgres`` user

     .. code-block:: bash

        $ sudo su postgres

   * Check the ability to upgrade |pdp| from |previous-version| to |version|:

     .. include:: .res/pdp-major-upgrade-check-rpm.txt

     .. note:: Sample output

        .. code-block:: text

           Performing Consistency Checks
           -----------------------------
           Checking cluster versions                                   ok
           Checking database user is the install user                  ok
           Checking database connection settings                       ok
           Checking for prepared transactions                          ok
           Checking for reg* data types in user tables                 ok
           Checking for contrib/isn with bigint-passing mismatch       ok
           Checking for tables WITH OIDS                               ok
           Checking for invalid "sql_identifier" user columns          ok
           Checking for presence of required libraries                 ok
           Checking database user is the install user                  ok
           Checking for prepared transactions                          ok

           *Clusters are compatible*

   * Upgrade the |pdp|

     .. include:: .res/pdp-major-upgrade-rpm.txt

#. Start the ``postgresql`` |version| service.

   .. code-block:: bash

      $ #Start postgresql service
      $ systemctl start postgresql-13
      $ #Check postgresql status
      $ systemctl status postgresql-13

#. Run the :command:`analyze_new_cluster.sh` script

   .. code-block:: bash

      $ #Log in as the postgres user
      $ sudo su postgres
      $ #Run the script
      $ ./analyze_new_cluster.sh

#. Delete |pdp| |previous-version| configuration files

   .. code-block:: bash

      $ ./delete_old_cluster.sh

#. Delete |pdp| |previous-version| packages

   .. code-block:: bash

      $ #Remove packages
      $ sudo yum -y remove percona-postgresql12*
      $ #Remove old files
      $ rm -rf /var/lib/pgsql/12/data

.. |previous-version| replace:: 12
.. |version| replace:: 13

.. include:: .res/replace.txt
