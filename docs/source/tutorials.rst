.. tutorials:

Tutorials
====================

Tutorial 1: Setting Up CatPlat
-------------------------------

Welcome to the first CatPlat tutorial!
We're thrilled to have you here.
Please go through the code examples, and don't hesitate to contact the CatPlat team if you have questions or feedback.

In this tutorial, we'll walk you through the process of setting up CatPlat.

To keep your project organized, let's create a dedicated folder for the tutorials.

.. code-block:: console

    $ mkdir ~/catplat_tutorial
    $ cd ~/catplat_tutorial

1.1 Activating Apptainer CatPlat Container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We distribute CatPlat as a container to manage the complex dependencies.
`Apptainer <https://apptainer.org>`_, formerly known as Singularity, is used for the containerization as this has been the most popular choice for the HPC community.
Apptainer is an available module on the ACRC machines, and NSCC Aspire2A has the Singularity/4.0.0 module available which is recent enough for all intents and purposes.

Before starting the tutorials, you might want to copy the vasp pseudopotentials to the current directory.

.. code-block:: console

    $ cp -r ~chenwjb/shared/potcars ./

You might also want to request an interactive session on a compute node of the server you're currently on. 
You can do so with the following commands.

.. code-block:: console
    :caption: For ACRC

    $ qsub -X -I -l software="VASP" -l select=1:ncpus=16:mpiprocs=16:mem=100G -l walltime=8:00:00
    $ cd ~/catplat_tutorial

.. code-block:: console
    :caption: For Aspire2A

    $ qsub -X -P your_project_name -I -l select=1:ncpus=16:mpiprocs=16:mem=100G -l walltime=8:00:00
    $ cd ~/catplat_tutorial

These commands should give you an interactive shell running on the compute node. We can now load Apptainer/Singularity: 

.. code-block:: console
    :caption: For ACRC

    $ module load apptainer

.. code-block:: console
    :caption: For Aspire2A

    $ module load singularity/4.0.0

To run the container with an interactive shell, execute the following command.

.. code-block:: console
    :caption: running the container interactively for ACRC

    $ apptainer shell ~chenwjb/shared/catplat.sif

.. code-block:: console
    :caption: running the container interactively for Aspire2A

    $ singularity shell ~chenwjb/shared/catplat.sif

The command prompt should change to ``<CatPlat>:username@machinename:~/catplat_tutorial``, indicating that we are now inside the container.
This is a full fledged BASH shell running on an Ubuntu OS that has your $HOME (from the host machine) conveniently mounted as its $HOME.
For details on the host locations being auto-mounted, and how to mount other locations from the host, please click `here <https://apptainer.org/docs/user/main/bind_paths_and_mounts.html>`_.

To enable autocompletion of catplat commands and some convenient aliases, source the startscript.

.. code-block:: console
    :caption: sourcing startscript

    $ source /etc/start

You're now all set!
Let's ensure that everything is working correctly by running Pyatoms and the CatPlat command-line interface:

.. code-block:: console

    $ pyatoms --help
    $ catplat --help

Let's also make use of the container shell and make a separate directory for tutorial 1 and drop down to it.

.. code-block:: console

    $ mkdir tutorial_1
    $ cd tutorial_1

.. note::
    The interactive shell is good for performing analyses, and we will be using this for the tutorials.
    However, CatPlat calculations that require DFT computations must be submitted through a job script.
    In that case, you will need to use the `Apptainer/Singularity exec command <https://apptainer.org/docs/user/main/cli/apptainer_exec.html>`_.

1.2 Setting Up a Project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pyatoms and CatPlat use yaml files in ``~/.pyatoms/pyatoms.yaml`` and ``~/.catplat/catplat.yaml`` to store user configurations.
Although they may be edited directly, we recommend using the ``pyatoms config`` and ``catplat config`` commands to set up your configurations.

As CatPlat is built on top of Pyatoms, we have to set up Pyatoms first.
To set up Pyatoms, we'll demonstrate how to configure a sample project named "myproject".

The Pyatoms project uses a yaml file, stored in ``~/.pyatoms/vasp/``, to specify details such as INCAR, KPOINTS, and POTCAR parameters for our calculations.
Let's create the directory for Pyatoms to store the project information:

.. code-block:: console

    $ mkdir -p ~/.pyatoms/vasp

Next, set the pseudopotential directory, which Pyatoms will use to automatically create POTCARs for VASP calculations:

.. code-block:: console

    $ pyatoms config vasp-pp --src ../potcars --dst ~/pymatgen_pp

Now, let's set up a Pyatoms project:
You may download a template of a ``myproject.yaml`` file :download:`here <_static/myproject.yaml>`

.. note:: 
    
    The yaml file should be named as {project_name}.yaml, where {project_name} is the name of your project.
    Running the ``pyatoms config vasp-project`` command with a project that already exists will overwrite the existing file.

.. code-block:: console

    $ wget https://raw.githubusercontent.com/chryston/catplat_tutorial/main/examples/myproject.yaml --no-check-certificate
    $ pyatoms config vasp-project --yaml-file myproject.yaml

Finally, we have to set up a CatPlat project of the same name to define the calculation and database paths.

.. code-block:: console

    $ mkdir -p ~/catplat_tutorial/myproject/calculation

    FOR LOCAL DATABASE
    $ catplat config project --name myproject --calculation-path ~/catplat_tutorial/myproject/calculation --database-path ~/catplat_tutorial/myproject/myproject.db
    
    FOR SQL DATABASE
    $ catplat config project --name myproject --calculation-path ~/catplat_tutorial/myproject/calculation --database-path mysql://{sqluser}:{sqlpassword}@172.20.116.103:3306/{myproject}


