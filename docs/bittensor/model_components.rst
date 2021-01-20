Components of a Bittensor Model
----------------------------------
Now that you've familiarized yourself with running Bittensor and the driver code, it's time we learn how to build a custom model to run on Bittensor. Inside each Synapse, the structure is rather similar with small changes according to modality. In this tutorial, we will cover the `FFNN.py <https://github.com/opentensor/bittensor/blob/master/bittensor/synapses/ffnn.py>`_ Synapse, which is a simple feed-forward neural network that is designed to classify the `MNIST <http://yann.lecun.com/exdb/mnist/>`_ dataset. `FFNN.py` contains the following functions:

.. automodule:: bittensor.synapses.ffnn
   :members: