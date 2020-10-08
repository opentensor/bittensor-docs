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

:code:`__init__(self, config: bittensor.Config)`
+++++++++++++++++++++++++++++++++++++++++++++++++

- Initializes the configuration according to arguments passed to :code:`main.py` in the Neuron.
- Initializes `gRPC <https://grpc.io/>`_ server objects using the :code:`config.axon_port` passed to it. 
- Sets up local synapses and the serving `Axon <https://github.com/opentensor/bittensor/tree/master/bittensor/axon.py>`_  thread. 

:code:`__del__(self)`
+++++++++++++++++++++
- Stops the entire Axon terminal server and deletes local synapses. 

:code:`start(self)`
++++++++++++++++++++
- Starts the Axon terminal and the gRPC server.

:code:`_stop(self)`
++++++++++++++++++++
- Stop the Axon terminal and the gRPC server.

:code:`_serve(self)`
++++++++++++++++++++
- Start the gRPC server.

:code:`serve(self)`
+++++++++++++++++++++
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
    - :code:`context` coming from a remote neuron.

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

Each function takes as input a list of peer synapses and a list of torch tensors that need to be sent to the peer. 

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

:code:`forward(self, synapses, x, mode)`
+++++++++++++++++++++++++++++++++++++++++