(Optional: only required for running VASP) Lastly, we need to specify the location of your VASP executables:

.. code-block:: console

    $ pyatoms config vasp --gam ${PATH/TO/VASP_GAM_EXECUTABLE} --std ${PATH/TO/VASP_STD_EXECUTABLE}


1.3 Additional Information
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1.3.1 Testing the SQL Tunnel (Only Needed if SQL Database is Used):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To ensure that the SQL database is working correctly, run the following command:

.. code-block:: console

    $ catplat test tunnel

1.3.2 Viewing Structures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To visualize structures, ensure you have an X11 software installed.
You can use `MobaXterm <https://mobaxterm.mobatek.net/>`_ or `PuTTY <https://www.putty.org/>`_ with `Xming <https://sourceforge.net/projects/xming/>`_ to follow along with the tutorials.

You may also run ase's visualization GUI directly to view structures:

.. code-block:: console

    $ wget https://raw.githubusercontent.com/chryston/catplat_tutorial/main/examples/sample_POSCAR --no-check-certificate
    $ ase gui sample_POSCAR

Tutorial 2: Monodentate Adsorption on Cu(100) Surface
-------------------------------------------------------------------

Now that you have CatPlat set up and ready to go, let's take a look at how you can use CatPlat to perform heterogeneous catalysis workflows.

.. admonition:: Scenario

    Computing adsorption energies is one of the fundamental tasks of heterogeneous catalysis.
    Yet, this demands an lengthy and extensive sequence of procedures. 
    
    Presented below is an overview of the traditional computational workflows required to compute adsorption energies:

    .. image:: _static/traditional_workflows.png
        :width: 400
        :align: center

    In this tutorial, we will be demonstrating the use of CatPlat to compute all **CO adsorption on all unique sites of a Cu(100) surface**.

To keep your project organized, let's create a dedicated folder for this tutorial.
Ensure that all commands for this tutorial are executed within this folder.

.. code-block:: console

    $ mkdir -p ~/catplat_tutorial/tutorial_2
    $ cd ~/catplat_tutorial/tutorial_2

2.1 CatPlat's Workflows
^^^^^^^^^^^^^^^^^^^^^^^^

We can use CatPlat's workflows to automate the creation and optimization of the structures.

2.1.1 CatPlat Calculate and Retrieve (Project Setup Required)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``catplat calculate`` is the main command that automatically creates, optimizes, and stores the results in a database.
We will be using the project created in `Section 1.2 <1.2 Setting Up a Project>`_ for this tutorial.
Do ensure that you have set up the project before proceeding.
If you have not, you may skip to the `next part <2.1.2 CatPlat Dryrun>`_.

.. warning::

    ``catplat calculate`` command performs the actual DFT calculations.
    Therefore, you should **not** run the next code block on the head node.
    These commands should be added into a job script and be submitted via the scheduler.

.. code-block:: console
    :caption: WARNING: DO NOT RUN THIS COMMAND ON THE HEAD NODE!!!

    $ catplat calculate -p myproject --chemsys Cu --miller-index 1 0 0 --unitcell-size 3 3 --adsorbate-atoms CO

Instead, we can add in the ``--test`` flag to see the output without running the actual calculations or writing any data.

.. code-block:: console
    :caption: YOU MAY RUN THIS COMMAND ON THE HEAD NODE.

    $ catplat calculate -p myproject --chemsys Cu --miller-index 1 0 0 --unitcell-size 3 3 --adsorbate-atoms CO --test

The output should look like this, the energies here are written as dummy values (123 eV) which indicates that the calculations are not run.

.. image:: _static/tutorial_2/tut2-5.png
   :height: 1400
   :align: center

The ``catplat retrieve`` command retrieves existing results from the project database.

.. code-block:: console 
    :caption: You do not need to run this command as there are no stored calculations.

    $ catplat retrieve -p myproject --chemsys Cu --miller-index 1 0 0 --unitcell-size 3 3 --adsorbate-atoms CO

    Add the --write-csv to write the results to a csv file.
    $ catplat retrieve -p myproject --chemsys Cu --miller-index 1 0 0 --unitcell-size 3 3 --adsorbate-atoms CO --write-csv

2.1.2 CatPlat Dryrun
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``catplat dryrun`` is analogous to ``catplat calculate`` but employs a dummy project, allowing us to trial and visualize the expected outputs of CatPlat workflows.
The options specified can subsequently be copied over to ``catplat calculate`` for the actual calculations.

.. note:: 
    Although ``catplat dryrun`` is similar to ``catplat calculate --test``, we strongly recommend users to use ``catplat dryrun`` for any testing.
    ``catplat dryrun`` does not take in any project information and therefore wouldn't risk writing dummy values to your project's calculation folder/database.

    As such, we will be using ``catplat dryrun`` instead of ``catplat calculate --test`` for all the tutorials.

Let's try the ``catplat dryrun`` command:

.. code-block:: console

    $ catplat --verbose dryrun --chemsys Cu --miller-index 1 0 0 --unitcell-size 3 3 --adsorbate-atoms CO --no-cleanup | tee dryrun.out

The ``--no-cleanup`` flag retains the database and calculation files in the temporary directory created for the dry run.
We can check the "INPUTS" section of the ``dryrun.out`` for the path in which the structures and database files are written to.
You can quickly see how the calculation files are structured and the database is organized.

