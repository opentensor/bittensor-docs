.. Bittensor documentation master file, created by
   sphinx-quickstart on Thu Oct 1 15:20:19 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Bittensor
==========
Welcome to the official documentation of `Bittensor <https://github.com/opentensor/bittensor>`_, the peer-to-peer machine intelligence marketplace. 

BitTensor is an open source peer-to-peer framework that commoditizes machine intelligence. With the rise of machine learning, 
machine intelligence is more and more becoming a commodity. Even moreso than data! In fact, in the last decade computational costs
have gone up 300,000x [1]_, and the number of researchers by 10x [2]_ due to the meteoric rise of production machine intelligence. 

Imagine a massive neural net that is trained on perhaps every image ever uploaded on the internet, or trained on every text available digitally, 
how do we even go about training something this massive? Typically, we would have to train from scratch. However, *is that truly the most efficient 
way to train a model?*

Inevitable Inefficiencies
-------------------------
As AI scientists and engineers, there are a few inevitable truths that we tend to accept: 

1. **Machine intelligence is continuously being produced globally.**
    - However, all the work you’ve done to train a model will only work with that model. If someone else wishes to use your model on a different dataset or perhaps reproduce your results in a different environment, then they will need to redo your work of training the model. 
2. **Any value found by combination of models is lost.** 
    - This is because they’re unable to share their works with each other and so if any one model learns something that may be valuable to other models it cannot share it with them. 
3. **There's a big barrier to entry to ML research.**
    - Today’s machine learning mechanism of “download/create dataset, create model, beat present state-of-the-art, rinse-repeat” makes it so that there is a big barrier to entry to market for new researchers and for researchers with low computational resources. Most researchers cannot contribute to the development of large neural networks because conducting the necessary experiments would be too expensive for them. If we continue to increase the model size by scaling up, eventually the only labs that can conduct competitive research will be those with massive budgets.

Accelerating Hardware and Software
----------------------------------
Typically, increasing the capacity of neural networks can give much better prediction accuracy. This has given rise to hardware and software vertical scaling. 
    - Hardware vertical scaling comes in the form of more compute power, companies like `Graphcore <https://www.graphcore.ai/>`_, `Tenstorrent <https://www.tenstorrent.com/>`_, and of course `Nvidia <https://www.nvidia.com/en-us/>`_, as well as others are presently in an arms race for AI acceleration hardware to enable faster training and inference. 
    - Software vertical scaling comes in the form of deeper neural networks, famous examples include the GPT family of natural language processing developed by `OpenAI <openai.com>`_.

Do we really need a supercomputer?
----------------------------------
What is the world's most powerful supercomputer? It's actually not one specific computer; it's a collection of computers. In fact, if we combined `500 of the fastest supercomputers in the world <https://www.top500.org/lists/top500/2020/11/>`_, the `Bitcoin <www.bitcoin.org>`_ network is still 8x more powerful [3]_.

In the most basic of terms, Bitcoin's utilization of volunteer computing incentivizes users to contribute their compute power to solve hash puzzles in order to verify transactions. 

What if we applied this massive power towards knowledge production?
-------------------------------------------------------------------
The possibilities are endless!

Enter Bittensor
---------------
Bittensor utilizes state-of-the-art techniques such as distributed mixture of experts and knowledge distillation in order to create a peer-to-peer network that incentivizes knowledge production, where producers can sell their work, and consumers can buy this knowledge to make their own models better. This opens the door for collaborative intelligence, where researchers are incentivized to help each other produce more powerful models. Ultimately, this "cognitive economy" has the potential to open a new frontier for the advancement of AI as a whole.

The core idea behind Bittensor is the contribution of knowledge. Users who contribute knowledge to the network are rewarded with <b>tao</b>, Bittensor's currency. Users who wish to use the knowledge on Bittensor to train their own models can also do so. 

Who is Bittensor for?
---------------------
Bittensor is useable by everyone with an interest in cryptocurrencies or AI. 

researchers
~~~~~~~~~~~~
Researchers can use Bittensor to deploy their neural networks onto the peer-to-peer network, and be rewarded for generating knowledge or use the network to further enhance and train their work by taking advantage of the compute power on the network. They can also deploy their published work onto the bittensor network and use it as a benchmark of intelligence, making it easier for other researchers to build on top of their work without inefficiently re-training models.

Companies
~~~~~~~~~
Similarly to researchers, companies can also use Bittensor to train a new model or further train their existing models without needing massive amounts of on-premise hardware. They can also be rewarded for contributing knowledge to the network. In fact, this cognitive economy opens the door to "AI mining" companies, that act as experts or supernodes in the network that contribute knowledge to the network. 

Developers
~~~~~~~~~~
Bittensor is open source, and is open to people who want to research machine learning models or game-theoretic aspects of the cognitive economy. We'd love to have you contribute to Bittensor.


References
-----------
.. [1] “Green AI”, Allen Institute for Artificial Intelligence, Aug 2019. 
.. [2] “AI Index Report”, Stanford University HAL, 2019.
.. [3] "The bitcoin network is now more powerful than the top 500 supercomputers, combined", Christopher Mims, https://qz.com/84056/the-bitcoin-network-is-now-more-powerful-than-the-top-500-supercomputers-combined/, Dec 2020. 

.. rubric:: Bittensor Documentation
.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: Getting Started

   /getting-started/getting_started.md
   /getting-started/hello_world.md
   /getting-started/configuration.md

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: Bittensor

   /bittensor/model_components.md
