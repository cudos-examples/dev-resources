# How to Deploy a Smart Contract on Cudos using `cudos-noded` - Fully Explained!
>**NOTE:** As always, if you see angle brackets surrounding text, eg: `<your-wallet-name>`, you need to fill in your own variable name.

1. **Install the Cudos node**

    The Cudos node daemon, `cudos-noded`, can be installed from here: [`cudos-noded`](https://github.com/CudoVentures/cudos-node/).
    > This will install the blockchain node daemon binary which will allow you to either run your own node locally or on the chain, or interact with other nodes via RPC using the same CLI.

2. **Compile your rust source files down to `.wasm`**

    You will need to have compiled your smart contract to the `.wasm` artifacts. This can be done locally for **testing** purposes only.
    >For **production**, you will need to compile your contract within the [`rust-optimizer`](https://github.com/CosmWasm/rust-optimizer) Docker container for reproducible and verifiable builds. This should only be done using Intel x86 processors. ARM processors cannot generate reproducible builds.

    Need some source file examples? You can start out by compiling all the CW contract files. These are the standard contracts for tokens in the Cosmos ecosystem and you can find them at [the `cw-plus` repo](https://github.com/CosmWasm/cw-plus). Since `cw-plus` is a monorepo, use the `workspace-optimizer` within the `rust-optimzer` mentioned above.

3. **Set environment variables**
    
    You will notice we use *a lot* of environment variables for all the flags of the CLI to keep things structured and avoid repetition. We use `_TN` at the end of any flags to denote "testnet". Here is an example list of flags:
    ```
    # environment variables for testnet, ending with "TN"
    export RPC_NODE_TN="https://rpc.testnet.cudos.org:443"
    export CHAIN_ID_TN=cudos-testnet-public-3
    export GAS_TN=auto
    export GAS_PRICES_TN=5000000000000acudos
    export GAS_ADJUSTMENT_TN=1.3
    export KEYRING_TN=os

    # environment variables for mainnet
    export RPC_NODE="https://rpc.cudos.org:443"
    export CHAIN_ID=cudos-1
    export GAS=auto
    export GAS_PRICES=5000000000000acudos
    export GAS_ADJUSTMENT=1.3
    export KEYRING=os

    # the TX_FLAGS variables combines a number of the above testnet variables
    export TX_FLAGS_TN="--node=$RPC_NODE_TN --gas=$GAS_TN --gas-adjustment=$GAS_ADJUSTMENT_TN --gas-prices=$GAS_PRICES_TN --chain-id=$CHAIN_ID_TN --keyring-backend=$KEYRING_TN"

    export TX_FLAGS="--node=$RPC_NODE --gas=$GAS --gas-adjustment=$GAS_ADJUSTMENT --gas-prices=$GAS_PRICES --chain-id=$CHAIN_ID --keyring-backend=$KEYRING"
    ```
    >It can be helpful to store this in a `vars.sh` file and then to run:
    ```console
    source vars.sh
    ```


    >For up to date flags for Cudos (and other Cosmos blockchains), you can refer to the [Cosmos Chain Registry](https://github.com/cosmos/chain-registry). Cudos testnet is [here](https://github.com/cosmos/chain-registry/tree/master/testnets/cudostestnet), and mainnet is [here](https://github.com/cosmos/chain-registry/tree/master/cudos). *(It's often good practice when building frontends to pull the chain info automatically from the `chain-registry` repo so that your configuration stays up-to-date).*


4. **Create a wallet address and add it to the CLI instance.**

    Create a Cudos wallet address if you haven't already by following the instructions [here](keplr-create.md), then add your mnemonic to your `cudos-noded` instance to make transactions on the chain with your private key *(remove the `_TN` from the end of any flags if you are working with the mainnet)*:
    ```console
    cudos-noded keys add <your-wallet-name> --keyring-backend $KEYRING_TN --recover
    ```
    This will ask you to type your BIP-39 mnemonic phrase which will be secured using your operating system keyring which you set with the flag.

    We now add this wallet to an environment variable as well, we call it `$OWNER_TN` for testnet:
    ```console
    OWNER_TN=$( cudos-noded keys show -a <your-wallet-name> --keyring-backend "$KEYRING_TN" | tee /dev/tty | tail -1 | tr -d '\r' )
    ```

    >:warning: As always, keep your mnemonic phrase safe and secret.

    > Your wallet will also need funds to pay for gas fees, you can add testnet funds via the faucet icon on the bottom left of [the testnet Cudos Dashboard](http://dashboard.testnet.cudos.org/). Mainnet funds you will need to purchase on an exchange and either send to your address or [bridge](https://bridge.cudos.org/) from the Ethereum ERC-20 version of the token once in your Ethereum wallet. Feel free to message in the [Cudos Discord](https://discord.gg/cudos/) if you need help, we may even send you a few cents worth of CUDOS to get you on your way with your first transaction on mainnet.


5. **Deploy your compiled `.wasm` contract from before to the Cudos blockchain.**

     To do this we use the `cudos-noded` CLI to run the `tx wasm store` command which uploads the contract file to the chain. We set the output of that command to the `$RESULT` environment variable:
    ```console
    STORE_RESULT=$(cudos-noded tx wasm store ./<path-to>/artifacts/<name-of-wasm-file.wasm> --from $OWNER_TN `echo $TX_FLAGS_TN`)
    ```

6. **Get the index of the contract file from the chain.**
    >Cosmos blockchain store wasm contracts in an array, we need to find the index of your contract file in the array which is returned as part of the `$STORE_RESULT` of the previous command.
    
    While you can sift through the output of the previous command, it's easier to get the index of your smart contract on the chain, here we use JQuery (`jq`) to extract it from the JSON:
    ```console
    CONTRACT_INDEX=$( echo $STORE_RESULT | jq -r '.logs[0].events[-1].attributes[-1].value' | tee /dev/tty )
    ```

7. **Instantiate your contract**

    This initialises the contract with its starting state. Your instantiation will require a JSON payload based off the fields in your `InstantiateMsg` struct within `msg.rs` of your contract source code. We set an environment variable for this too.

    Some contracts don't need a payload if the `InstantiateMsg` struct is empty:
    ```console
    INST="{}"
    ```
    Here is an example of a payload for a standard CW20 token instantiation, we use `jq` again to format the JSON payload, note the use of the `--arg` to pass an `$address` argument based on your `$OWNER_TN` environment variable you set earlier:
    ```console
    INST=$( jq -n --arg address $OWNER_TN '{ "name": "icecream", "symbol": "icream", "decimals": 6, "initial_balances": [ { "address": $address, "amount": "1000000" } ], "mint": { "minter": $address, "cap": "99900000000" } }' | tee /dev/tty )
    ```

    Before we instantiate our contract let's set two more variables, first we need a label name for the contract to give a human readable label:
    ```console
    $CONTRACT_LABEL="<label-name-for-contract>"
    ```
    Now we set an admin variable. For upgradeable smart contracts, we need to specify the address of the admin of the contract. While you can set the admin address to any address you like, in this case we use the owner address we used to store the contract:
    ```console
    ADMIN="--admin $OWNER_TN"
    ```
    Or, you can set this as `no-admin` if you want the contract to remain immutable:
    ```console
    ADMIN="--no-admin"
    ```

    :rocket: Now we instantiate! This calls the `instantiate` method on the stored contract and passes in the JSON from the `$INST` variable we set above. This gets stored in it's own `$INST_RESPONSE` variable:
    ```console
    INST_RESPONSE=$(cudos-noded tx wasm instantiate $CONTRACT_INDEX "$INST" --from $OWNER_TN --label $CONTRACT_LABEL `echo $TX_FLAGS_TN` $ADMIN)
    ```

8. **Get the contract address.**
    
    Now that our contract is instantiated, it has its own address on the network which was returned in the output of the instantiation, so we extract it with `jq`, and set it as yet another environment variable:
    ```console
    CONTRACT_ADDRESS=$(echo $INST_RESPONSE | jq -r '.logs[0].events[0].attributes[0].value' | tee /dev/tty | tail -1 | tr -d '\r')
    ```
    **You have successfully deployed and instantiated a smart contract on Cudos! Congratulations!**
    >Having deployed and retrieved the contract address, you can now interact with the contract on the chain, you can do this with a frontend to build your project into a full decentralised application (dApp). Take a look at `create-cosmos-app` to get going, it's a simple way of scaffolding a React frontend within the Cosmos ecosystem - Cudos included. [This YouTube video](https://www.youtube.com/live/hPec5D_lI1A?feature=share&t=1880) helps use `create-cosmos-app` for Cudos, as well as the docs in [the README of this very repo](../README.md).

9. **BONUS: Execute functions on the contract from the CLI**

    While most end users will interact with the smart contract via a dApp frontend, you can also interact and execute functions on the contact with the `cudos-noded` CLI. Here's an example using the CW20 standard token contract again:

    We start by also creating our JSON payload, this is structured to map to the parameters of the type defined within `ExecuteMsg` found via the `contract.rs` file. In the case of the CW20 standard token contract, we can see [here](https://github.com/CosmWasm/cw-plus/blob/4da476f9e426fb87689b6f0c3398ff08a65248d1/contracts/cw20-base/src/contract.rs#L195) that the parameters for a `Transfer` are the `recipient` and the `amount`.
    > `Transfer` is a base message to move tokens to another account without triggering actions. It is similar but different to `TransferFrom` and `Send`. You can see more in the `msg.rs` file for the standard CW20 token [here](https://github.com/CosmWasm/cw-plus/blob/main/packages/cw20/src/msg.rs).

    For our JSON payload, we need an address to send to so we set an environment variable for `$OLLIE`:
    ```console
    OLLIE="cudos1d7jw3ply86e4kcudmlv78ask32kdp7r2usf8xn"
    ```
    Now we structure our payload, again using `jq`:
    ```console
    TRANSFER_TO_OLLIE=$( jq -n --arg recipient $OLLIE '{ "transfer": { "recipient": $recipient, "amount": "1111" } }' | tee /dev/tty )
    ```
    Then we execute this on chain:
    ```console
    cudos-noded tx wasm execute $CONTRACT_ADDRESS "$TRANSFER_TO_OLLIE" --from $OWNER_TN `echo $TX_FLAGS_TN`
    ```

10. **BONUS 2: Query the contract state from the CLI**

    > Similarly to Ethereum, queries on Cosmos chains don't cost gas fees.

    We structure the JSON payload for a query similar to above:
    ```console
    BALANCE_OF_OLLIE=$( jq -n --arg address $OLLIE '{ "balance": { "address": $address } }' | tee /dev/tty )
    ```
    And we run it as a query from the CLI:
    ```console
    cudos-noded query wasm contract-state smart $CONTRACT_ADDRESS "$BALANCE_OF_OLLIE" --node $RPC_NODE_TN
    ```
    Where we should get back the balance of the `$OLLIE` address.

## Congratulations on getting this far! You've done a lot!
Feel free to reach out on [Discord](https://discord.gg/cudos) if you have any questions, click the :computer: icon in the `#role-assignment` channel there to get access to the dev channels.

Now your next step, build that dApp frontend to interact with your contract from a great UI! Check the README for info on `create-cosmos-app` for more information!

Good luck fren!


