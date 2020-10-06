Run Bittensor via Docker
============================================


Since Docker containers encapsulate everything an application needs to run (and only those things), any host with the Docker runtime 
installed (be it a developer's laptop or a public cloud instance) can run a Docker container. This is 
very useful when deploying your model to a cloud GPU or some remote peer if you do not wish to handle
the hassle of setting up an environment locally. Hence, once a model has been trained and is ready to be 
deployed onto the Bittensor network, you may wish to use Docker to deploy it somewhere instead of a native Python call. 
For this purpose, Bittensor contains a bash script (:code:`start_bittensor.sh`) that automatically pulls the latest 
image from Docker Hub and constructs a docker container that is able to run a given model. The parameters of :code:`start_bittensor.sh` are summarized as follows:

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
