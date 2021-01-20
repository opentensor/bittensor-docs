# Configuration
Bittensor's documentation can be set up either through command line arguments or through a YAML file. To save trouble and avoid writing redundant descriptions, we will leave you to look up what each command line argument means. 

Alternatively, you can also get a full list of command line arguments by running the `--help` flag on any of the examples. For example,
```bash
python3 examples/mnist.py --help
```

You can also set up your own YAML configuration of Bittensor if you need more extensive runs of Bittensor. Simply create a YAML file and pass it to your training script, for example:
```bash
python3 examples/mnist.py --neuron.config_file mnist_config.yaml
```
where `mnist_config.yaml` is a custom config that you've defined. 

> **NOTE**: Your custom config can simply include configs that you care about, it does not need to include every single config for bittensor. Missing configs from the YAML file will simply be defaulted.

An example config file can be as follows for `gpt2-wiki.py`:

```yaml
# Axon settings
axon:
   max_gradients: 100
   max_workers: 10
   port: 8091

# Dendrite settings
 dendrite:
   do_backoff: true
   key_dim: 100
   max_backoff: 100
   pass_gradients: true
   stale_emit_filter: 10000
   timeout: 0.5
   topk: 10

# Metagraph settings (only thing you need to change here is your endpoint)
 metagraph:
   chain_endpoint: localhost:9944 # Production network
   stale_emit_filter: 10000

 # Your neuron settings
 neuron:
   coldkeyfile: ~/your/path/to/your/wallet/coldkeypub.txt #~/.bittensor/wallets/kusanagi_adam/coldkeypub.txt
   hotkeyfile: ~/your/path/to/your/wallet/hotkeys/default

# your settings for your training script, where session can be replaced with whatever cmd line arguments you used, see examples/*.py for examples
 session:
   accumulation_interval: 5
   apply_remote_gradients: true
   batch_size_train: 8
   datapath: data
   learning_rate: 0.001
   log_interval: 10
   momentum: 0.98
   name: Adam
   sync_interval: 100
   trial_id: '1608300539'
   trial_path: data/adam/1608300539
   record_log: true

# your Synapse settings, see bittensor/synapses/*.py for examples on synapse command line arguments. These are
# typically just model settings like hidden layers, attention heads, etc. 
 synapse:
   activation_function: gelu_new
   attn_pdrop: 0.1
   embd_pdrop: 0.1
   initializer_range: 0.02
   layer_norm_epsilon: 1.0e-05
   n_head: 16
   n_inner: null
   n_layer: 36
   resid_pdrop: 0.1
   summary_activation: null
   summary_first_dropout: 0.1
   summary_proj_to_labels: true
   summary_type: cls_index
   summary_use_proj: true
   ```
 