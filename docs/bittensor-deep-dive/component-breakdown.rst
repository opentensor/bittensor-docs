Bittensor Component Breakdown
===============================

Bittensor consists of several components that allow neurons to communicate with peers (other neurons) 
and send/receive machine knowledge. This section goes through each component and describes its role in the neuron, its inputs
, its outputs, and its context within the big picture of the Bittensor network. 

These components can all be found in `bittensor/bittensor <https://github.com/opentensor/bittensor/tree/master/bittensor>`_.

`Axon <https://github.com/opentensor/bittensor/tree/master/bittensor/axon.py>`_ class 
---------------------------------------------------------------------------------------

Analogous to the Axon terminal in a biological neuron cell, the axon class is responsible 
for deploying a Synapse instance and receiving machine knowledge from other synapses in the network 
(also called remote synapses, we will be using these terms interchangeably).  

The main job of the Axon class is to process :code:`forward` and :code:`backward` requests through a
set of local synapses. This requests come from remote neurons. 

:code:`__init__ (self, config: bittensor.Config)`
+++++++++++++++++++++++++++++++++++++++++++++++++

- Initializes the configuration according to arguments passed to :code:`main.py` in the Neuron.
- Initializes `gRPC <https://grpc.io/>`_ server objects using the :code:`config.axon_port` passed to it. 
- Sets up local synapses and the serving `Axon <https://github.com/opentensor/bittensor/tree/master/bittensor/axon.py>`_  thread. 

.. _del1:

:code:`__del__ (self)`
+++++++++++++++++++++++
- Stops the entire Axon terminal server and deletes local synapses. 

.. _start1:
    :code:`start (self)`
    ++++++++++++++++++++++
    - Starts the Axon terminal and the gRPC server.

:code:`_stop (self)`
++++++++++++++++++++++
- Stop the Axon terminal and the gRPC server.

:code:`_serve (self)`
++++++++++++++++++++++
- Start the gRPC server.

:code:`serve (self)`
+++++++++++++++++++++++
- Add a synapse to the set of synapses presently being served by this Axon terminal.
- Creates a new bittensor_pb2.Synapse proto and adds it to the list of local synapses.

:code:`Forward (self, request, context)`
++++++++++++++++++++++++++++++++++++++++

The :code:`Forward` call takes the :ref:`TensorMessage <tensor-message>` request and looks up the local synapse that this request
is targetting. In this context, the local synapse is a synapse belonging to this neuron. Once it has found the synapse,
it deserializes the Tensor in the :ref:`TensorMessage <tensor-message>` and makes a :code:`forward` call on the target synapse. It then 
serializes the local synapse's response (a vector of the output of its last layer) and returns it. 

**Inputs**:
    - :ref:`TensorMessage request <tensor-message>` coming from a remote synapse.
    - :code:`congrpc.ServicerContext context` : coming from a remote neuron.

**Outputs**:
    - Serialized response of the local synapse.


`Dendrite <https://github.com/opentensor/bittensor/tree/master/bittensor/dendrite.py>`_ class 
------------------------------------------------------------------------------------------------

Also analogous to the Dendrite in a biological neuron cell, the Dendrite class is responsible for 
sending dataset batches across the network to remote synapses.

:code:`forward_text`, :code:`forward_image`, and :code:`forward_tensor`
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

As the function name implies, each of these functions is responsible for forwarding a different modality 
across the `gRPC <https://grpc.io/>`_ network to peer neurons. 

Each function takes as input a list of peer synapses and a list of torch tensors that need to be sent to the peer. The list
of peer synapses can be retrieved with a call to :code:`metagraph.get_synapses()` as described in :ref:`Bittensor Architecture <bittensor-network>`.

.. code-block:: Python

    def forward_text(self, synapses: List[bittensor_pb2.Synapse], x: List[ List[str] ]) -> List[torch.Tensor]:
        """ forward tensor processes """
        return self.forward(synapses, x, bittensor_pb2.Modality.TEXT)
    
    def forward_image(self, synapses: List[bittensor_pb2.Synapse], x: List[ torch.Tensor ]) -> List[torch.Tensor]:
        """ forward tensor processes """
        return self.forward(synapses, x, bittensor_pb2.Modality.IMAGE)
    
    def forward_tensor(self, synapses: List[bittensor_pb2.Synapse], x: List[ torch.Tensor ]) -> List[torch.Tensor]:
        """ forward tensor processes """
        return self.forward(synapses, x, bittensor_pb2.Modality.TENSOR)

:code:`forward (self, synapses, x, mode)`
+++++++++++++++++++++++++++++++++++++++++
The main functionality behind the Dendrite component of Bittensor, the :code:`forward()` call forwards mini-batches of the dataset
by invoking :code:`_RemoteModuleCall` out to the target remote synapses and returns their results. 

