OME development platform: devspace
==================================

Devspace is a Continuous Integration tool managed by Jenkins CI
providing an automation framework that runs repeated jobs.
The default deployment initializes a Jenkins CI master with
a predefined set of jobs. More information can be found in the
`source code page <https://github.com/openmicroscopy/devspace>`_.

If you find a bug or if you want an additional feature to be implemented,
please `open an issue <https://github.com/openmicroscopy/devspace/issues>`_.

Running and maintaining Devspace requires:

-  Docker engine https://docs.docker.com/.

Optionally a `brief understanding of Ansible <http://docs.ansible.com/ansible/intro_getting_started.html>`_,
`Ansible inventory <http://docs.ansible.com/ansible/intro_inventory.html>`_,
and `Ansible playbooks <http://docs.ansible.com/ansible/playbooks.html>`_.

Installation
------------

Manual
^^^^^^

Install following prerequisites:

-  Docker engine https://docs.docker.com/engine/installation/
-  Docker compose

   ::

      $ pip install docker-compose

Checkout `git repository <https://github.com/openmicroscopy/devspace>`_ and run

::

   $ docker-compose -f docker-compose.yml up --build


OpenStack
^^^^^^^^^

This is an example of how to provision and deploy Devspace using ansible on openstack.
Check out management tools and run:

::

   $ source tenancy.rc
   $ cd infrastructure/ansible
   $ ansible-playbook os-devspace.yml -e vm_name=devspace-test -e vm_key_name=your_key
   $ ansible-playbook -l devspace-test -u centos devspace.yml


To deploy devspace from custom branch, first set up vars:

::

   omero_branch: develop
   snoopy_dir_path: "/path/to/snoopy"

   git_repo: "https://github.com/user_name/devspace.git"
   version: "your_branch"