.. code-block:: console
     
    Let's view the output of CatPlat's dryrun command.
    $ cat dryrun.out

    Viewing calculation folders structure
    $ grep calculation_path dryrun.out | awk '{print $2}' | xargs -I % find % -type d

    Viewing database organization
    $ grep database_path dryrun.out | awk '{print $2}' | xargs -I % ase db -c id,workflow_type,formula,miller_index,unitcell_size,adsorbate_atoms,connectivity,energy,relaxed %

    Viewing database using web interface (Only works if you have firewall access to the server)
    $ grep database_path dryrun.out | awk '{print $2}' | xargs -I % ase db % -w

Web interface of the database:

.. image:: _static/tutorial_2/tut2-6.png
   :height: 350
   :align: center

2.2 Creating/Analyzing Structures with CatPlat's CLI (Optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CatPlat provides convenient CLI commands in case we want to perform some parts of the workflow manually. 
Let's try to recreate the workflow above using these CLI commands. This will also help us understand how CatPlat works.

``catplat build`` is dedicated to creating and writing structures, whereas ``catplat analyze`` is designed for in-depth structural analysis, offering valuable insights.

For this tutorial, we will only be generating and analyzing the structures without running any optimization calculations.

.. tip::
    The cli provides numerous options that can be tailored to suit a broad range of use cases.
    The sample code detailed in this tutorial is intended to serve as a foundation to get you started.
    To explore the full spectrum of options available for each subcommand, simply use the ``--help`` option.

2.2.1 Creating the CO Adsorbate Structure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First, let's see how we can obtain the structure of our adsorbate.
For CO, CatPlat retrieves the adsorbate structure from the `ase.collections.g2 database <https://wiki.fysik.dtu.dk/ase/ase/collections.html#ase.collections.g2>`_.
We'll also explore how to retrieve structures from `PubChem's database <https://pubchem.ncbi.nlm.nih.gov>`_ in later tutorials.

.. warning:: 
    Executing the command ``catplat build`` will automatically write structures and overwrite any existing structures with the same name.
    You may specify ``--no-write`` if you wish to inspect the generated structures without saving them.

.. code-block:: console
    :caption: Creating adsorbate structures.

    $ catplat build adsorbate --name CO --box-size 15

You should see a new folder named ``catplat_adsorbate`` containing the adsorbate structure.
The adsorbate structure should be centered in a vacuum box of 15 Angstroms.

.. code-block:: console
    :caption: Analyzing adsorbate structures.

    $ catplat analyze adsorbate --name catplat_adsorbate/CO/POSCAR

.. image:: _static/tutorial_2/tut2-1.png
   :height: 250
   :align: center

CatPlat also solves for the Lewis structure of the adsorbate, offering valuable insights into its properties.
This is useful for automatically determine the atom by which the adsorbate should bind to the surface, which defined as the ``bond`` attribute.

.. admonition:: CatPlat Attribute: Bonds

    * ``Bonds`` correspond to the atom indices of the adsorbate that will subsequently bind on the adsorption site. 
    * You may view the atom indices in the ase gui by clicking ``View > Show Labels > Atom Index``. 
    * In this example:
        *  ``bonds 0`` binds CO to the surface via the C atom, 
        *  ``bonds 1`` binds CO to the surface via the O atom
        *  ``bonds 0,1`` binds CO to the surface bidentate via C and O.

.. admonition:: Pyatoms Job Submission

    After creating the adsorbate structure, you may use Pyatoms to do a gas phase geometry optimization of the adsorbate.
    However, we will not be relaxing the structures in this tutorial.

    .. code-block:: console
        :caption: FOR REFERENCE ONLY. DO NOT RUN THIS COMMAND ON THE HEAD NODE.

        $ pyatoms sub -f catplat_adsorbate/CO vasp --project myproject --gas geomopt


2.2.2 Getting Cu Bulk Structure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, we obtain our bulk structure by querying the `Materials Project database <https://next-gen.materialsproject.org/>`_.
Let's attempt to get the traditional Cu fcc bulk structure:

.. code-block:: console
    :caption: Creating bulk structures.

    $  catplat build bulk --chemsys Cu --spacegroup 225

You should see a new folder named ``catplat_bulk`` containing the bulk structure.
We can analyze the symmetries of the bulk structure using the following command:

.. code-block:: console
    :caption: Analyzing bulk structures.

    $ catplat analyze bulk --name catplat_bulk/Cu_225_0/POSCAR

.. image:: _static/tutorial_2/tut2-2.png
   :height: 200
   :align: center

.. admonition:: Pyatoms Job Submission

    Similarly, we can use Pyatoms to do a cell optimization of the bulk structure.

    .. code-block:: console
        :caption: FOR REFERENCE ONLY. DO NOT RUN THIS COMMAND ON THE HEAD NODE.

        $ pyatoms sub -f catplat_bulk/Cu_225_0 vasp --project myproject cellopt

2.2.3 Building the Cu(100) Slab
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Then, we cleave the bulk using our in-house algorithms to obtain the slab structure.
Let's build a 3x3 Cu(100) slab with 4 layers, where the bottom 2 layers are fixed.

.. code-block:: console
    :caption: Creating slab structures.

    $  catplat build slab --name catplat_bulk/Cu_225_0/POSCAR --miller 1 0 0 --num-layers 4 --num-fixed-layers 2 --unitcell-size 3 3

You should see a new folder named ``catplat_slab`` containing the slab structure.
We can analyze the slab structure and adsorption sites using the following commands:

