Bittensor Architecture
========================

The inspiration for Bittensor came from brain cells (called neurons). In the same way that a network
of neurons consists of multiple neurons communicating with each other continuously, so does the Bittensor
network consist of multiple machine learning models (also called neurons) communicating with each other.

Therefore to help in understanding the Bittensor network architecture, we use 
biological neurons as inspiration and an analogous example. 

Biological Inspiration
------------------------

The Figure below describes a biological neuron that contains three important parts:

1. **Dendrite**: responsible for receiving information from other neurons. 
2. **Axon Terminal**: responsible for sending information to other neurons. 
3. **Soma**: Contains the nucleus and other structures common to living cells. 

These three parts together enable a neuron to exchange messages with each other and form a 
massive interconnected network that -- on the grand scale -- form a brain. Note that
this is a highly simplified explanation, and biological neurons are significantly more complex. 

.. figure:: 
    /images/bio_neuron.jpg

    Biological neuron cell. Credit: David Baillot/ UC San Diego.

The Bittensor Network
------------------------

Similarly to its biological counterpart, the Bittensor neuron also exchanges messages with other 
neurons in the form of `machine knowledge`. A group of these neurons can connect together to form a 
network of interconnected models that are able to learn form each other. 

Bittensor Neuron examples can be found under :code:`examples`. Presently, there are 3 examples: 

- `MNIST <https://github.com/opentensor/bittensor/tree/master/examples/mnist>`_ : The most basic neuron example. A simple feed-forward network that takes 32x32 images as input. 

- `CIFAR <https://github.com/opentensor/bittensor/tree/master/examples/cifar>`_ : A more complicated Dual Path Neural Network (As described by `Chen et al. <https://arxiv.org/abs/1707.01629>`_) that takes as input the CIFAR-10 dataset.

- `BERT <https://github.com/opentensor/bittensor/tree/master/examples/bert>`_ : A complete implementation of `Huggingface's bert model <https://huggingface.co/transformers/>`_. 

For the rest of this document, we will refer to **MNIST** during our examples as it is the most straightforward network, and hence 
easier to understand the architecture of Bittensor through it. 

Assume we have two neurons as in the figure below, where each neuron contains 3 main components:

1. **Axon Terminal**: Responsible for deploying a synapse and receiving information coming from a remote synapse. 

2. **Dendrite**: Responsible for sending information to a remote synapse. 

3. **Neuron**: Effectively the "soma" of a bittensor node. Contains the local model as well as the training and testing logic.  

.. figure:: 
    /images/Bittensor.png

    Bird's-eye view of the communication between two Bittensor nodes.