gRPC Protocol
===============
The BitTensor `gRPC <https://grpc.io/>`_ protocol definition can be 
found `here <https://github.com/opentensor/bittensor/blob/master/bittensor/bittensor.proto>`__. The following define
the communications protocol and message types that travel between nodes/instances of Bittensor.

1. :code:`DataType` enum. 
    - Used for serialization/deserialization of messages as they travel through the wire.
    - Data types can be  :code:`UNKNOWN`, :code:`FLOAT32`, :code:`INT32`, :code:`INT64`, or :code:`UTF8`.
    - Described `here <https://github.com/opentensor/bittensor/blob/master/bittensor/bittensor.proto#L230>`__. 

2. :code:`Modality` enum.
    - Modality of the message (TEXT, IMAGE, TENSOR).
    - Described `here <https://github.com/opentensor/bittensor/blob/master/bittensor/bittensor.proto#L239>`__.


3. :code:`TensorMessage` message.
    - The primary protobuf message object passed between tensor processing servers.
    - Contains a payload of 1 or more serialized tensors and their definitions.
        - **version** –  Indentifies protocol version for backward compatibility.
        - **neuron_key** – Public key of the caller.
    - Also contains information to identity and verify the sender.

    =======================================  =============================================================
        TensorMessage Payload                           Description
    =======================================  =============================================================
    :code:`STRING` version [required]           Identifies protocol version for backward compatibility.
    :code:`STRING` neuron_key [required]        | Public key of the calling server. This is used to 
                                                | identify the calling server (i.e. neuron) to 
                                                | the synapse of the receiving server (i.e. 
                                                | the receiving synapse).
    :code:`STRING` synapse_key [required]       | Public key of the receiving synapse. This is used to 
                                                | identify which synapse to query behind the endpoint, 
                                                | as an endpoint may have multiple synapses.
    :code:`INT64`  nounce [optional]            | Incrementing nounce to identify message ordering. 
                                                | Used ensure with signature to protect against 
                                                | spoofing attacks.   
    :code:`BYTES`  signature [optional]         | Digital signature linking the nounce, neuron_key 
                                                | and synapse_key. This prevents spoofing attacks 
                                                | where an adversary sends messages pretending 
                                                | to be other peers.
    :code:`TENSOR` tensors [required]           | 1 or more tensors passed on the wire. This is the 
                                                | whole point of having a TensorMessage, 
                                                | and thus is required. 
    =======================================  =============================================================

4. :code:`Tensor` message.
    - A serialized tensor object created using the serializer class.
    - Essentially used to describe a tensor being passed on the wire.

    =======================================  =================================================================
        Tensor Payload                           Description
    =======================================  =================================================================
    :code:`STRING` version [required]           Identifies protocol version for backward compatibility.
    :code:`BYTES` buffer [required]             | Serialized raw tensor content. Since this is serialized, 
                                                | this representation can be used for all tensor types. 
                                                | The purpose of this serialization is to reduce overhead 
                                                | during RPC calls by avoiding the serialization of 
                                                | many repeated small items. 
    :code:`repeated INT64` shape [required]     Tensor shape.
    :code:`DataType` dtype [required]           | The tensor datatype. Used for serialization 
                                                | and deserialization.
    :code:`Modality` modality [required]        Modality of the message (TEXT, IMAGE, TENSOR)
    :code:`bool` requires_grad [optional]       Whether or not this tensor require a gradient.
    =======================================  =================================================================

5. :code:`Synapse` message.
    - "Synapse" or "Expert" endpoint definition.
    - This fully describes a tensor processing service for Bittensor (as well as `hivemind <https://learning-at-home.readthedocs.io/en/latest/?badge=latest>`_).
    
    =======================================  =================================================================
        Synapse Payload                           Description
    =======================================  =================================================================
    :code:`STRING` version [required]           Identifies protocol version for backward compatibility.
    :code:`STRING` neuron_key [required]        | Public key of the calling neuron. 
                                                | This is used to identify the calling server (i.e. neuron) to 
                                                | the synapse of the receiving server 
                                                | (i.e. the receiving synapse). Links this synapse definition 
                                                | to the containing neuron-account.
    :code:`STRING` synapse_key [required]       | Public key of this synapse. 
                                                | This is used to identify which synapse to query 
                                                | behind the endpoint, as an endpoint may have 
                                                | multiple synapses.
    :code:`STRING` address [required]           Synapse ip address. 
    :code:`STRING` port [required]              Synapse endpoint listening port.
    :code:`STRING` block_hash [required]        Hash of latest chain block service definition creation.
    :code:`STRING` nounce [optional]            Incrementing nounce.            
    :code:`STRING` proof_of_work [optional]     | Work hash which ensures service definition creation 
                                                | was computationally expensive. Protects the network 
                                                | cache from flooding attacks.
    :code:`STRING` signature [optional]         | Public_key signature for this synapse definition. 
                                                | Ensures that the connected neuron_key signed this proto.
    =======================================  =================================================================