.. code-block:: console
    :caption: Analyzing slab structures.

    $ catplat analyze slab --name catplat_slab/Cu36_0/POSCAR
    $ catplat analyze sites --name catplat_slab/Cu36_0/POSCAR

.. image:: _static/tutorial_2/tut2-3.png
   :height: 350
   :align: center
  
There are 3 unique sites on Cu(100).
Analyzing the sites gives us valuable information on the site properties, such as the connectivity and average coordination number.
We'll learn to use this information to achieve fine control over the adsorption site(s) of interest in future tutorials.

.. admonition:: Pyatoms Job Submission

    We can use Pyatoms to do a geometry optimization of the slab structure.

    .. code-block:: console
        :caption: FOR REFERENCE ONLY. DO NOT RUN THIS COMMAND ON THE HEAD NODE.

        $ pyatoms sub -f catplat_slab/Cu36_0 vasp --project myproject geomopt

2.2.4 Adsorption of CO on the Slab
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Finally, let's adsorb CO on all 3 unique sites of Cu(100).

.. code-block:: console
    :caption: Creating complex structures.

    $ catplat build complex --slab catplat_slab/Cu36_0/POSCAR --adsorbate catplat_adsorbate/CO/POSCAR

.. image:: _static/tutorial_2/tut2-4.png
   :height: 350
   :align: center

.. tip::
    The visualization window that opens up contains information on the adsorption schematic.
    You may press ``View > Show forces`` or ``ctrl+f`` to view the adsorption vector of the adsorbate.
    The small red atom, X, is a dummy atom showing the adsorption coordinates. 

The 3 structures in ``catplat_complex`` correspond to the 3 adsorption sites.
Let's analyze the structures. We have to specify the adsorbate and slab chemical symbols to properly differentiate between the slab and adsorbate atoms.

.. code-block:: console
    :caption: Analyzing complex structures
    
    $ catplat analyze complex --name catplat_complex/CCu36O_0/POSCAR --adsorbate CO --slab-elements Cu
    $ catplat analyze complex --name catplat_complex/CCu36O_1/POSCAR --adsorbate CO --slab-elements Cu
    $ catplat analyze complex --name catplat_complex/CCu36O_2/POSCAR --adsorbate CO --slab-elements Cu