**Inputs**:
    - :code:`List[bittensor_pb2.Synapse] synapses` : A list of remote synapses running in remote neurons.
    - :code:`List[ object ] x` : A list of objects to be sent through gRPC to the target remote synapses.
    - :code:`bittensor_pb2.Modality mode` : Modality of the data being sent (i.e. Text, Image, etc.)

**Outputs**:
    - The response of each remote synapse. Given that each minibatch runs through the remote synapse's local model, the output back 
      is a vector output of that remote synapse's :code:`forward()` call. 


:code:`RemoteSynapse` class
++++++++++++++++++++++++++++
This class bundles a grpc connection to a remote neuron as a standard auto-grad torch.nn.Module. Making it possible to send data to
that neuron and even calculate its partial derivative on a backwards pass. 

:code:`__init__ (self, synapses, config)`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This function sets up the synapse's remote address and a gRPC channel for communication. 

**Inputs**:
    - :code:`bittensor_pb2.Synapse synapse`: The target remote synapse to which we are bundling a gRPC connection.
    - :code:`bittensor.Config config` : Run configuration of the local neuron. 

**Outputs**:
    - :code:`None`

:code:`forward (self, object, mode)`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Makes a :code:`_RemoteModuleCall` to send the batch of inputs over to the remote synapse. 

**Inputs**:
    - :code:`Object inputs` : The mini-batch of data to be sent to the remote synapse. Note that it is of type :code:`object` as 
                              it can be of type string or image data. More data types will be integrated in later versions.
    - :code:`mode` : The modality of the data being sent.

**Outputs**
    - :code:`torch.Tensor outputs` : A Tensor of outputs from the remote synapses. 

:code:`_RemoteModuleCall` class
++++++++++++++++++++++++++++++++++
This class is responsible for invoking the actual stub to call remote synapses. As an Autograd differentiable function, the forward 
call passes tensors, while the backward pass computes the gradients to get the partial derivative of the remote synapses' output.  

:code:`forward (ctx, RemoteSynapse, dummy, inputs, mode)`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This call passes tensors to the remote synapse. Note that it also takes a :code:`dummy` tensor to prevent torch from excluding the synapse from backward if no other inputs require grad. 
This method first serializes the inputs, sends them across the wire, then deserializes the outputs of the remote synapses. 

**Inputs**:
    - :code:`ctx` : Object that can be used to stash information for backward computation.
    - :code:`RemoteSynapse caller` : The calling synapse of type :code:`RemoteSynapse`. 
    - :code:`torch.Tensor dummy` : A dummy tensor to trigger autograd in the remote synapse. 
    - :code:`List[ object ] x` : A list of objects to be sent through gRPC to the target remote synapses.
    - :code:`bittensor_pb2.Modality mode` : Modality of the data being sent (i.e. Text, Image, etc.)

**Outputs**:
    - The response of each remote synapse. Given that each minibatch runs through the remote synapse's local model, the output back 
      is a vector output of that remote synapse's :code:`forward()` call. 


:code:`backward (ctx, grads)`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The backward function pass computes the gradients of the remote synapse's outputs to get their partial derivative (that is, the gradients). However,
at the moment this function does not apply the gradients to the Synapse. Note that if the modality is text, then the derivative would be 0 and thus it returns a
:code:`none` tuple. However if the modality is :code:`Image`, :code:`Tensor`, or otherwise, then the gradients of the remote synapses are returned. 

**Inputs**:
    - :code:`ctx` : Object that can be used to stash information for backward computation.
    - :code:`torch.Tensor grads` : gradients of the local neuron.

**Outputs**:
    - The response of each remote synapse. 


`Metagraph <https://github.com/opentensor/bittensor/tree/master/bittensor/metagraph.py>`_ class 
------------------------------------------------------------------------------------------------
The Metagraph class is responsible for allowing Neurons to connect to each other and -- later -- to the blockchain, however this is still under development. 

:code:`__init__(self, config)`
++++++++++++++++++++++++++++++++++
Initializes a new metagraph proof-of-work object cache, initializes gRPC metagraph servicer, and loads config or default configs. 

:code:`start (self)`
++++++++++++++++++++++++++++++++++
Starts two threads:

    1. The gossip thread that will communicate with and find potential peers on the network. Target method :code:`_update()`.
    2. The gRPC server thread.

:code:`update(self)`
+++++++++++++++++++++++++++++++++
This keeps the metagraph record of the local neuron up to date by discovering peers using gossip protocol and removing stale peers. 

:code:`stop (self)`
+++++++++++++++++++++++++++++++++++
Stops the gossip thread. This is typically called at the end of Bittensor's execution cycle as part of cleanup.

