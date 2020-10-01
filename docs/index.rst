.. Bittensor Documentation documentation master file, created by
   sphinx-quickstart on Thu Oct  1 15:20:19 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Bittensor Documentation
===================================================
Welcome to the official documentation of Bittensor, the peer-to-peer machine intelligence framework. 

.. toctree::
   :maxdepth: 2
   :caption: Contents:



Getting Started
==================
There are two ways to start BitTensor: via a native Python command, or via a dockerized instance. Typically, the dockerized instance is recommended 
when training a full model to completion, whereas the Python command is best for debugging purposes. 

Start Bittensor via docker
---------------------------

Bittensor has a bash script that automatically pulls the latest image from Docker Hub 
and constructs a docker container that is able to run a given model.

.. code-block:: Bash

   ./start_bittensor.sh

By default, the script will run the MNIST example network under :code:`examples/mnist` on :code:`localhost`.
However, it is possible to specify the parameters of the run using the following flags:

====================   ====================================================================
   Flag                           Description
====================   ====================================================================
:code:`n, --neuron`     Which model (also called a neuron) to run. E.g. MNIST, CIFAR, etc.
:code:`-l, --logdir`    Logging directory
:code:`-p, --port`      Bind side port for accepting requests.
:code:`-r, --remote`    Run instance locally or remotely
:code:`-t, --token`     Digital ocean API token
====================   ====================================================================