.. admonition:: Pyatoms Job Submission

    We can use Pyatoms to do a geometry optimization of the complex structure(s).

    .. code-block:: console
        :caption: FOR REFERENCE ONLY. DO NOT RUN THIS COMMAND ON THE HEAD NODE.

        $ pyatoms sub -N 3 -o catplat_complex/* vasp --project myproject geomopt

Tutorial 3: Bidentate adsorption on Cu(211) Surface
-------------------------------------------------------------------

.. admonition:: Scenario

    Bidentate adsorption -- where an adsorbate binds to the surface via two atoms -- is very common in heterogeneous catalysis,.
    More complex, stepped surfaces such as the fcc(211) are also commonly used to model the interface between the fcc(111) and fcc(100) facets of a nanoparticle.
    Additionally, stepped surfaces are also more reactive due to the lower coordination number of the step-edge atoms.
    In this tutorial, we show how CatPlat can handle a more complex example of **acetate adsorption on a stepped Cu(211) surface**.

To keep your project organized, let's create a dedicated folder for this tutorial.
Ensure that all commands for this tutorial are executed within this folder.

.. code-block:: console

    $ mkdir -p ~/catplat_tutorial/tutorial_3
    $ cd ~/catplat_tutorial/tutorial_3

3.1 CatPlat Workflows
^^^^^^^^^^^^^^^^^^^^^^^

We can simply analyze this using CatPlat's workflows.

Quickly add the acetate structure to your adsorbate folder.
We will explain this command in more detail in the next section.

.. code-block:: console

    $ catplat build adsorbate --name pubchem:175 --box-size 15
    $ mkdir -p ~/.catplat/adsorbate;cp catplat_adsorbate/C2H3O2/POSCAR ~/.catplat/adsorbate/acetate_POSCAR

Let's try the ``catplat dryrun`` command to bind acetate near the step-edge of the Cu(211) surface, note that this may take a few minutes to run:

.. code-block:: console

    $ catplat dryrun --chemsys Cu --spacegroup 225 --miller 2 1 1 --unitcell-size 2 4 --num-layers 12 --num-fixed-layers 6 --adsorbate-atoms acetate --bonds 0,1 --avg-coord-num "<=8,<=8" | tee dryrun.out

We can see that 6 complex structures were produced.
We will breakdown the options specified in the dryrun command in the next section. 

3.2 Creating/Analyzing Structures with CatPlat's CLI (Optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Similar to Tutorial 2, we will now break down the above command into smaller steps to understand how the structures are created.

3.2.1 Creating the Acetate Asorbate Structure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Acetate is not found in the `ase.collections.g2 database <https://wiki.fysik.dtu.dk/ase/ase/collections.html#ase.collections.g2>`_. 
Therefore, we will use the PubChem API to create the adsorbate.
The link to the pubchem molecule that we will be building today can be found `here <https://pubchem.ncbi.nlm.nih.gov/compound/Acetate>`_.

There are multiple ways to specify this molecule.
We can either use its SMILES string, PubChem ID, or simply its name. 

.. code-block:: console
    :caption: Multiple ways to build the adsorbate structure. The double quotes in the first command are required to escape the parenthesis.

    Using SMILES string
    $ catplat build adsorbate --name "pubchem:CC(=O)[O-]" --box-size 15

    Using PubChem ID
    $ catplat build adsorbate --name pubchem:175 --box-size 15

    Using the name of the molecule
    $ catplat build adsorbate --name pubchem:acetate --box-size 15

.. image:: _static/tutorial_3/tut3-1.png
   :height: 300
   :align: center

Acetate typically binds in a bidentate fashion. To achieve this, we need to let CatPlat which two atoms to bind to the surface.
Let's analyze the adsorbate to determine the likely bonded atoms.

.. code-block:: console

    $ catplat analyze adsorbate --name catplat_adsorbate/C2H3O2/POSCAR

The output, ``Automatic bond detection (bidentate)``, suggests that the acetate adsorbate would bind via ``0,1``, corresponding to the two O atoms.
This information would be specified later during the complex building.

3.2.2 Building the Cu(211) Slab
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As mentioned in the previous tutorial, bulk structures can be obtained using the following command.

.. code-block:: console

    $  catplat build bulk --chemsys Cu --spacegroup 225

Slab building is often an iterative process.
We can build and view trial structures without writing them using the ``--no-write`` flag.
Firstly, let's build a trial fcc(211) slab to see what the slab looks like.

.. code-block:: console
    :caption: Trial slab structure

    $  catplat build slab --name catplat_bulk/Cu_225_0/POSCAR --miller 2 1 1 --num-layers 1 --num-fixed-layers 1 --no-write

.. image:: _static/tutorial_3/tut3-2.png
   :height: 150
   :align: center

Due to the stepped nature of the surface, we observe that the minimum basis of repetition in the z-direction consists of 3 atoms.
Therefore, the ``--num-layers`` specified for the 211 slab should be multiples of 3.
We can also see that each ``--num-fixed-layers`` here is defined by atoms with the same z-coordinates.
We will also need to account for the minimum basis when specifying ``--unitcell-size``.

After playing around with the slab attributes, we can proceed to create the slab structure.

.. code-block:: console
    :caption: Creating slab structure.

    $  catplat build slab --name catplat_bulk/Cu_225_0/POSCAR --miller 2 1 1 --unitcell-size 2 4 --num-layers 12 --num-fixed-layers 6

.. image:: _static/tutorial_3/tut3-3.png
   :height: 250
   :align: center

The generated slab should now be large enough to accommodate adsorbates.
Let's proceed to analyze the sites of the fcc(211) surface.

.. code-block:: console
    :caption: Analyzing slab structure.

    $  catplat analyze sites --name catplat_slab/Cu96_0/POSCAR

.. |tut3-4| image:: _static/tutorial_3/tut3-4.png
   :height: 300

.. |tut3-5| image:: _static/tutorial_3/tut3-5.png
   :height: 300

|tut3-5| |tut3-4|

The slab has 14 unique adsorption sites.

.. |tut3-7| image:: _static/tutorial_3/tut3-7.png
   :height: 600

.. tip::
    To get more information regarding the specific sites, you may turn on the atom index on the ase gui (``View > Show Labels > Atom Index``).
    The coordinates, connectivity, and average coordination number of each site corresponds to the ``site_idx``

    |tut3-7|

Performing an exhaustive analysis of all 14 unique sites may be too computationally intensive.
Generally, sites closer to the step edge are more reactive as the surface atoms have lower coordination numbers.
We can use ``--connectivity`` and ``--avg-coord-num`` to filter the sites of interest.

.. admonition:: CatPlat Attribute: Connectivity

    * ``Connectivity`` corresponds to the number of surface atoms coordinated to the adsorbate.
    * Connectivity should be integers.
    * In this example:
        *  ``connectivity 1``: red atom represents the top sites
        *  ``connectivity 2``: white atom represents the bridge sites
        *  ``connectivity 3``: green atom represents the 3-fold hollow sites (fcc and hcp)
        *  ``connectivity 4``: blue atom represents the 4-fold hollow sites.
        *  ``connectivity 1,1``: bidentate on top-top side.
  
.. admonition:: CatPlat Attribute: Average Coordination Number

    * ``Average Coordination Number`` corresponds to the average coordination number of the atoms around the site.
    * Generally: The lower the average coordination number, the more reactive the site.
    * Average coordination numbers are often described using comparators.
    * In this example:
        *  ``avg-coord-num`` for this surface ranges from 7 - 10 
        *  ``avg-coord-num 9``: sites on the fcc(111) terrace.
        *  ``avg-coord-num "<=8"``:  sites closer to the step-edge
        *  ``avg-coord-num 7,7``:  bidentate on the step-edge.

Let's select sites of interest based on these two attributes:

.. code-block:: console

    Top sites only:
    $  catplat analyze sites --name catplat_slab/Cu96_0/POSCAR --connectivity 1

    Top and Bridge sites:
    $  catplat analyze sites --name catplat_slab/Cu96_0/POSCAR --connectivity 1 --connectivity 2

    Sites nearer the step-edge:
    $  catplat analyze sites --name catplat_slab/Cu96_0/POSCAR --avg-coord-num "<=8"

    Step-edge top site:
    $  catplat analyze sites --name catplat_slab/Cu96_0/POSCAR --connectivity 1 --avg-coord-num 7


3.2.3 Building Bidentate Complexes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's proceed to bind acetate bidentate near the step edge of the Cu(211) surface.

.. note::
    Bidentate bonds are specified using a ``,`` to separate the two bonds.
    When specifying bidentate ``bonds``, we need to ensure that the denticities of ``avg_coord_num``, ``connectivity``, and ``sites`` are bidentate.

.. code-block:: console  

    $  catplat build complex -s catplat_slab/Cu96_0/POSCAR -a catplat_adsorbate/C2H3O2/POSCAR --bonds 0,1 --avg-coord-num "<=8,<=8"

.. image:: _static/tutorial_3/tut3-8.gif
   :height: 300
   :align: center

CatPlat automatically attempts to enumerate all reasonable bidentate conformations that fit within the specifications.
Here, 6 structures were generated.

Tutorial 4: Specifying Own Structures
-------------------------------------------

.. admonition:: Scenario

    So far, we have been using bulk structures from the Materials Project database.
    However, there may be cases where you may already have your own bulk/slab/adsorbate structures to analyze.
    In this tutorial, we will show how we can use them with CatPlat.

To keep your project organized, let's create a dedicated folder for this tutorial.
Ensure that all commands for this tutorial are executed within this folder.

.. code-block:: console

    $ mkdir -p ~/catplat_tutorial/tutorial_4
    $ cd ~/catplat_tutorial/tutorial_4

4.1 Preparing User Structures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First, let's create the default directories for CatPlat to read the structures from.

.. code-block:: console

    $ mkdir -p ~/.catplat/adsorbate; mkdir -p ~/.catplat/bulk; mkdir -p ~/.catplat/slab

.. note::
    Any structures readable by ase can be used with CatPlat.
    Some examples are, name.xyz, name.cif, POSCAR_name and name_POSCAR.
    ``name`` is be used to specify the structure in CatPlat, file extensions are not required.

Next, let's download some sample structures, we will try to incorporate these structures into our workflow:

.. code-block:: console

    $ wget https://raw.githubusercontent.com/chryston/catplat_tutorial/main/examples/user_ads.xyz --no-check-certificate
    $ wget https://raw.githubusercontent.com/chryston/catplat_tutorial/main/examples/POSCAR_user_bulk --no-check-certificate
    $ wget https://raw.githubusercontent.com/chryston/catplat_tutorial/main/examples/user_slab_POSCAR --no-check-certificate

.. code-block:: console
    :caption: Visualizing user structures.

    $ ase gui user_ads.xyz POSCAR_user_bulk user_slab_POSCAR

.. image:: _static/tutorial_4/tut4-1.png
    :height: 250
    :align: center

Now, let's copy the structures to their respective folders!

.. code-block:: console

    $ cp user_ads.xyz ~/.catplat/adsorbate; cp POSCAR_user_bulk ~/.catplat/bulk; cp user_slab_POSCAR ~/.catplat/slab

4.2 Using Existing Structures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After all the structures are in the respective directories, we can use them with CatPlat by specifying the ``name`` of the structure.
A quick reminder that the file extensions should not be specified when specifying the ``name`` of the structure.

.. note::
    When  specifying ``--bulk-atoms``, other bulk-related flags such as ``--chemsys`` and ``--spacegroup`` should not be specified.
    Likewise, bulk and slab-related flags should not be specified when specifying ``--slab-atoms``.
    
.. code-block:: console
    :caption: Executing workflow with user structures.

    Using user bulk structure
    $ catplat dryrun --bulk-atoms user_bulk

    Using user slab and user adsorbate structures
    $ catplat dryrun --slab-atoms user_slab --adsorbate-atoms user_ads 

.. tip::
    If you do not want to use the CatPlat directories, you may also specify the path directly.

    .. code-block:: console
        :caption: Specifying user structures by path

        $ catplat dryrun --bulk-atoms ~/catplat_tutorial/tutorial_4/POSCAR_user_bulk
        $ catplat dryrun --slab-atoms ~/catplat_tutorial/tutorial_4/user_slab_POSCAR --adsorbate-atoms ~/catplat_tutorial/tutorial_4/user_ads.xyz

Tutorial 5: Cu-X Alloys
------------------------------------------

.. admonition:: Scenario

    You are conducting research on Cu-X binary alloys for CO2 reduction reactions (CO2RR) and wish to leverage on CatPlat for this purpose.
    This tutorial will walk you through the process of analyzing CO adsorption on a fcc(111) facet Cu-X alloy.

To keep your project organized, let's create a dedicated folder for this tutorial.
Ensure that all commands for this tutorial are executed within this folder.

.. code-block:: console

    $ mkdir -p ~/catplat_tutorial/tutorial_5
    $ cd ~/catplat_tutorial/tutorial_5

5.1 Building Alloy Bulks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's start by finding Cu-X alloy bulks in the Materials Project database.

.. code-block:: console

    $ catplat build bulk --chemsys Cu-* --no-write

This command will get all the Cu-X alloy structures. If the list is too extensive, you can narrow it down to stable Cu-Rh and Cu-Pt structures:

.. code-block:: console

    $ catplat build bulk --chemsys Cu-[Rh,Pt] --e-above-hull "<0.05"

We have now refined the number of alloy bulks to a manageable number. Let's see how we can build the alloy slabs in the next section.

5.2 Building Alloy Slabs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Alloy structures can pose unique challenges when specifying miller indices, as they may not always correspond to the traditional Miller indices.
For the sake of the tutorial, let's use the example of ``Cu3Rh_139_*`` and specify the slab using ``--miller-index 1 1 1``.
Keep in mind that the resulting slab may not resemble a typical fcc(111) surface.

Furthermore, we do not know how many atoms there are in each basis of repetition in the x, y and z direction.
Hence, we use ``--length`` and ``--thickness`` instead of ``--unitcell-size`` and ``--num-layers`` to specify the size of the slab.

.. code-block:: console
    
    $ catplat build slab --name catplat_bulk/Cu3Rh_139_*/POSCAR --miller-index 1 1 1 --thickness 5 --length 10 10 --no-write

