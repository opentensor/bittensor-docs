Running Multiple Bittensor Instances 
=======================================

The advantage of Bittensor is that it allows for *knowledge sharing* between models training together. This not only allows for more efficient training, 
but also increases the possibilities for us to develop resilient generalist models, as opposed to narrow specialists. 

Let's dive into how we can test running two models at the same time that can share knowledge with each other as they train. Bittensor utilizes 
`gRPC <https://grpc.io/>`_ communication to send `PIL <http://www.pythonware.com/products/pil/>`_ data, tensors, 
and gradients across the wire from one model to another model (also called a "peer"). We will first show how we can achieve this using Docker containers, and 
then show the same process using native Python commands.

Each Bittensor instance (native Python or Dockerized) requires some specific ports to be opened up for it to be able to communicate 
with other instances. These are:

1. **Axon Port** : This port is where communications are received. If it is not opened, then your bittensor instance cannot receive gRPC communications 
from its peers. This is automatically exposed on a random port number on the Docker container via the :code:`start_bittensor.sh` script if 
running on Docker, and via Python as well if running on native Python. Hence, there is typically no need to set it yourself unless
you are going after some rather specific architectural setup for your network.

2. **Metagraph Port** : This port is largely for testing purposes, and is responsible for specifying which metagraph port to connect to. 
In a nutshell, the metagraph is the connecting fabric between all the nodes/models and acts as a "simulated" chain when a blockchain is not present. 
When testing locally, the first instance you run needs to specify its metagraph port.

3. **Bootstrap port** : When testing, each node will need to bootstrap itself to a peer so they can begin the communication process. When
testing locally on Docker, the IP of this peer automatically resolves to :code:`host.docker.internal`. When testing via native Python, 
the IP is specified as :code:`0.0.0.0`. When testing locally, this port needs to be set to be the same as the metagraph port that the 
first instance you launched is using.

Via Dockerized Containers
---------------------------

To run multiple instances via Docker containers, we must expose the Axon, Metagraph, and Bootstrap ports. We must also first create an instance
that binds itself to the Metagraph via a given port, and the second instance must then bootstrap itself to the first instance to 
begin communication. 

Let's start by running the first instance and binding it to metagraph port :code:`8120`.

.. code-block:: Bash

   ./start_bittensor.sh -m 8120

This starts a Dockerized instance of Bittensor bound to metagraph port :code:`8120` without a bootstrapped peer. 

The training output for each iteration will look like the following:

.. code-block:: Bash

    2020-10-06 14:21:59.100 | INFO     | __main__:train:291 - Train Epoch: 0 [0/60000 (0%)]	Local Loss: 2.306969	Target Loss: 2.306806	Distillation Loss: 0.027752	nP|nS: 0|1

Note that the very last values of this line: :code:`nP|nS: 0|1`. This indicates the number of peers (:code:`nP`) and the number of Synapses (:code:`nS`), at the moment there are no peers
and one synapse that belongs to this instance that we've just launched. 

Now let's launch another instance that will communicate with the initial instance we just launched. The appropriate bootstrap IP is
automatically determined when running locally, so we just need to specify the :code:`bootstrap_port`. Open another terminal and run the following call:

.. code-block:: Bash

   ./start_bittensor.sh -b 8120

Now if we take a look at the training output, note that :code:`nP` and :code:`nS` changed to :code:`nP|nS: 1|2` as there is one peer and two synapses firing now.

.. code-block:: Bash

    2020-10-06 14:22:08.616 | INFO     | __main__:train:291 - Train Epoch: 0 [0/60000 (0%)]	Local Loss: 2.299487	Target Loss: 2.299433	Distillation Loss: 0.038577	nP|nS: 1|2

This indicates that our new instance has bootstrapped successfully to the first instance that we launched, and is communicating with it appropriately. 

Via Native Python 
--------------------

The process to run multiple instances via native Python is very similar to the Dockerized example, the only difference being the syntax of the 
command line arguments as we are sending them directly to Python in this case. As in the Docker container example, we first create an instance
that binds itself to the Metagraph via a given port, and the second instance must then bootstrap itself to the first instance to 
begin communication. 

Let's start by running the first instance and binding it to metagraph port :code:`8120`.

.. code-block:: Bash

    python3 examples/mnist/main.py --metagraph_port=8120

Now let's launch another instance that will communicate with the initial instance we just launched. The difference from Docker, however, 
is that the appropriate bootstrap IP is **not** determined automatically now, so we must specify it along with the bootstrap port. Open 
another terminal and run the following call:

.. code-block:: Bash

    python3 ./examples/mnist/main.py --bootstrap='0.0.0.0:8120' 

The output from the training loops both instances will be the same as the dockerized versions:

The training output for each iteration of the first instance will look like the following:

.. code-block:: Bash

    2020-10-06 14:21:59.100 | INFO     | __main__:train:291 - Train Epoch: 0 [0/60000 (0%)]	Local Loss: 2.306969	Target Loss: 2.306806	Distillation Loss: 0.027752	nP|nS: 0|1

whereas the training output for each iteration of the second (boostrapped) instance will look like:

.. code-block:: Bash

    2020-10-06 14:22:08.616 | INFO     | __main__:train:291 - Train Epoch: 0 [0/60000 (0%)]	Local Loss: 2.299487	Target Loss: 2.299433	Distillation Loss: 0.038577	nP|nS: 1|2
