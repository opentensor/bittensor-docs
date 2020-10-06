Installation
==============


1. Install or make sure you have installed `Python3 <https://www.python.org/downloads/>`_.
2. Install or make sure you have installed `pip3 <https://pip.pypa.io/en/stable/installing/>`_.
3.  If you do not have it installed already, install `virtualenv <https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/#installing-virtualenv>`_.
4. If you wish to run Bittensor in dockerized containers, install `Docker Desktop`_.

   .. _`Docker Desktop`: https://www.docker.com/products/docker-desktop

5. Clone the `Bittensor <https://github.com/opentensor/bittensor>`__ repository locally: 

.. code-block:: Bash

   git clone git@github.com:opentensor/bittensor.git

6. If you wish to simply run a dockerized Bittensor instance, you may do so by running:

.. code-block:: Bash

   ./start_bittensor.sh

This will set up and bootstrap Bittensor in a local Docker container. If the process succeeds, 
you should see a "Bittensor" ASCII banner as the model is bootstrapped and run on docker. The default model that runs in this case is the "MNIST" dataset model. 
This is an example model that is set up under :code:`examples/mnist`. You can specify other models by using the :code:`-n, --neuron` flag.

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
you should see an ASCII "Bittensor" printed before the model starts training.
