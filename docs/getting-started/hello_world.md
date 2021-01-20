# Hello, Machines!
With Bittensor installed, let's try running one of the examples! Each example is basically driver code for the models you build inside [bittensor/synapses](https://github.com/opentensor/bittensor/tree/master/bittensor/synapses). This will give you an overview to the development flow of Bittensor as well as what to expect when creating new models. 

For this example, we will be running `examples/gpt2-wiki.py` which is exactly what it sounds like: a very small version of [HuggingFace OpenAI GPT2](https://huggingface.co/transformers/model_doc/gpt2.html). 


Each example is powered by a [`Synapse`](https://github.com/opentensor/bittensor/tree/master/bittensor/synapses). Synapses are where you would be building and setting up your PyTorch model. We will cover Synapses later in this documentation. Within each example, there is a class `Session()` that is responsible for training, testing, and setting up your model. *This is an optional naming and development convention and is not required for the functionality of Bittensor.*

## Training flow
The general training flow in the Bittensor network can be summarized as follows, all examples follow the same flow:

1. Initialize your model with one you created in [`Synapse`](https://github.com/opentensor/bittensor/tree/master/bittensor/synapses).
    ```Python
    self.model = GPT2LMSynapse( self.config )
    ```

2. Set up your optimizer, scheduler, and dataset. 
    ```Python
    self.optimizer = torch.optim.SGD(self.model.parameters(), lr = self.config.session.learning_rate, momentum=self.config.session.momentum)
    self.scheduler = WarmupCosineWithHardRestartsSchedule(self.optimizer, 50, 300)
    self.dataset = load_dataset('ag_news')['train']
    ```
    Note that `self.config` is generated from the command line arguments or an optional `configuration.yaml`. In this example, we simply load a pre-loaded dataset from the `datasets` library called [ag_news](https://huggingface.co/datasets/ag_news). This is a small dataset that can be run on any machine without eating up the memory with batches.

3. Set up your device where your model will run.
    ```Python
    self.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    ``` 

4. Start a [Neuron](https://github.com/opentensor/bittensor/blob/master/bittensor/neuron.py) session. Neurons allow you to start communicating with nodes in the bittensor network. 
    ```Python
    with self.neuron:
    ```

>Each neuron has an [Axon](https://github.com/opentensor/bittensor/blob/master/bittensor/axon.py) terminal that serves a grpc endpoint which provides access to other synapses on other nodes. Each neuron also has a [Dendrite](https://github.com/opentensor/bittensor/blob/master/bittensor/dendrite.py) which is a differentiable object used to make calls to the network. 

5. Now is when you define your training epoch loop. In the examples, we simply train forever using `while True:` but you can define it to be any number of epochs you wish to use. The first thing we do in this loop is we serve the model on the Axon. This is because after each loop we update our model and so we always want to serve the latest version of our model. 
    ```Python
    self.neuron.axon.serve( self.model )
    ```

6. The fun part starts here, though very straightforward. We can now start training our model. After obtaining the inputs and possibly targets for the current batch, we can just call our model:
    ```Python
        output = self.model.remote_forward(
                self.neuron,
                inputs.to(self.model.device),
                training = True,
            )
    ```
    We call the synapse model's `remote_forward` call as it will run a local forward call and send the present batch of inputs to experts on the network for processing as well. Once the experts have made their own forward call, they will respond with their outputs back to us. The remote expert's output is then used as input into our model. 
   > We now have 3 training losses: 
   > - **local_target_loss**: The loss of our local model with respect to the local target inputs.
   > - **remote_target_loss**: The loss of the remote models.
   > - **distillation_loss**: The loss of the [distilled](https://arxiv.org/abs/1503.02531) version of the model deployed on the >network. This model is the one we can download from the network and use in production. 
  
7. We can sum all our losses now and run a backwards call to accumulate the gradients on the model, then apply them and zero our gradients again. 
   ```Python
        loss = output.local_target_loss + output.distillation_loss + output.remote_target_loss
        loss.backward() # Accumulates gradients on the model.
        self.optimizer.step() # Applies accumulated gradients.
        self.optimizer.zero_grad() # Zeros out gradients for next accummulation
    ```

8. Finally, we have to train our row weights. This is so that we can emit these weights to the network and update our list of experts that we communicate with. 
    ```Python
            batch_weights = torch.mean(output.dendrite.weights, axis = 0)
            self.row = 0.97 * self.row + 0.03 * batch_weights # Moving avg update.
            self.row = F.normalize(self.row, p = 1, dim = 0) # Ensure normalization.
    ```

**And that's it! You can peruse through any of the examples and see how we are applying all these steps to create models that run forever and trade information with other models.**




