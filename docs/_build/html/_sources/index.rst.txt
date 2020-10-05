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

Installation
-------------

1. Install or make sure you have installed `Python3 <https://www.python.org/downloads/>`_.
2. Install or make sure you have installed `pip3 <https://pip.pypa.io/en/stable/installing/>`_.
3.  If you do not have it installed already, install `virtualenv <https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/#installing-virtualenv>`_.
4. If you wish to run BitTensor in dockerized containers, install `Docker Desktop`_.

   .. _`Docker Desktop`: https://www.docker.com/products/docker-desktop

5. Clone the `Bittensor <https://github.com/opentensor/bittensor>`_ repository locally: 

.. code-block:: Bash

   git clone git@github.com:opentensor/bittensor.git

6. If you wish to simply run a dockerized bittensor instance, you may do so by running:

.. code-block:: Bash

   ./start_bittensor.sh

This will set up and bootstrap BitTensor in a local Docker container. If the process succeeds, 
you should see a "BitTensor" ASCII banner as the model is bootstrapped and run on docker. The default model that runs in this case is the "MNIST" dataset model. 
This is an example model that is set up under :code:`examples/mnist`. You can specify other models by using the :code:`-n, --neuron` flag described in :ref:`single bittensor`

7. Create a new virtual environment and activate it:

.. code-block:: Bash

   python3 -m venv env
   source env/bin/activate

8. Install required packages:

.. code-block:: Bash

   pip3 install -r requirements.txt

9. You can verify your installation has succeeded by going to one of the examples and running its :code:`main.py` folder.

.. code-block:: Bash

   cd examples/mnist
   python3 main.py

Similarly to step 6, this will run the example with all the default flags. If everything installed as it should, 
you should see an ASCII "BitTensor" printed before the model starts training.

.. _single bittensor:

Start a single Bittensor Instance via Docker
--------------------------------------------

Bittensor contains a bash script (:code:`start_bittensor.sh`) that automatically pulls the latest image from Docker Hub 
and constructs a docker container that is able to run a given model. The parameters of :code:`start_bittensor.sh` are summarized as follows:

=============================   ====================================================================
   Flag                           Description
=============================   ====================================================================
:code:`-n, --neuron`             Which model (or "neuron") to run in :code:`examples/`. E.g. :code:`mnist`, :code:`cifar`, :code:`bert`, etc.
:code:`-l, --logdir`             Logging directory
:code:`-p, --port`               Bind side port for accepting requests.
:code:`-c, --chain_endpoint`     Bittensor chain endpoint.
:code:`-a, --axon_port`          Axon terminal bind port.
:code:`-m, --metagraph_port`     Metagraph bind port.
:code:`-s, --metagraph_size`     Metagraph cache size.
:code:`-b, --bootstrap`          Metagraph boot peer.
:code:`-k, --neuron_key`         Neuron Key.
:code:`-r, --remote_ip`          Remote serving IP.
:code:`-mp, --model_path`        Path to a saved version of the model to resume training.
=============================   ====================================================================

To start a dockerized version of a model, simply run the following: 

.. code-block:: Bash

   ./start_bittensor.sh

This will utilize the default values for all of the above command line arguments and run the network under :code:`examples/mnist` on :code:`localhost`.

Start a Single Bittensor Instance via native Python 
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

Start Multiple Bittensor Instances via Docker
-----------------------------------------------
The advantage bittensor is that it allows for *knowledge sharing* between models training together. This not only allows for more efficient training, 
but also increases the possibilities for us to develop resilient generalist models, as opposed to narrow specialists. 

Let's dive into how we can test running two models at the same time that can share knowledge with each other as they train. BitTensor utilizes 
`gRPC <https://grpc.io/>`_ communication to send `PIL <http://www.pythonware.com/products/pil/>`_ data, tensors, 
and gradients across the wire from one model to another model (also called a "peer").