:code:`do_gossip (self)`
+++++++++++++++++++++++++++++++++++
Sends a gossip query to a random peer and records their response into the metagraph via a :code:`sink()` call.

:code:`do_clean (self, ttl)`
+++++++++++++++++++++++++++++++++++++++++++
Checks whether a peer hasn't sent a heart beat signal in a given time (i.e. timed out). If it hasn't, then the neuron deletes it from its list of peers.

**Inputs**:
    - :code:`Integer ttl`: Time to live for a given target neuron (defaults to 5 minutes).
 
**Outputs**:
    - :code:`None`


:code:`_sink (self, request)`
++++++++++++++++++++++++++++++++++++++++++
Records a peer's gossip request to the local neurons list of synapses and peers, and sets its latest heartbeat time. 

:code:`get_peers(self, n)`
++++++++++++++++++++++++++++
Returns the number of active peer endpoints in the network. 

**Inputs**:
    - :code:`Integer n` : maximum number of peers to return. This defaults to 10 peers. 

**Outputs**:
    - :code:`None`

:code:`get_synapses(self, n)`
+++++++++++++++++++++++++++++++
Returns the number of active synapses in the network. 

**Inputs**:
    - :code:`Integer n` : maximum number of synapses to return. This defaults to 1000 synapses. 

**Outputs**:
    - :code:`None`

:code:`getweights(self, synapses)`
+++++++++++++++++++++++++++++++++++
Retrieves the weights for a list of synapses in the network. 

**Inputs**:
    - :code:`List[bittensor_pb2.Synapse] synapses` : List of remote synapse endpoints. 

**Outputs**:
    - :code:`List(Float) results` : List of remote synapse weights. 

:code:`setweights(self, synapses)`
+++++++++++++++++++++++++++++++++++
Sets the weights for a list of synapses in the network. 

**Inputs**:
    - :code:`List[bittensor_pb2.Synapse] synapses` : List of remote synapse endpoints. 
    - :code:`torch.Tensor weights` : Weights to set to each synapse. 

**Outputs**:
    - :code:`None`


:code:`subscribe(self, synapse)`
+++++++++++++++++++++++++++++++++++
Subscribes a synapse class object to the neuron's metagraph.

**Inputs**:
    - :code:`bittensor_pb2.Synapse synapses` : Synapse to subscribe to metagraph.

**Outputs**:
    - :code:`None`

`Synapse <https://github.com/opentensor/bittensor/tree/master/bittensor/synapse.py>`_ class 
------------------------------------------------------------------------------------------------
This is the **superclass** of every custom deep learning model that users will write on Bittensor. More specifically,
each deep learning model (read: neuron, or miner) will extend this class to gain the capabilities of communicating 
with other bittensor neurons. 


:code:`__init__ (self)`
+++++++++++++++++++++++++++++++++++
Initializes the :ref:`Synapse public key <tensor-message>` and identifies whether the current machine is cuda-capable (i.e. can we
train on the GPU?).


:code:`forward_text (self)`, :code:`forward_image (self)`, and :code:`forward_tensor (self)`
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
These methods are not implemented in this class, but are defined as contractual methods where one (or all) of them should be implemented by the user and
their model they are building (See examples like `MNIST <https://github.com/opentensor/bittensor/tree/master/examples/mnist>`_).

**Inputs**:
    - :code:`torch.Tensor inputs` : The batch of data to be sent through the :code:`forward()` call of the custom model.

:code:`call_forward (self, inputs, modality)`
++++++++++++++++++++++++++++++++++++++++++++++++
Apply forward pass to the bittensor.synapse given inputs and modality. The modality passed to this function determines whether it calls
:code:`forward_text (self)`, :code:`forward_image (self)`, or :code:`forward_tensor (self)`. Therefore, one of them **must** be implemented in the subclass (custom) model. 

**Inputs**:
    - :code:`Object inputs` : The batch of data to be sent through the :code:`forward()` call of the custom model.
    - :code:`modality` : The modality of the data being sent.

**Outputs**:
    - :code:`torch.Tensor Outputs` : The output of the :code:`forward()` call of the model. 

:code:`call_backward (self, inputs, grads)`
++++++++++++++++++++++++++++++++++++++++++++++++
Apply a backward pass to the synapse given grads and inputs. Presently, this only returns a tensor of zeros as it is not being used. 

**Inputs**:
    - :code:`Object inputs` : The batch of data to be sent through the :code:`forward()` call of the custom model.
    - :code:`grads` : Gradients of the remote synapses.

**Outputs**:
    - :code:`torch.zeros((1,1))` : A tensor of zeros. 

