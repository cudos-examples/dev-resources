# dev-resources
A list of great resources to get you going with building dApps using Rust/CosmWasm and CosmJS as well as other tools 😊

## Overview of the Cudos stack:
**The Cudos network is a smart-contract-enabled, layer-1 Cosmos blockchain, which facilitates a network of compute infrastructure for running Virtual Machines.**

It is the perfect place to build powerful decentralised applications (dApps) with access both to the consensus-driven underlying blockchain, as well as the cloud infrastructure for the elements of an application that don't require a distributed ledger.

### Your flow for building dApps with Cudos:
1. Get yourself a Cudos wallet address with the help of [this guide](./minidoc/keplr-create.md).
2. Write a smart contract in Rust, compiled to CosmWasm.
3. Deploy your smart contract to the blockchain and initialise it, follow the guide [here](./minidoc/deploy-noded.md).
4. Build a frontend application to interact with the blockchain and your contract on it from a neat UI. These interactions are made using CosmJS to the blockchain, which you can think of simply as a database underlying a traditional application.
5. Deploy your frontend application onto the CudoCompute network in order to host your dApp.

## Great resources for each of:

### **Cudos** (The blockchain and compute platform):

- **The Go-To:** [Cudos Docs](https://docs.cudos.org/docs/build/intro)
- `cudos-blast` [Contract Deployment and Interaction Tool](https://www.npmjs.com/package/cudos-blast)
- [Cudos Sample Contracts and Code](https://github.com/cudos-examples)
- [Cudos Node Repo](https://github.com/CudoVentures/cudos-node)

### **Rust** (The language used for Cudos smart contracts):

- **The Go-To:** [The Rustlang Book](https://doc.rust-lang.org/book/)
- [Let's Get Rusty YouTube Channel](https://www.youtube.com/@letsgetrusty)
- [Intro to Command Line Apps in Rust](https://rust-cli.github.io/book/index.html)

### **CosmWasm** (The smart contract platform that yor Rust contracts are compiled to):

- **The Go-To:** [CosmWasm Documentation](https://docs.cosmwasm.com/docs/1.0/)
- [Area-52 CosmWasm Tutorial](https://area-52.io/)

### **CosmJS** (The frontend library for interacting with Cosmos blockchains from a UI):
- **The Go-To:** [CosmJS Documentation](https://github.com/cosmos/cosmjs)
- `create-cosmos-app` [dApp Frontend Scaffold Tool from Cosmology](https://github.com/cosmology-tech/create-cosmos-app)

---
## Run `create-cosmos-app` on CudoCompute:

### Get set up on CudoCompute:
1. Register an account at [CudoCompute](https://accounts.cudo.org/sign-in?redirect_url=https://compute.cudo.org&_gl=1*1c22xsu*_ga*OTIzMTMzOTA2LjE2NjYxNjM0MzU.*_ga_KFR6C2NZHG*MTY2NjM0MjkzNy4zLjEuMTY2NjM0Mjk0Ni41MS4wLjA.&_ga=2.82121237.2016755034.1666342937-923133906.1666163435&utm_campaign=athenahackathon)
2. Add an SSH key per [the documentation](https://docs.cudocompute.com/web/ssh-keys)
3. Create a project as per [the documentation](https://docs.cudocompute.com/web/projects#create-project)
4. Browse to the project and follow [this video](https://drive.google.com/file/d/1QCPyy8Kte1vfsK0ZB11uoHPWnCgWtPsQ/view?usp=sharing) to create a Virtual Machine.
*Note: select Ubuntu Minimal 20.04*
5. SSH into the machine.
6. (leave feedback on the CudoCompute platform!)

### Install create-cosmos-app on Ubuntu 20.04

Follow the the guide in [this video](https://drive.google.com/file/d/1_1aQh1596sCOnhWnnwRLN96cNIzmr9oi/view?usp=sharing) with the text steps below:

1. Update the package manager (press 1 when prompted to):
```console
apt-get update && apt-get -y upgrade
```
2. `curl` node:
```console
curl -fsSL https://deb.nodesource.com/setup_19.x | sudo -E bash -
```
3. Use `apt-get` to install `nodejs` and `git`:
```console
sudo apt-get install -y nodejs
```
```console
sudo apt-get install -y git
```
4. Use `npm` to install `yarn` and `create-cosmos-app` ([npm package here](https://github.com/cosmology-tech/create-cosmos-app)):
```console
npm install --global yarn
```
```console
npm install -g create-cosmos-app
```
5. Run `create-cosmos-app`:
```console
create-cosmos-app
```
6. Name your app and set multi-chain:

> `? [name] Enter your new app name: my-app`

`Cloning into 'my-app'...`
> `? [template] which template (Use arrow keys)`

> `> connect-multi-chain`

*(you may need to now wait a while)*

7. Change directory into your application folder and run `yarn`:
```console
cd my-app
```
```console
yarn && yarn dev
```
8. Open the IP of your server in a web browser on port 3000, eg (http://34.246.34.92:3000)
9. *(Share any feedback you might have about CudoCompute [here](https://cudoventures.typeform.com/to/FZYRvI2l))*