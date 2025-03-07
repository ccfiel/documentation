# Deploying Smart Contracts

The `agoric deploy` command in the [agoric command line tool](/guides/agoric-cli/commands.md#agoric-deploy) 
supports deploying contracts and off-chain web applications that talk to them. The command 
has two primary uses:

* Deploy smart contract source code onto the blockchain
* Deploy and setup an application program to a local server running an Agoric process

Use the `agoric deploy` command to run your Dapp's `contract/deploy.js` 
and `api/deploy.js` scripts. You can use the deploy scripts created when you copied an existing 
Dapp into your directory as they are, or you can modify the scripts as suggested later in this document.

Remember, your Dapp has three primary subdirectories:
- `contract/`which contains files relating to your smart contract itself.
- `api/`which contains files enabling the UI frontend to communicate via HTTP/WebSocket to an on-chain backend contract instance and start your Dapp contract instance and backend.
- `ui/` which contains files relating to your contract's user interface.

## How it works

All deployment happens via the local running Agoric process. This is usually the `ag-solo` process, 
and frequently referred to as that or just as `ag-solo`. It is also sometimes described as/called an Agoric VM or a local server.

`ag-solo` communicates with either a locally running or remote chain. The local process has a `home` object, which contains 
references to services on-chain, including `zoe`, the `board` for
sharing objects, and an application user's `wallet`. Developers can
use these service references to call the service's associated API commands.

Deploying to the chain first uploads the bundled contract source code to the local Agoric process (`ag-solo`).
The deployment script then uses the `home` object to access `zoe` which installs the code on chain. 

Via the REPL associated with the wallet, developers can use all the on-chain commands that deployment scripts use to deploy 
contracts and Dapps.

Each `deploy.js` runs in its own temporary process, connected to `ag-solo`, through which it can reach the chain.

## Contract deployment

First, let's look at *contract deployment*. `contract/deploy.js` bundles up a contract's source code
(which may consist of multiple files and modules) and "installs" 
it on the blockchain as source code, using [`Zoe`](/getting-started/intro-zoe.md). 
This does _not_ execute contract code; it just makes the code available on-chain.

The contract deployment process uses [`zoe.install()`](/zoe/api/zoe.md#e-zoe-install-bundle) 
to install the contract source code on-chain. This returns an *installation* associated with the 
source code. In a typical contract deployment, the deploy script adds the installation 
to the default shared board so it is broadly accessible on the chain. The script then writes
the board id to a config file in the Dapp's `ui` directory as shown below.

By default, when you run `agoric init`, your Dapp gets 
the [`dapp-fungible-faucet` `contract/deploy.js` file](https://github.com/Agoric/dapp-fungible-faucet/blob/main/contract/deploy.js), 
which is our example of a typical contract deploy script.

Deploying the `dapp-fungible-faucet` contract (e.g., with `agoric deploy contract/deploy.js` after `agoric init` 
copied it into a local directory) installs it on chain, and generates the 
file `./ui/public/conf/installationConstants.js`with contents like:
```js
// GENERATED FROM dapp-fungible-faucet/contract/deploy.js
export default {
    "CONTRACT_NAME": "fungibleFaucet",
    "INSTALLATION_BOARD_ID": "1456154132"
};
```
The board provides a unique ID per object, in this case
`"1456154132"`, so it is different for each contract deployment.

## Application service deployment and setup

Next, let's look at *application deployment and setup*. As compared to contract deployment, 
you need to customize the Agoric API server deployment and setup much more
for your particular application and contract. Some Dapps use a singleton contract instance 
which presumes that it will be installed once and serve all customers (as opposed to an auction
or swap contract, which is installed once, but instantiated separately for each sale it manages).
A singleton potentially must:
- Be created 
- Find and connect to on-chain resources such as issuers for specific currencies
- Create new on-chain resources like new currencies or NFTs
- Create new Purses for the application to use
- Seed the on-chain contract with initial orders or configuration.

These example contract `api/deploy.js` scripts show some of the 
range of the above custom setup actions:
* [`dapp-fungible-faucet`](https://github.com/Agoric/dapp-fungible-faucet/blob/main/api/deploy.js)
* [`dapp-card-store`](https://github.com/Agoric/dapp-card-store/blob/main/api/deploy.js)
* [`dapp-simple-exchange`](https://github.com/Agoric/dapp-simple-exchange/blob/main/api/deploy.js)

Application deployment steps may include:
* Bundle the `api` code and deploy it to the running local "api" process (ag-solo)
* Include the contract installation configuration information in the bundle
* Create new currencies and add them to the application's wallet

Steps for contracts that use a singleton instance for all clients may further include:
* Instantiate a contract instance using the installation created when the contract deployed
* Use the invitation from that instance creation to configure the new instance
* Register the contract instance's `instance` with the Board
* Record the contract instance's Board ID in a configuration file

