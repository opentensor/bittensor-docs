Run Bittensor via Python
===========================

First, ensure that you have :code:`Python 3.5` or above to run Bittensor. Once you have done so, you can move to the 
:code:`examples` directory and pick a model to run. You can run the model directly by calling: 

.. code-block:: Bash

   python main.py

This will run a single instance of the model that will not speak to any peers with default parameters. The parameters for
running a Bittensor model are as follows:

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