.. image:: _static/tutorial_5/tut5-1.png
   :height: 300
   :align: center

To ensure you're working with the correct miller indices, you can analyze the Miller indices of the bulk using ``catplat analyze miller``.
In this scenario, we try all possible combinations of ``±{max-index}`` for (h,k,l) and use the surface coordination to determine if it corresponds to the traditional fcc(111) facet.

.. code-block:: console

    $ catplat analyze miller --name catplat_bulk/Cu3Rh_139_*/POSCAR --facet 111 --max-index 3

Based on the output, we see that 112 corresponds to the traditional fcc(111) facet. Let's verify this by building the slab:

.. code-block:: console
    
    $ catplat build slab --name catplat_bulk/Cu3Rh_139_*/POSCAR --miller-index 1 1 2 --thickness 5 --length 10 10

.. image:: _static/tutorial_5/tut5-2.png
   :height: 200
   :align: center

Feel free to determine fcc(111)-like Miller indices and build the slab for the other two bulks (``CuPt_166_*`` and ``Cu6Rh2_194_*``) if you have the time.

5.3 Adsorbate Adsorption
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The process of adsorbate adsorption is similar to previous tutorials and won't be covered in detail here.

5.3 Using CatPlat Workflows
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We can use CatPlat's workflows to automate the process. Below is an example:

