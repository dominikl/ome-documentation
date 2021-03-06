OMERO.scripts user guide
========================

:doc:`/developers/server-blitz` provides a service to run scripts
on the server. The scripts are then passed on to a grid of processors
called OMERO.grid that executes the script and returns the result to the
server which in turn passes the result onto the caller. All scripts are of the
form:

::

    # import the omero package and the omero.scripts package.
    import omero, omero.scripts as script

    '''
    This method creates the client script object, with name SCRIPTNAME and SCRIPTDESCRIPTION.
    The script then supplies a number of variables that are both inputs and outputs to the
    script. The types allowed are defined in the script package, with the qualifier after the
    variable of in, out or inout depending on whether the variable if for input, output or input
    and output.
    '''
    client = script.client("SCRIPTNAME", "SCRIPTDESCRIPTION",
             script.TYPE("VARIABLENAME").[in()|out()|inout()], …)
    # create a session on the server.
    client.createSession()

    # All variables are stored in a map accessed by getInput and setOutput via the client object.
    VARIABLENAME = client.getInput("VARIABLENAME");
    client.setOutput("VARIABLENAME", value);

This is a guide to getting started with the scripting service, without
going into the 'behind the scenes' details. More technical details can
be found on the :doc:`/developers/scripts/advanced` page. In addition to this
guide, you may find the following pages useful for more information on using
the OMERO Python API: |OmeroClients|, |OmeroPy|.

Sample scripts
--------------

Below are two sample scripts. You can find the core scripts that are
distributed with the OMERO.server under the
`scripts repository <https://github.com/ome/scripts>`_ or download them from
OMERO.insight (from the bottom-left of any run-script dialog), or use the
GitHub repositories forked from `ome/omero-user-scripts <https://github.com/ome/omero-user-scripts/network/members>`_ 
to find scripts written by other users.

Ping script
^^^^^^^^^^^

This script echoes the input parameters as outputs.

::

    import omero, omero.scripts as script
    client = script.client("ping.py", "simple ping script",
             script.Long("a"), script.String("b"))
    client.createSession()

    keys = client.getInputKeys()
    print "Keys found:"
    print keys
    for key in keys:
          client.setOutput(key, client.getInput(key))

Accessing an Image and Channels on the server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This example shows usage of the Python Blitz Gateway to
access an Image, using its ID. We then list the Channel names
and the script returns them as outputs.

::

    import omero, omero.scripts as scripts
    from omero.gateway import BlitzGateway
    from omero.rtypes import wrap

    # Define the script name & description, and a single 'required' parameter
    client = scripts.client("Get_Channels.py", "Get channel names for an image",
        scripts.Long("imageId", optional=False))

    # get the Image Id from the parameters.
    imageId = client.getInput("imageId", unwrap=True)   # unwrap the rtype

    # Use the Python Blitz Gateway for convenience
    conn = BlitzGateway(client_obj=client)

    # get the Image, print its name
    image = conn.getObject("Image", imageId)
    print image.getName()

    # Print each channel 'label' (Name or Excitation wavelength)
    for i, ch in enumerate(image.getChannels()):
        print ch.getLabel()
        # Return as output. Key is string, value is rtype
        client.setOutput("Channel%s" % i, wrap(str(ch.getLabel())))

    # Cleanup
    client.closeSession()

Dynamic arguments
^^^^^^^^^^^^^^^^^

In general, script parameters are processed on server startup
and cached for later use. If you have a script which should
recalculate its arguments on each invocation, use the
`NSDYNAMIC` namespace:


.. literalinclude:: Example_Dynamic_Script.py
   :start-after: # Included in

:download:`Example_Dynamic_Script.py <Example_Dynamic_Script.py>`


Script writing as 'Admin'
-------------------------

The basic steps in a script-writing workflow are:

-  Write a script using your favorite text editor, save locally
-  Use command line (or OMERO.insight) to upload script to server
-  Use command line (or OMERO.insight or web clients) to run script on the
   server (results will be displayed)
-  Edit script and replace copy on server and run again, etc.

Working with scripts is far more straightforward if you have admin access to
your OMERO.server installation - this is the preferred workflow. It is
possible to work with scripts as a regular user (see :doc:`advanced`) but the
software you would be required to install means it is easier to install a
server on your local machine so you have admin rights.

It is assumed that scripts written by a server admin are "trusted" to
run on the server without causing any disruption or security risks. Once
uploaded these scripts are available to all regular users of the server
alongside the official scripts included in each OMERO release.

Download / Edit script
^^^^^^^^^^^^^^^^^^^^^^

