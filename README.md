Jupyter Notebooks for OpenShift
===============================

This repository contains software to make it easier to use Jupyter Notebooks on OpenShift.

This includes input source code for creating a minimal Jupyter notebook image using the Source-to-Image (S2I) build process. The image can be built in OpenShift, or separately using the ``s2i`` tool.

The minimal Jupyter notebook image can be deployed to create an empty Jupyter notebook workspace in OpenShift that you can work with. The same image, can also be used as a S2I builder to create customised Jupyter notebook images with additional Python packages installed, or notebook files preloaded.

Building the Minimal Notebook
-----------------------------

To build the minimal Jupyter notebook run the command:

```
oc create -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyter-notebooks/master/resources.json
```

This will create a build configuration in your OpenShift project to build the minimal Jupyter notebook image using the Python 3.5 S2I builder. You can watch the progress of the build by running:

```
oc logs --follow bc/minimal-notebook
```

A tagged image ``minimal-notebook:3.5`` should be created in your project.

The command to create the minimal Jupyter notebook image above will also load a template for creating custom notebook images. This will be used below.

Deploying the Minimal Notebook
------------------------------

To deploy the minimal Jupyter notebook image run the following commands:

```
oc new-app minimal-notebook:3.5 --name mynotebook \
    --env JUPYTER_NOTEBOOK_PASSWORD=mypassword
```

The ``JUPYTER_NOTEBOOK_PASSWORD`` environment variable will allow you to access the notebook instance with a known password.

Deployment should be quick, but you can monitor progress if necessary by running:

```
oc rollout status dc/mynotebook
```

Because the notebook instance is not exposed to the public network by default, you will need to expose it. To do this, and ensure that access is over a secure connection run:

```
oc create route edge mynotebook --service mynotebook \
    --insecure-policy Redirect
```

To see the hostname which is assigned to the notebook instance, run:

```
oc get route/mynotebook
```

Access the hostname shown using your browser and enter the password you used above.

To delete the notebook instance when done, run:

```
oc delete all --selector app=mynotebook
```

Creating Custom Notebook Images
-------------------------------

To create custom notebooks images, you can use the ``minimal-notebook:3.5`` image as a S2I builder. This is preferably done using a template which was also loaded when creating the initial minimal Jupyter notebook image above.

Two examples of custom notebook images are included with this repository. These can be used to create custom notebook images similar to the ``scipy-notebook`` and ``tensorflow-notebook`` images provided by the Jupyter project. The examples will only include Python 3.5 support and do not include Python 2.7 support in the same image.

To build the ``scipy-notebook`` image, run:

```
oc process notebook-builder \
  --param NOTEBOOK_NAME=scipy-notebook \
  --param GIT_REPOSITORY_URL=https://github.com/jupyter-on-openshift/jupyter-notebooks \
  --param CONTEXT_DIR=scipy-notebook | oc create -f -
```

If any build of a custom image fails because the default memory limit on builds in your OpenShift cluster is too small, increase the limit by running:

```
oc patch bc/scipy-notebook \
  --patch '{"spec":{"resources":{"limits":{"memory":"1Gi"}}}}'
```

and start a new build by running:

```
oc start-build bc/scipy-notebook
```

Once the build is complete, to build the ``tensorflow-notebook`` image, run:

```
oc process notebook-builder \
  --param NOTEBOOK_NAME=tensorflow-notebook \
  --param GIT_REPOSITORY_URL=https://github.com/jupyter-on-openshift/jupyter-notebooks \
  --param BUILDER_IMAGE=scipy-notebook:latest \
  --param CONTEXT_DIR=tensorflow-notebook | oc create -f -
```

In the case of the ``tensorflow-notebook`` image, it is layered on top of the ``scipy-notebook`` image by using the ``scipy-notebook`` image as the S2I builder.

The directory in the Git repository the S2I build is run against can contain a ``requirements.txt`` file listing the Python package to be installed in the custom notebook image. Any other files in the directory will also be copied into the image. When the notebook instance is started from the image, those files will then be present in your workspace.

As an additional example, if you want to create a custom notebook image which includes the notebooks from Jake Vanderplas' book found at:

* https://github.com/jakevdp/PythonDataScienceHandbook

run:

```
oc process notebook-builder \
  --param NOTEBOOK_NAME=jakevdp-notebook \
  --param GIT_REPOSITORY_URL=https://github.com/jakevdp/PythonDataScienceHandbook | oc create -f -
```

Using the OpenShift Web Console
-------------------------------

The notebook images can also be deployed from the web console by selecting _Deploy Image_ from the _Add to Project_ menu, and then choosing the image from _Image Stream Tag_.

To build a custom notebook image from the web console, select _Browse Catalog_ from _Add to Project_. Enter ``jupyter`` in the search field and then select _Jupyter Notebook Builder_.