.. warning::
    To prevent the inadvertent creation of incorrect slabs, we recommend to limit the number of bulks per ``catplat calculate`` / ``catplat dryrun`` to one.
    We also highly recommend testing the commands before conducting the actual calculations.

.. code-block:: console

    $ catplat dryrun --chemsys Cu-Rh --max-e-above-hull 0.05 --spacegroup 139 --miller-index 1 1 2 --min-thickness 5 --min-length 10 10 --adsorbate-atoms CO

Tutorial 6: Co-adsorption of Multiple Adsorbates
-----------------------------------------------------

.. admonition:: Scenario

    Up to this juncture, our focus has been on scenarios involving the adsorption of a single molecule on the surface.
    However, certain situations, such as coverage effects, necessitate the adsorption of multiple molecules on the surface.
    Manually achieving this can be incredibly time-consuming, given the numerous potential adsorption configurations.

    Fortunately, CatPlat offers a solution by automating the generation of these structures.
    In this tutorial, we will explore how to efficiently handle the co-adsorption of 3 oxygen atoms on a small Pt(331) surface.

.. note::

    As of version 0.1.0, CatPlat is only capable of generating co-adsorption structures for a single type of adsorbate.
    Future plans for the next version include:

        * Integration of co-adsorption into CatPlat's workflows.
        * Support for co-adsorption of multiple adsorbate species.

To keep your project organized, let's create a dedicated folder for this tutorial.
Ensure that all commands for this tutorial are executed within this folder.

.. code-block:: console

    $ mkdir -p ~/catplat_tutorial/tutorial_6
    $ cd ~/catplat_tutorial/tutorial_6

6.1 Building the Pt(331) Surface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As covered in the previous tutorials, we can build and analyze the Pt(331) slab with these commands:

.. code-block:: console

    $  catplat build bulk --chemsys Pt --spacegroup 225
    $  catplat build slab --name catplat_bulk/Pt_225_0/POSCAR --miller 3 3 1 --unitcell-size 1 1 --num-layers 12 --num-fixed-layers 6
    $  catplat analyze sites --name catplat_slab/Pt24_0/POSCAR
    $  catplat analyze sites --name catplat_slab/Pt24_0/POSCAR --no-unique

.. image:: _static/tutorial_6/tut6-1.png
   :height: 400

From the command, we can see that there are 20 sites in the unit cell and 9 unique sites.

6.2 Generation of Co-adsorption Complexes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CatPlat enumerates all the possible combinations of `n` adsorbates on the sites in cell, where `n` is the number of adsorbates specified by the user.
CatPlat then removes equivalent structures by graph theory and specified constraints.
The resulting structures are then written to the ``catplat_multicomplex`` directory.

Let's try to generate the co-adsorption structures of 3 oxygen atoms on the Pt(331) surface without any constraints.

.. code-block:: console

    $  catplat build multicomplex --adsorbate atom:O --slab catplat_slab/Pt24_0/POSCAR --quantity 3 --dist-tol 0

.. image:: _static/tutorial_6/tut6-2.png
   :height: 400

As seen in the earlier section, the number of sites in the unit cell is 20.
Hence, the number of possible structures here is 1140 (20 choose 3).
However, we are left with 425 structures after filtering equivalent structures.

6.3 Reducing the Number of Complexes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Although we have reduced the number of structures to 425, it is still a lot of structures to perform DFT relaxations on.
Additionally, the number of possible structures can increase significantly based on the following criteria(s):
    
        * Number of adsorbates increase.
        * Low symmetry of the slab (more unique sites).
        * Large slabs (more sites in cell)
  
In this section, we explore some tricks we can use to limit the number of generated structures. 