The easiest way to get started is to take an existing script and edit it
for your needs. An example created for the purpose of this tutorial can
be found at
:source:`Edit_Descriptions.py <examples/ScriptingService/Edit_Descriptions.py>`.
You should organize your scripts on your local machine in a way that
makes sense to users, since your local file paths will be mimicked on
the server and used to organize script menus in OMERO.insight (see screen-shot
above).

::

    # Save the script to a suitable location. The tutorial will use this location:
    # Desktop/scripts/demo_tutorial/Edit_Descriptions.py

The action of this script (editing Image descriptions) is trivial but it
demonstrates a number of features that you may find useful, including
conventions for inputs and outputs to improve interaction with
OMERO.insight (as discussed on the :doc:`style-guide`).

The script is well documented and should get you started. A few points
to note:

Since the OMERO 4.3 release, if you are using the 'Blitz Gateway',
you can get a connection wrapper like this:

::

    from omero.gateway import BlitzGateway

    conn = BlitzGateway(client_obj=client)
    # now you can do E.g. conn.getObject("Image", imageId) etc.

Alternatively, if you are working directly with the OMERO services, you
can get a service factory like this:

::

    session = client.getSession()
    # now you can do E.g. session.getQueryService() etc.

More example scripts
^^^^^^^^^^^^^^^^^^^^

Several official scripts are included in the release of OMERO and can be
found under the lib/scripts/omero/ directory of the server package. Any
script can also be download from the OMERO.insight client (bottom-left
of the run-script dialog).

.. Warning::
    If you wish to edit the official scripts that are part of the
    OMERO release, you should be prepared to apply the same changes to
    future releases of these scripts from OMERO. If you think that your
    changes should be included as part of future released scripts, please
    let us know.

Upload script
^^^^^^^^^^^^^

You can use the command line, OMERO.insight or the server file system to
upload scripts. The ``script`` command line tool is discussed in more
detail below.

You may find it useful to add the OMERO.server/bin/ folder to your PATH
so you can call ``bin/omero`` commands when working in the scripts folder.
E.g:

.. parsed-literal::

    export PATH=$PATH:/Users/will/Desktop/OMERO.server-|release|/bin/

Upload the script we saved earlier, specifying it as 'official' (trusted
to run on the server processor). You will need to log in the first time
you use the :program:`omero script` command. The new script ID will be printed
out:

::

    $ cd Desktop/scripts/
    $ omero script upload demo_tutorial/Edit_Descriptions.py --official
    Previously logged in to localhost:4064 as root
    Server: [localhost]          # hit 'enter' to accept default login details
    Username: [root]
    Password:
    Created session 09fcf689-cc85-409d-91ac-f9865dbfd650 (root@localhost:4064). Idle timeout: 10.0 min. Current group: system
    Uploaded official script as original file #301

You can add, remove and edit scripts directly in the
OMERO\_HOME/lib/scripts/omero/ folder. Any changes made here will be
detected by OMERO. Official scripts are uniquely identified on the OMERO
server by their 'path' and 'name'.

Any folders in the script path are created on the server under
/lib/scripts/ E.g. the above example will be stored at
/lib/scripts/examples/Edit\_Descriptions.py

The ID of the script is printed after upload and can also be determined
by listing scripts (see below).

Run script
^^^^^^^^^^

You can run the script from OMERO.insight by browsing the scripts (see
screen-shot above). A UI will be generated from the chosen script and
the currently selected images or datasets will be populated if the
script supports this (see :doc:`style-guide`).

Or launch the script from the command line, specifying the script ID.
You will be asked to provide input for any non-optional parameters that
do not have default values specified. Any stdout and stderr will be
displayed as well as any outputs that the script has returned.

::

    wjm:examples will$ omero script launch 301  # script ID
    Using session 1202acc0-4424-4fa2-84fe-7c9e069d3563 (root@localhost:4064). Idle timeout: 10.0 min. Current group: system
    Enter value for "IDs": 1201
    Job 1464 ready
    Waiting....
    Callback received: FINISHED

        *** start stdout ***
        * {'IDs': [1201L], 'Data_Type': 'Dataset'}
        * Processing Images from Dataset: LSM - .mdb
        * Editing images with this description:
        * No description specified
        *
        *    Editing image ID: 15651 Name: sample files.mdb [XY-ch-02]
        *    Editing image ID: 15652 Name: sample files.mdb [XY-ch-03]
        *    Editing image ID: 15653 Name: sample files.mdb [XY-ch]
        *    Editing image ID: 15654 Name: sample files.mdb [XYT]
        *    Editing image ID: 15655 Name: sample files.mdb [XYZ-ch-20x]
        *    Editing image ID: 15656 Name: sample files.mdb [XYZ-ch-zoom]
        *    Editing image ID: 15658 Name: sample files.mdb [XYZ-ch0]
        *    Editing image ID: 15657 Name: sample files.mdb [XYZ-ch]
        *
        *** end stdout ***


        *** out parameters ***
        * Message=8 Images edited
        ***  done ***

