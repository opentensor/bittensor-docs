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
====================   ====================================================================

Start Bittensor via native Python on a single machine
-------------------------------------------------------

First, ensure that you have :code:`Python 3.5` or above to run Bittensor. Once you have done so, you can move to the 
:code:`examples` directory and pick a model to run. You can run the model directly by calling: 

.. code-block:: Bash

   python main.py

This will run a single instance of the model that will not speak to any peers with default parameters. The parameters for
running a bittensor model are as follows:

=========================   ====================================================================
   Flag                           Description
=========================   ====================================================================
:code:`--chain_endpoint`      Bittensor chain endpoint.
:code:`--axon_port`           Axon terminal bind port.
:code:`--metagraph_port`      Metagraph bind port.
:code:`--metagraph_size`      Metagraph cache size.
:code:`--bootstrap`           Metagraph boot peer.
:code:`--neuron_key`          Neuron Key.
:code:`--remote_ip`           Remote serving IP.
:code:`--model_path`          Path to a saved version of the model to resume training.
=========================   ====================================================================


To run a model on a given 
port number (e.g. 8120) that peers can connect to, you can run it as follows:

.. code-block:: Bash

   python main.py --metagraph_port 8120

Once this model is running, you can run another model that bootstraps itself to port 8120 as follows:

.. code-block:: Bash

   python main.py --bootstrap '0.0.0.0:8120'

This will run another instance of the model that will communicate with the first model and they will begin to share knowledge with each other. 