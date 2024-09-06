.. cli:

Command Line Interface
====================

.. contents:: Table of Contents

.. list-table:: Summary of Catplat CLI Sub-commands
   :widths: 25 25 50
   :header-rows: 1

   * - Sub-command 1
     - Sub-command 2
     - Description
   * - analyze
     - adsorbate
     - Analyze adsorbate structure.
   * - analyze
     - bulk
     - Analyze bulk structure.
   * - analyze
     - complex
     - Analyze complex structure.
   * - analyze
     - miller
     - Analyze bulk structure to obtain facet-like miller indices.
   * - analyze
     - sites
     - Analyze adsorption sites of a slab.
   * - analyze
     - slab
     - Analyze slab structure.
   * - build
     - adsorbate
     - Create/Write adsorbate structures.
   * - build
     - bulk
     - Create/Write bulk structures.
   * - build
     - complex
     - Create/Write complex structures.
   * - build
     - multicomplex
     - Create/Write multi-adsorbate complex structures.
   * - build
     - slab
     - Create/Write slab structures.
   * - calculate
     - \-
     - Run Catplat workflow calculations.
   * - config
     - \-
     - Set up new project/view current project settings.
   * - database
     - backup
     - Backup database to file.
   * - dryrun
     - \-
     - Trial run for catplat calculate.
   * - entry
     - delete
     - Delete selected entries.
   * - entry
     - delete-key
     - Delete attribute keys.
   * - entry
     - ignore
     - Ignore selected entries.
   * - entry
     - update-key
     - Update existing values.
   * - entry
     - view
     - View data from selected entries.
   * - entry
     - write
     - Write data from selected entries.
   * - extract
     - \-
     - Extract calculation information from folders.
   * - laspinit
     - \-
     - Create Initial/Transition/Final structures for LASP-CBD.
   * - retrieve
     - \-
     - Queries the database.
   * - test
     - db
     - Test database integrity.
   * - test
     - self
     - Quick test for tunnel and db integrity.
   * - test
     - tunnel
     - Test sql tunnel.

Help
--------------------

You may get more details of the options available by running the command with the ``--help`` option.

.. code-block:: console

    $ catplat --help
    $ catplat sub-command-1 --help
    $ catplat sub-command-1 --sub-command-1-required-options sub-command-2 --help

Documentation
-------------------
.. click:: catplat.cli.main:main_entry_point
   :prog: catplat
   :nested: full