Parameter values can also be specified in the command.

::

    # simply specify the required parameters that don't have defaults
    $ omero script launch 301 IDs=1201

    # can also specify additional parameters
    $ omero script launch 301 Data_Type='Image' IDs=15652,15653 New_Description="Adding description from script to Image"

Edit and replace
^^^^^^^^^^^^^^^^

Edit the script and upload it to replace the previous copy, specifying
the ID of the file to replace.

::

    $ omero script replace 301 examples/Edit_Descriptions.py

Finally, you can upload and run your scripts from OMERO.insight.

Other script commands
^^^^^^^^^^^^^^^^^^^^^

Start by printing help for the script command:

::

    $ omero script -h
    usage: /Users/will/Documents/workspace/Omero/dist/bin/omero script
           [-h] <subcommand> ...

    Support for launching, uploading and otherwise managing OMERO.scripts

    Optional Arguments:
      In addition to any higher level options

      -h, --help          show this help message and exit

    Subcommands:
      Use /Users/will/Documents/workspace/Omero/dist/bin/omero script <subcommand> -h for more information.

      <subcommand>
        demo              Runs a short demo of the scripting system
        list              List files for user or group
        cat               Prints a script to standard out
        edit              Opens a script in $EDITOR and saves it back to the server
        params            Print the parameters for a given script
        launch            Launch a script with parameters
        disable           Makes script non-executable by setting the mimetype
        enable            Makes a script executable (sets mimetype to text/x-python)
        jobs              List current jobs for user or group
        serve             Start a usermode processor for scripts
        upload            Upload a script
        replace           Replace an existing script with a new value
        run               Run a script with the OMERO libraries loaded and current login

To list scripts on the server:

::

    $ omero script list
    Using session 09fcf689-cc85-409d-91ac-f9865dbfd650 (root@localhost:4064). Idle timeout: 10.0 min. Current group: system
     id  | Official scripts                            
    -----+---------------------------------------------         
     202 | /omero/export_scripts/Batch_Image_Export.py 
     203 | /omero/export_scripts/Make_Movie.py         
     204 | /omero/figure_scripts/Movie_Figure.py       
     205 | /omero/figure_scripts/Movie_ROI_Figure.py   
     206 | /omero/figure_scripts/ROI_Split_Figure.py   
     207 | /omero/figure_scripts/Split_View_Figure.py  
     208 | /omero/figure_scripts/Thumbnail_Figure.py   
     8   | /omero/import_scripts/Populate_ROI.py         
     209 | /omero/util_scripts/Channel_Offsets.py      
     210 | /omero/util_scripts/Combine_Images.py       
     211 | /omero/util_scripts/Images_From_ROIs.py     
    (14 rows)

If you want to know the parameters for a particular script you can use
the params command. This prints out the details of the script, including
the inputs.

::

    $ wjm:examples will$ omero script params 301
    Using session 1202acc0-4424-4fa2-84fe-7c9e069d3563 (root@localhost:4064). Idle timeout: 10.0 min. Current group: system

    id:  301
    name:  Edit_Descriptions.py
    version:
    authors:
    institutions:
    description:  Edits the descriptions of multiple Images,
    either specified via Image IDs or by the Dataset IDs.
    namespaces:
    stdout:  text/plain
    stderr:  text/plain
    inputs:
      New_Description - The new description to set for each Image in the Dataset
        Optional: True
        Type: ::omero::RString
        Min:
        Max:
        Values:
      IDs - List of Dataset IDs or Image IDs
        Optional: False
        Type: ::omero::RList
        Subtype: ::omero::RLong
        Min:
        Max:
        Values:
      Data_Type - The data you want to work with.
        Optional: False
        Type: ::omero::RString
        Min:
        Max:
        Values: Dataset, Image
    outputs:

Debugging scripts
-----------------

The stderr and stdout from running a script should always be returned to
you, either when running scripts from the command line, via
OMERO.insight or using the scripts API. This should allow you to debug
any problems you have.

You can also look at the output from the script in the OriginalFile
directory, commonly stored in /OMERO/File/. The script file when
executed is uploaded as a new OriginalFile, and the standard error,
standard out are saved as the next two OriginalFiles after that. These
files can be opened in a text editor to examine contents.

