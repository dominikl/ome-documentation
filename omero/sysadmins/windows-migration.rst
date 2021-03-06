Migration from OMERO 5.2 on Windows to OMERO 5.3 on Linux
=========================================================

Backup (Windows)
----------------

- Create a dump of the OMERO SQL database, either using
  `pgAdmin <https://www.pgadmin.org/>`_ or the :command:`pg_dump` command line
  program::

    pg_dump -Fc -f omero52.pg_dump omero_database

- Please keep a pristine copy of your Windows OMERO server, until you are
  sure that the new 5.3 installation works flawlessly. That means if you use
  a network storage system for hosting the 
  :doc:`binary data store <unix/server-binary-repository>`, back it up before
  mounting it onto the new Linux system.

- Save custom configuration properties which might have been set::

    omero config get > omero.config

- For more details on the standard backup procedure, 
  see :doc:`/sysadmins/server-backup-and-restore`.


Install OMERO 5.3 (Linux)
-------------------------

Follow the :doc:`/sysadmins/unix/server-installation` documentation to install
OMERO 5.3 on your Linux system, until (including) step 3 of the Configuration
part (setting the binary repository location).

There is no need to generate the database initialization script, as you are
going to use the previously generated database backup from your old 5.2 
server installation.


Restore Data (Linux)
--------------------

- Restore the 5.2 database dump::

     sudo -u postgres pg_restore -Fc -d omero_database omero52.pg_dump

- Upgrade the database to 5.3 following the :ref:`upgradedb` documentation

- Adjust the paths stored in the database from Windows style to Unix style, 
  e. g. using :command:`psql` command line::

    UPDATE pixels SET path = regexp_replace(path,'\\','/','g');

- Copy the :doc:`binary data store <unix/server-binary-repository>` from your
  Windows system (e.g. :file:`C:\\OMERO`) over to the Linux system 
  (e.g. :file:`/OMERO`), or if you use a network storage system then mount it
  onto the Linux system (**make sure you have a backup beforehand**)

- Restore custom configuration properties (see Backup step 3)::

    omero config load omero.config


Finally start the OMERO server and check the log files for errors.


