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
network of interconnected models that are able to learn from each other. 

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

    Communication between two Bittensor neurons.

Let's also assume that as per the :doc:`getting started </getting-started/run-multiple-bittensor-instances>` guide, we started Neuron 1 first. 

Let's go through a breakdown of what happens when we start a Neuron. You can follow along in the `MNIST code <https://github.com/opentensor/bittensor/blob/master/examples/mnist/main.py#L198>`_.

1. Neuron loads its configuration (set by the command line argumnets) and dataset, then sets up the hyper parameters such as the learning rate, batch size, etc. 

    .. code-block:: Python

        # Additional training params.
        batch_size_train = 64
        batch_size_test = 64
        learning_rate = 0.01
        momentum = 0.9
        ...
        # Load (Train, Test) datasets into memory.
        train_data = torchvision.datasets.MNIST(root=model_toolbox.data_path, train=True, download=True, transform=transforms.ToTensor())
        trainloader = torch.utils.data.DataLoader(train_data, batch_size = batch_size_train, shuffle=True, num_workers=2)
        
        test_data = torchvision.datasets.MNIST(root=model_toolbox.data_path, train=False, download=True, transform=transforms.ToTensor())
        testloader = torch.utils.data.DataLoader(test_data, batch_size = batch_size_test, shuffle=False, num_workers=2)

2. Neuron builds the local synapse (this is the model to be trained on the network). 

    .. code-block:: Python

        # Build local synapse to serve on the network.
        model = MnistSynapse() # Synapses take a config object.
        model.to( device ) # Send model to device (GPU or CPU)

   where :code:`MnistSynapse` is a class that extends the :code:`Synapse` class. 
   In true Pytorch fasion, it contains an :code:`__init__` and a :code:`forward()` call. 

3. The Neuron will then build and start the :code:`Metagraph` object. 
   This object is responsible for connecting to the Blockchain
   and finding other neurons on the network.

    .. code-block:: Python

        metagraph = bittensor.Metagraph( config )
        metagraph.subscribe( model ) # Adds the synapse to the metagraph.
        metagraph.start() # Starts the metagraph gossip threads.

4. The Axon server is built next. This server allows other neurons 
   (Neuron 2 in this example) to make queries to Neuron 1 through 
   the Dendrite. It also deploys the synapse model we set up in step 2. 

    .. code-block:: Python

        axon = bittensor.Axon( config )
        axon.serve( copy.deepcopy(model) )
        axon.start() # Starts the server background threads. Must be paired with axon.stop().

5. Neuron builds the Dendrite object next. The Dendrite is 
   responsible for sending dataset batches across the 
   network to remote synapses.

    .. code-block:: Python

        dendrite = bittensor.Dendrite( config ).to(device)

6. Finally, the Neuron builds the router. The router is 
   responsible for learning **which** synapse to call. 
    
    .. code-block:: Python

        router = bittensor.Router(x_dim = 1024, key_dim = 100, topk = 10)

7. We can set up the optimizer the same way we normally do 
   with any other Pytorch model. The important piece here is that 
   we are optimizing both the model parameters **and** the 
   router's parameters, as we are dealing with two models here. 
   The Synapse model that we are training, and the router's 
   model that learns which synapse model to tell the Dendrite 
   to send a dataset batch to. 

    .. code-block:: Python

        # Build the optimizer.
        params = list(router.parameters()) + list(model.parameters())
        optimizer = optim.SGD(params, lr=learning_rate, momentum=momentum)

8. If we have previously saved a Bittensor model 
   and wish to continue training it, we can load it 
   back up using the :code:`model_toolbox`. 

    .. code-block:: Python

        # Load previously trained model if it exists
        if config._hparams.load_model is not None:
            model, optimizer, epoch, best_test_loss = model_toolbox.load_model(model, config._hparams.load_model, optimizer)
        
9. The training loop proceeds as it would with static 
   training models that we know and love in Pytorch. 
   However there is one difference now that they are 
   communicating with each other: we need to actually tell 
   them to talk to each other during training. Hence, 
   during the training loop we invoke the following lines: 

    .. code-block:: Python

         # Encode inputs for the router context.
         context = model.forward_image(images).to(device)
    
    Note that by :code:`context` here we mean the input coming from the local network, which is a vector output of its forward pass. This same context is used as input for the remote networks. 

   This invokes the model for one forward pass of the image inputs through the 
   :code:`MnistSynapse` model we defined. We then query the network
   of peers and send them the vector output of this forward pass and the current batch of examples that we are training on. 

    .. code-block:: Python

         # Query the remote network of peers
         synapses = metagraph.get_synapses( 1000 ) # Returns a list of synapses on the network (max 1000).
         requests, scores = router.route( synapses, context, images ) # routes inputs to network.
         responses = dendrite.forward_image( synapses, requests ) # Makes network calls.
         network = router.join( responses ) # Joins responses based on scores..

   Let's unpack this line by line: 
    - :code:`metagraph.get_synapses()` will simply query the network, and return a list of synapses that are presently on the network that we can query. Recall that a remote synapse is a model running on a remote neuron. 
    - :code:`router.route()` will utilize the router model to find which synapses to query that will return the best responses. It will return :code:`requests`: a list of input minibatches to be sent to the remote synapses, and :code:`scores`: a list of scores of the performance of the remote synapses.
    - :code:`dendrite.forward_image()` will forward the minibatches to the remote synapses, and return their responses -- a vector output of their :code:`forward` call. 
    - :code:`router.join()` will join the responses of all the synapses together. 
   
   **NOTE**: If we are running only one instance of Bittensor with no peers, then this call would simply recursively send the batch and vector output of the forward pass to the instance itself, effectively learning locally as if it's a local learning model.  

10. Now that we have the responses from all the remote synapses, we can call :code:`forward()` on the local model to calculate the following:

    - :code:`loss`: Total loss acumulation to be used by :code:`loss.backward()`.
    - :code:`local_output`: Output encoding of image inputs produced by using the local distillation model as context rather than the network. 
    - :code:`local_target`: MNIST Target predictions using student model as context.
    - :code:`local_target_loss`: MNIST Classification loss computed using the local_output, student model and passed labels.
    - :code:`network_target`: MNIST Target predictions using the network as context. 
    - :code:`network_output`: Output encoding of inputs produced by using the network inputs as context to the model rather than the student.
    - :code:`network_target_loss`: MNIST Classification loss computed using the local_output and passed labels.
    - :code:`distillation_loss`: Distillation loss produced by the student with respect to the network context.

    Remember that by :code:`context` here we mean the input coming from the remote network of neurons, which is a vector output of their own forward passes. The student distillation model learns to emulate this context from the network as well to increase accuracy and reduce loss. 

11. Finally, let's set the weights of the remote synapses. We first get the weights for list of Synapse endpoints, normalize them by the scores we calculated for the remote synapses, then we set them back. 
    This process is what allows a Bittensor neuron pick the best remote synapses (or peers) upon the next training iteration. 

    .. code-block:: Python

            weights = metagraph.getweights(synapses).to(device)
            weights = (0.99) * weights + 0.01 * torch.mean(scores, dim=0)
            metagraph.setweights(synapses, weights)