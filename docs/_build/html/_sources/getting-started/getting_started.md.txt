# Installation
Bittensor's installation is fairly straightforward and involves first installing the subtensor module which runs the underlying Blockchain protocol, then Bittensor, which runs your models and the whole system protocol. We will start with the subtensor installation.


>**Note**: Bittensor does not yet support Microsoft Windows support, but you are more than welcome to try and make changes to get it working on Windows and submit a PR for our review. The following guide is for Linux and OS X.

## Subtensor installation

> ### Linux pre-installation instructions
> Prior to running the installation instructions, it's important to install some specific libraries in Linux to get this working. Run the following commands to update your package lists and update those libraries:
>```bash
> sudo apt update && sudo apt install -y cmake pkg-config libssl-dev git build-essential clang libclang-dev curl
>```

1. If you don't have it already, you need to install [Rust](https://www.rust-lang.org/). Open the terminal app on your machine:

    ```bash
    $ curl https://sh.rustup.rs -sSf | sh
    ```
2. Now, source your cargo environment:
    ```bash
    $ source ~/.cargo/env
    ```
3. Now add the WASM32 nightly toolchain:
    ```bash
    $ rustup target add wasm32-unknown-unknown --toolchain nightly-2020-10-06
    ```
4. Set the toolchain target:
    ```bash
    $ rustup target add wasm32-unknown-unknown --toolchain nightly-2020-10-06
    ```
5. Now that you have the toolchain set up, clone the subtensor blockchain repository:
    ```bash
    $ git clone https://github.com/opentensor/bittensor.git 
    ```

6. You can either build the blockchain binary yourself, or you can run the binary directly. To run the binary directly, simply run the subtensor bin.
    ```
    $ ./bin/release/node-subtensor --chain akira
    ```
 > **NOTE**: **'Akira'** is the test network of Bittensor -- You can think of it as the staging environment. You can also run `./bin/release/node-subtensor --dev` to run the chain only locally. To run the chain in the main network (only recommended when you are ready) then you need to use the **'Kusanagi'** chain. 
 
 7. *(Optional)* If you wish to build the chain binary yourself, run:
    ```
    $ WASM_BUILD_TOOLCHAIN=nightly-2020-10-06 cargo build --release
    ```

That's it! You've set up the subtensor chain to run on your computer. Let's move on to setting up BitTensor now.


## Bittensor installation
1. Clone the bittensor repository:
    ```
    $ git clone https://github.com/opentensor/bittensor.git  
    ```
2. Install bittensor:
    ```
    $ cd bittensor && pip install -r requirements && pip install -e .
    ```
3. Create a new wallet in order to stake/unstake your tokens:
    ```
    $ bittensor-cli new_wallet 
    ```
    and follow the instructions. Make sure to generate a **secure password** when prompted. We recommend
    a good password manager.