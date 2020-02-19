![](https://img.shields.io/badge/STATUS-NOT%20CURRENTLY%20MAINTAINED-red.svg?longCache=true&style=flat)

# Important Notice
We have decided to stop the maintenance of this public GitHub repository.


# HANA XS Advanced Python Buildpack

### IMPORTANT NOTICE!!!
As of the release of The HANA Platform version 2 Service Pack 03 (April, 2018) an official python buildpack is provided as part of the standard set of supported language runtimes.  This python buildpack can be used with Service Pack 02 and earlier as needed, but the recommendation will be to upgrade to Service Pack 03 in order to utilize the full SAP support network.

Information on the SP03 Python Buildpack can be found here: [The SAP HANA XS Advanced Python Run Time](https://help.sap.com/viewer/4505d0bdaf4948449b7f7379d24d0f0d/2.0.03/en-US/8d786ec8ab964145a7453c1f53f452db.html)

### Python buildpack info and disclaimers
This Buildpack adds support for modules based on Python code to a HANA XS Advanced system.  If you want to deploy your Python to a Cloud Foundry based multi-tenant system such as SAP Cloud Foundry Cloud Platform or Pivotal Cloud Foundry, this buildpack WILL NOT WORK!  For those Cloud Foundry based deployment environments, use the community Python Buildpack documented at [CloudFoundry.org Python Buildpack](https://docs.cloudfoundry.org/buildpacks/python/index.html).

This Python Buildpack is flexible in that it can use various versions of Python as needed.  Not all versions of Python were tested so your results may vary.

Also note that this buildpack is not feature complete and may not be suitable for your purpose.  It is intended as a simple example of buildpack construction and can not be guaranteed for any particular fitness of purpose.  Please use it at your own risk and discretion.

### Organization of this repository
While this repository is named hana-xsa-python-buildpack, you would expect the files of the buildpack to be at the top of the folder structure and an example of how to use it to be in a sub-folder.  The opposite is true here.  The reason for this is to allow the repo to be git cloned into the HANA WebIDE directly and the non-python built using the standard functions the WebIDE provides.  Once the non-python modules are built and running, you will need to run a script to build the python module and connect it to the other modules.  The reason for this is that the WebIDE currently cannot build a custom python module so we must build it manually.

### Notes on creating the python buildpack
Currently the python buildpack is generated by running a script found in the tools folder of this repo.

The python buildpack will download the version of python in the VERSION file using the wget command so make sure it's installed on your system.  It will then proceed to compile the python core code.  Make sure you have the usual build tools installed on your server.  For SuSe based systems run this as root.

```
zypper install wget
zypper install git-core
zypper install --type pattern devel_basis
zypper install tk-devel
zypper install tcl-devel
zypper install python3
pip install -U pip setuptools
zypper install libffi-devel
zypper install openssl-devel
zypper install readline-devel
zypper install sqlite3-devel
zypper install ncurses-devel
zypper install xz-devel
```

It is assumed that you've cloned this repo into a folder that resides on the HANA server.

First make sure the scripts in the tools folder have execute permissions.

```
cd tools
chmod 755 *
```

Now run the create script giving it the name of a folder that will be created.
```
./create_my_python_buildpack.sh my_python_buildpack
```

The last line of the create script instructs you to install the buildpack into your XS Advanced system.
Make sure you're logged into the XS CLI with a user with OrgManger role.  Check with "xs org-users HANAExpress"
```
xs create-buildpack my_python_buildpack my_python_buildpack 99
```

You python buildpack is now ready and you can test it with the example application.

See the README.md file in the tools/my_python_buildpack folder that got created for more information on how to control how the python buildpack functions.

### Building and testing the example app
Go into the HANA WebIDE and in the code view and at the Workspace level, right-click and select Git and then Clone Repository.
Enter this repository's Git HTTPS URL in the dialog and click Clone.

Once the git clone operation has completed, right-click on the project level and select Project Settings and then select Space in the list.
Select a space that had been set up with the di-space-enablement-ui that isn't the SAP space and click Save.

Build the db module.  Make sure it completed successfully by watching the console window.

You will need to create a service instance of the UAA service.  SSH into your server and target the space that you're deploying into.

Create the service instance required by this example by running the following command.

```
xs create-service xsuaa default mta-python-uaa
```

Build and run the js module.

Build and run the web module.

You can test the application at this point, but understand that the links to the python module will result in a 404 error until we build the python module manually.

### Manually building the python module

Although you have to this point cloned this repo within the WebIDE, the following steps require that you git clone the repository again to a place in the filesystem of your server where you can run some manual steps.  You may have done this already if you followed the instructions above for creating the python buildpack.

If you haven't already, ssh into your server as a normal user capable of running the XSA CLI.  Find a suitable folder where you can git clone this repo again(if you already haven't) and perform the following steps.

```
git clone <this repo URL>
cd <this repo folder>
```

Run the "xs a" command in the target space where you deployed the js and web modules.

```
xs a
```

You should should see 2 modules running, one ending in -hana-xsa-python-buildpack-js and one ending in -hana-xsa-python-buildpack-web.  Use the part of the name before this for the username and magic strings in the following instruction.

Edit the env.sh script and modify it to match your user and server-generated magic string.
```
vi tools/env.sh
```

Run the build script.

```
tools/build.sh
```

This will build the python module and then bind it to the uaa and hdi-container.  The first time the python buildpack is run it will read the VERSION file in it's folder and download the python source code and build it.  Subsequent runs will skip these steps.  It will then reconfigure the app-router to use the new python modules details in the destination.  You can check this by inspecting the environment of the app-router (USER-magic-web) app.

```
xs a | grep web | cut -d ' ' -f 1 | while read -r line ; do xs env $line | grep destinations ; done
```

Now when you click on the python links, they should not result in 404 errors as before.  Check the notes in the tools/build.sh script for details and the code in the python/server.py for comments on the python module's responsibilities.

### Configuration

At the buildpack compile stage (invoked during a push operation or as part of a deploy), the buildpack will read the python/runtime.txt file located in top folder of the python module source.  It then determines if that version is already compiled and if not, pulls the required source from the python source repository and attempts to build it.  If it has been successfully built before, this step is skipped.


### Limitations

This buildpack has been tested in online mode with the following Python versions on Linux based systems.


Python 2.7.11 

Python 2.7.13 (last known version in the Python2 series)

Python 3.6.2 (current stable version) as of 2017-09-07


If you require a different python version, specifiy it in the python/runtime.txt file as described in the configuration section.

Again as mentioned at the top of this README, this buildpack is for HANA XS Advanced usage only.

### Known Issues

Python 3.6.2 (current stable version) has an issue with python_jws with UnicodeDecodeError.

In order to overcome this issue, I cloned 

**https://github.com/brianloveswords/python-jws** to **https://github.com/alundesap/python-jws**

and adjusted the setup.py file.  In order to override the standard PIP behavior, I modifiled the **python/requirements.txt** file and replaced **jws** with **git+https://github.com/alundesap/python-jws.git/#egg=jws** and moved it before **python_jwt** since python_jwt also requires jws.

### How to obtain support

This buildpack is considered experimental and therefore does not qualify for SAP standard or paid support.  Please use the community support pages found at the [SAP Community Network](https://www.sap.com/community.html) and search with keywords "python" and "buildpack" to locate the latest information.

### License

This buildpack is licensed under [Apache License 2](/LICENSE).
