# FVM
Enables modular complex secure DeFi interactions to be generated without a single line of code. This system is extensively used by the Pie DAO.

# Finance Virtual Machine
Enables modular complex secure DeFi interactions to be generated without a single line of code. This system is extensively used by the Pie DAO.

## Abstract
This paper describes a system which is like what the EVM is for Ethereum but for DeFi. We call this system the Financial Virtual Machine or FVM for short. DeFi users can use this system to construct complex financial interactions without writing a single line of code. We will refer to similar naming conventions used in the EVM in this paper.


## Introduction

### Problem

DeFi is becoming increasingly more fragmented and troublesome to use. Moving for example a USDC lending position from Compound to a DAI fulcrum lending position will take you as many as 5 transactions, with a lot of overhead in gas usage and countless waiting for transactions. With the FVM we can reduce this to just one atomic transaction.

### Insight

DeFi allows users to manage their wealth and create returns in ways previously deemed impossible. A popular way to refer to this ecosystem is "money lego's" because of the composabilty these different protocols bring. However, with composability comes fragmentation. If you want to maximise yields across these platforms you almost need to make it your day to day job and switch between interfaces and do countless numbers of transactions.

### Advantage

We are solving these issues by creating a modular framework with OPCODES that simplify interactions with DeFi protocols without users needing to write a single line of code.

By not holding any user funds outside of a transaction inside the system we greatly reduce the attack vector compared to smart contract wallets. Additionally all smart contract wallets need to code any complex interaction by hand and go through a auditing process for each of them.

## Prior work

The FVM is heavily inspired by other projects in the DeFi ecosystem especially smart contract wallets like: DeFi Saver, Argent, Instadapp, Zerion, Gnoses and Dapper.

Dexlab also made it easier to interface with DeFi protocols by providing a unified user interface for the most popular DeFi protocols.

## Architecture

### Smart contracts

The FVM is composed of a set of extendible smart contracts which each have very specific functionality to follow the separation of concerns principle to make the code easily auditable make adding functionality easy.

The system is composed of the following smart contracts:

### MoneyProxy

Heavily inspired by the DSProxy used in Maker. This contracts main function is:

```solidity
function execute(address _target, bytes memory _data)
    public
    payable
    returns (bytes32 newState)
```

This function takes an address of a Action(target) and a data(_data) argument as parameters and performs a delegateCall to execute the Action code in the context of the money proxy. It returns the new memory state of the transaction.

Only the GateKeeper can call this method to prevent it from selfdestruction. 

### GateKeeper

Since the MoneyProxy executes arbitrary code it needs to be protected against selfdestruction. The only contract that is allowed to call methods on the MoneyProxy is the GateKeeper. Since checking selfdestruction of a contract is not possible due to how the EVM cleans up storage at the end of a transaction the Whitelist is used to allow only execution of whitelisted Actions.

Since all users share the same MoneyProxy and it can execute the ```transferFrom(address _from, address _to, uint256 _amount)``` method on ERC20 tokens the GateKeeper also delivers any needed input tokens.

The GateKeeper has one public method:

```solidity
function sendTokensAndExecute(
    IERC20[] memory _tokens,
    uint256[] memory _amounts,
    address[] memory _targets,
    bytes[] memory _data
) public
```

This method transfers the tokens to the MoneyProxy and executes a batch of actions in the context of the MoneyProxy.

// TODO consider using the registry to fetch contracts so they can be updated by the Pie DAO.
// TODO add documentation about possible fees for this system.


### Actions

Actions were previously referenced to as Partials. After some consideration and inspiration by Gelato finance we decided to rename them to actions.

Actions are stateless by default, the state they do use is passed as a parameter and the output of a action is returned by the action function and passed along to the next one. This concept is often referred to as functional programming and is widely uses in redux which manages state in a similar manner.

If an Action relies on other contracts these are fetched from the Registry at runtime to allow for upgrading of these contracts by the Pie DAO.

Actions can append to the shared state or modify existing entries.

#### Interface


##### action

Each Action contract contains one function with the following signature:

```solidity
funcion action(bytes32[] _state, uint256[] _params) public payable
```

The in memory state is passed as a bytes32 array(_state) and the params(_params) contain index that reference individual parameters.

#### fee

The amount of fee to be charged for a specific action is saved inside the contract. This amount is expressed in PIE tokens.

// TODO consider using a registry to make this fee dynamic


### Registry

Since Actions are stateless and to prevent repetitive data and thus cumbersome updating of smart contract addresses the FVM uses a registry contract to keep track of these addresses.

The registry implements the following methods:

```solidity
function lookup(bytes32 _hashedName) public view returns(address)
```

```solidity
function setAddress(string _contractName) public
```

```solidity
function setAddressByHash(bytes32 _hashedName) external
```

// TODO consider to create a generalized Registry which acts like a global key value store.

### Whitelist

The whitelist keeps track off the Action contracts. We use the generic openzeppelin whitelist contract for this.


### TokenSaver

In case a user makes a mistake and sends tokens or ETH directly to one of the contracts they can be saved by the DAO via the TokenSaver. The TokenSaver is only used in emergency situations

// TODO consider to implement triggers to have a IFTTT like experience, maybe via Gelato?


## Credits
Concept Development & early prototype was published by [Alexintosh](https://twitter.com/Alexintosh) in the [Dexwallet paper](https://github.com/dexlab-io/whitepaper-official#native-integrations-and-recipies).

Author of this document and lead developer of the `FVM` is [MickdeG010](https://twitter.com/MickdeG010).