6.3.1 Distance Constraints
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Generally, adsorbates prefer not to coordinate with the adjacent sites of the same surface atom.
We can use ``--dist-tol`` to remove adsorbates which are too close to each other.
``dist-tol`` (default=0.8) is a ratio corresponding to the average top-top distance of the surface atoms (d\ :sup:`top_top`).
The actual minimum distance in Angstroms can be obtained by `dist-tol * d\ :sup:`top_top`.

.. code-block:: console

    $  catplat build multicomplex --adsorbate atom:O --slab catplat_slab/Pt24_0/POSCAR --quantity 3 --dist-tol 0.8 --no-write

Adding this constraint narrowed down the number of complexes from 425 to 128.

6.3.2 Connectivity Constraints
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If we have prior information on which site is more stable for the adsorbate, we can filter out the sites that are not so stable.
The adsorbate, for example, could only be stable on top and bridge sites.

.. code-block:: console

    $  catplat build multicomplex --adsorbate atom:O --slab catplat_slab/Pt24_0/POSCAR --quantity 3 --dist-tol 0 --connectivity 1 --connectivity 2 --no-write

Adding this constraint narrowed down the number of complexes from 425 to 35.

6.3.3 Average Coordination Number Constraints
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When analyzing a step surface, the adsorbates are generally more stable on the step edge due to the lower coordination number of the surface atoms.
We can only include the sites near the step edge to save on computing resources.

.. code-block:: console

    $  catplat build multicomplex --adsorbate atom:O --slab catplat_slab/Pt24_0/POSCAR --quantity 3 --dist-tol 0 --avg-coord-num "<8" --no-write

Adding this constraint narrowed down the number of complexes from 425 to 10.

6.3.4 Site Constraints
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can iteratively determine the coverage effects of 3 oxygen atoms on this slab through site constraints.
Before we proceed, let's look at how to specify the exact site for an adsorbate to bind on.

.. code-block:: console

    $  catplat analyze sites --name catplat_slab/Pt24_0/POSCAR --no-unique

If we enable atom index, we can see the number corresponding to each site.
We can fix one or more atoms at the sites using the ``--sites`` option.

.. code-block:: console

    Add 1 adsorbate (9 structures, corresponding to 9 unique sites)
    $  catplat build multicomplex --adsorbate atom:O --slab catplat_slab/Pt24_0/POSCAR --quantity 1 --dist-tol 0 --no-write
    
    Let's say that site 236 is the most stable site, create coadsorption complex with 2 adsorbates (17 structures)
    $  catplat build multicomplex --adsorbate atom:O --slab catplat_slab/Pt24_0/POSCAR --quantity 2 --sites 236 --dist-tol 0 --no-write

    Let's say that site 236 and 30 is the most stable site, create coadsorption complex with 3 adsorbates (16 structures)
    $  catplat build multicomplex --adsorbate atom:O --slab catplat_slab/Pt24_0/POSCAR --quantity 3 --sites 30,236 --dist-tol 0 --no-write

This way, we can narrow down the number of complexes to 42 (9+17+16).

Tutorial 7: Machine Learning Preoptimization
-------------------------------------------------------

.. admonition:: Scenario

    Machine Learning Potentials (MLP) offer a rapid means of obtaining estimates for the relaxed structure and energy.
    CatPlat can employ existing MLP models to preoptimize the generated structures before performing DFT calculations.

    .. image:: _static/tutorial_7/tut7-1.png
        :width: 550
    
    
    Here, we see that ml preoptiomzation of HCOO binding on the Cu(100) surface significantly reduces the number of DFT calls compared to a pure DFT approach.

    .. image:: _static/tutorial_7/tut7-2.png
        :width: 300

    In this tutorial, we will try to apply the GemNet-OC and MACE pretrained models to the workflow in tutorial 2.

To keep your project organized, let's create a dedicated folder for this tutorial.
Ensure that all commands for this tutorial are executed within this folder.

.. code-block:: console

    $ mkdir -p ~/catplat_tutorial/tutorial_7
    $ cd ~/catplat_tutorial/tutorial_7

7.1 Downloading the GemNet-OC Model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pretrained models are stored in ``~/.catplat/mlp``.
You may include your own models by placing them in this directory.

The GemNet-OC model is a Graph Neural Networks (GNN) based model trained on a diverse array of structures sourced from the Open Catalyst 2020 dataset.
You may download the GemNet-OC model using the following command:

.. code-block:: console

    $  catplat download resources

7.2 Viewing Available Models and Hashes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You may check all your available models with the following command:

.. code-block:: console

    $  catplat mlp view

.. image:: _static/tutorial_7/tut7-3.png
   :width: 800
   :align: center

As of the current version, you should see 2 available models.
The first model is the GemNet-OC model, which we have just downloaded.
The second model is a specialized model for Cu-C-H-O systems, developed through training with MACE.

7.3 Using MLP Models
^^^^^^^^^^^^^^^^^^^^^^^^

Using the MLP models is as simple as adding the ``--ml-preopt`` flag to the ``catplat calculate`` or ``catplat dryrun`` command.
You may select which model you want via the ``--ml-potential-hash`` option.
If no hash is specified, CatPlat will automatically select the a suitable model.

.. tip::
    Activate the ``--verbose`` flag to see the ML preoptimization in action.

.. code-block:: console

    Using GemNet-OC model
    $ catplat --verbose dryrun --chemsys Cu --miller-index 1 0 0 --unitcell-size 3 3 --adsorbate-atoms CO --ml-preopt --ml-potential-hash 2830e3c9a0d5910c59f9a3136daeb607

    Using MACE model
    $ catplat --verbose dryrun --chemsys Cu --miller-index 1 0 0 --unitcell-size 3 3 --adsorbate-atoms CO --ml-preopt --ml-potential-hash 922c3f9c9a2e1c7e567ac09db8797b6c

.. image:: _static/tutorial_7/tut7-4.png
   :width: 800
   :align: center
