---
title: Send Transactions & Deploy Contracts with Web3.py
description: Follow this tutorial to learn how to use the Ethereum Web3 Python Library to deploy Solidity smart contracts to Moonbeam.
---

# Web3.py Python Library

![Intro diagram](/images/builders/build/eth-api/libraries/web3py/web3py-banner.png)

## Introduction {: #introduction } 

[Web3.py](https://web3py.readthedocs.io/) is a set of libraries that allow developers to interact with Ethereum nodes using HTTP, IPC, or WebSocket protocols with Python. Moonbeam has an Ethereum-like API available that is fully compatible with Ethereum-style JSON RPC invocations. Therefore, developers can leverage this compatibility and use the Web3.py library to interact with a Moonbeam python3 as if they were doing so on Ethereum.

In this guide, you'll learn how to use the Web3.py library to send a transaction and deploy a contract on Moonbase Alpha. This guide can be adapted for [Moonbeam](/builders/get-started/networks/moonbeam/){target=_blank}, [Moonriver](/builders/get-started/networks/moonriver/){target=_blank}, or a [Moonbeam development node](/builders/get-started/networks/moonbeam-dev/){target=_blank}.

## Checking Prerequisites {: #checking-prerequisites } 

For the examples in this guide, you will need to have the following:

 - An account with funds.
  --8<-- 'text/faucet/faucet-list-item.md'
 - 
--8<-- 'text/common/endpoint-examples.md'

!!! note
    --8<-- 'text/common/assumes-mac-or-ubuntu-env.md'

## Create a Python Project {: #create-a-python-project }

To get started, you can create a directory to store all of the files you'll be creating throughout this guide:

```
mkdir web3-examples && cd web3-examples
```

For this guide, you'll need to install the Web3.py library and the Solidity compiler. To install both packages, you can run the following command:

```
pip3 install web3 py-solc-x
```

## Setup Web3.py with Moonbeam {: #setup-web3-with-moonbeam } 

You can configure Web3.py to work with any of the Moonbeam networks.
--8<-- 'text/common/endpoint-setup.md'

The simplest way to get started with each of the networks is as follows:

=== "Moonbeam"

    ```python
    from web3 import Web3

    web3 = Web3(Web3.HTTPProvider('{{ networks.moonbeam.rpc_url }}')) # Insert your RPC URL here
    ```

=== "Moonriver"

    ```python
    from web3 import Web3

    web3 = Web3(Web3.HTTPProvider('{{ networks.moonriver.rpc_url }}')) # Insert your RPC URL here
    ```

=== "Moonbase Alpha"

    ```python
    from web3 import Web3

    web3 = Web3(Web3.HTTPProvider('{{ networks.moonbase.rpc_url }}'))
    ```

=== "Moonbeam Dev Node"

    ```python
    from web3 import Web3

    web3 = Web3(Web3.HTTPProvider('{{ networks.development.rpc_url }}'))
    ```

## Send a Transaction {: #send-a-transaction }

During this section, you'll be creating a couple of scripts. The first one will be to check the balances of your accounts before trying to send a transaction. The second script will actually send the transaction. 

You can also use the balance script to check the account balances after the transaction has been sent.

### Check Balances Script {: #check-balances-script }

You'll only need one file to check the balances of both addresses before and after the transaction is sent.  To get started, you can create a `balances.py` file by running:

```
touch balances.py
```

Next, you will create the script for this file and complete the following steps:

1. [Set up the Web3 provider](#setup-web3-with-moonbeam)
2. Define the `address_from` and `address_to` variables
3. Get the balance for the accounts using the `web3.eth.get_balance` function and format the results using the `web3.fromWei`

```python
# 1. Add the Web3 provider logic here:
# {...}

# 2. Create address variables
address_from = "ADDRESS-FROM-HERE"
address_to = "ADDRESS-TO-HERE"

# 3. Fetch balance data
balance_from = web3.fromWei(web3.eth.get_balance(address_from), "ether")
balance_to = web3.fromWei(web3.eth.get_balance(address_to), "ether")

print(f"The balance of { address_from } is: { balance_from } ETH")
print(f"The balance of { address_to } is: { balance_to } ETH")
```

You can view the [complete script on GitHub](https://raw.githubusercontent.com/PureStake/moonbeam-docs/master/.snippets/code/web3py-tx/balances.py){target=_blank}.

To run the script and fetch the account balances, you can run the following command:

```
python3 balances.py
```

If successful, the balances for the origin and receiving address will be displayed in your terminal in ETH.

### Send Transaction Script {: #send-transaction-script }

You'll only need one file for executing a transaction between accounts. For this example, you'll be transferring 1 DEV token from an origin address (from which you hold the private key) to another address. To get started, you can create a `transaction.py` file by running:

```
touch transaction.py
```

Next, you will create the script for this file and complete the following steps:

1. Import the `rpc_gas_price_strategy` which will be used in the following steps to get the gas price used for the transaction
2. [Set up the Web3 provider](#setup-web3-with-moonbeam)
3. Define the `account_from`, including the `private_key`, and the `address_to` variables. The private key is required to sign the transaction. **Note: This is for example purposes only. Never store your private keys in a Python file**
4. Use the [Web3.py Gas Price API](https://web3py.readthedocs.io/en/stable/gas_price.html){target=_blank} to set a gas price strategy. For this example, you'll use the imported `rpc_gas_price_strategy`
5. Create and sign the transaction using the `web3.eth.account.sign_transaction` function. Pass in the `nonce` `gas`, `gasPrice`, `to`, and `value` for the transaction along with the sender's `private_key`. To get the `nonce` you can use the `web3.eth.get_transaction_count` function and pass in the sender's address. To predetermine the `gasPrice` you'll use the `web3.eth.generate_gas_price` function. For the `value`, you can format the amount to send from an easily readable format to Wei using the `web3.toWei` function
6. Using the signed transaction, you can then send it using the `web3.eth.send_raw_transaction` function and wait for the transaction receipt by using the `web3.eth.wait_for_transaction_receipt` function

```python
# 1. Import the gas strategy
from web3.gas_strategies.rpc import rpc_gas_price_strategy

# 2. Add the Web3 provider logic here:
# {...}

# 3. Create address variables
account_from = {
    "private_key": "YOUR-PRIVATE-KEY-HERE",
    "address": "PUBLIC-ADDRESS-OF-PK-HERE",
}
address_to = "ADDRESS-TO-HERE"

print(
    f'Attempting to send transaction from { account_from["address"] } to { address_to }'
)

# 4. Set the gas price strategy
web3.eth.set_gas_price_strategy(rpc_gas_price_strategy)

# 5. Sign tx with PK
tx_create = web3.eth.account.sign_transaction(
    {
        "nonce": web3.eth.get_transaction_count(account_from["address"]),
        "gasPrice": web3.eth.generate_gas_price(),
        "gas": 21000,
        "to": address_to,
        "value": web3.toWei("1", "ether"),
    },
    account_from["private_key"],
)

# 6. Send tx and wait for receipt
tx_hash = web3.eth.send_raw_transaction(tx_create.rawTransaction)
tx_receipt = web3.eth.wait_for_transaction_receipt(tx_hash)

print(f"Transaction successful with hash: { tx_receipt.transactionHash.hex() }")
```

You can view the [complete script on GitHub](https://raw.githubusercontent.com/PureStake/moonbeam-docs/master/.snippets/code/web3py-tx/transaction.py){target=_blank}.

To run the script, you can run the following command in your terminal:

```
python3 transaction.py
```

If the transaction was succesful, in your terminal you'll see the transaction hash has been printed out.

You can also use the `balances.py` script to check that the balances for the origin and receiving accounts have changed. The entire workflow would look like this:

![Send Tx Web3py](/images/builders/build/eth-api/libraries/web3py/web3py-1.png)

## Deploy a Contract {: #deploy-a-contract }

--8<-- 'text/libraries/contract.md'

### Compile Contract Script {: #compile-contract-script } 

In this section, you'll create a script that uses the Solidity compiler to output the bytecode and interface (ABI) for the `Incrementer.sol` contract. To get started, you can create a `compile.py` file by running:

```
touch compile.py
```

Next, you will create the script for this file and complete the following steps:

1. Import the `solcx` package
2. **Optional** - If you haven't already installed the Solidity compiler, you can do so with by using the `solcx.install_solc` function
3. Compile the `Incrementer.sol` function using the `solcx.compile_files` function
4. Export the contract's ABI and bytecode

```python
--8<-- 'code/web3py-contract/compile.py'
```

### Deploy Contract Script {: #deploy-contract-script }

With the script for compiling the `Incrementer.sol` contract in place, you can then use the results to send a signed transaction that deploys it. To do so, you can create a file for the deployment script called `deploy.py`:

```
touch deploy.py
```

Next, you will create the script for this file and complete the following steps:

1. Import the ABI and bytecode
2. [Set up the Web3 provider](#setup-web3-with-moonbeam)
3. Define the `account_from`, including the `private_key`. The private key is required to sign the transaction. **Note: This is for example purposes only. Never store your private keys in a Python file**
4. Create a contract instance using the `web3.eth.contract` function and passing in the ABI and bytecode of the contract
5. Build a constructor transaction using the contract instance and passing in the value to increment by. For this example, you can use `5`. You'll then use the `buildTransaction` function to pass in the transaction information including the `from` address and the `nonce` for the sender. To get the `nonce` you can use the `web3.eth.get_transaction_count` function 
6. Sign the transaction using the `web3.eth.account.sign_transaction` function and pass in the constructor transaction and the `private_key` of the sender
7. Using the signed transaction, you can then send it using the `web3.eth.send_raw_transaction` function and wait for the transaction receipt by using the `web3.eth.wait_for_transaction_receipt` function

```python
# 1. Import the ABI and bytecode
from compile import abi, bytecode

# 2. Add the Web3 provider logic here:
# {...}

# 3. Create address variable
account_from = {
    'private_key': 'YOUR-PRIVATE-KEY-HERE',
    'address': 'PUBLIC-ADDRESS-OF-PK-HERE',
}

print(f'Attempting to deploy from account: { account_from["address"] }')

# 4. Create contract instance
Incrementer = web3.eth.contract(abi=abi, bytecode=bytecode)

# 5. Build constructor tx
construct_txn = Incrementer.constructor(5).buildTransaction(
    {
        'from': account_from['address'],
        'nonce': web3.eth.get_transaction_count(account_from['address']),
    }
)

# 6. Sign tx with PK
tx_create = web3.eth.account.sign_transaction(construct_txn, account_from['private_key'])

# 7. Send tx and wait for receipt
tx_hash = web3.eth.send_raw_transaction(tx_create.rawTransaction)
tx_receipt = web3.eth.wait_for_transaction_receipt(tx_hash)

print(f'Contract deployed at address: { tx_receipt.contractAddress }')
```

You can view the [complete script on GitHub](https://raw.githubusercontent.com/PureStake/moonbeam-docs/master/.snippets/code/web3py-contract/deploy.py){target=_blank}.

To run the script, you can enter the following command into your terminal:

```
python3 deploy.py
```

If successful, the contract's address will be displayed in the terminal. 

![Deploy Contract Web3py](/images/builders/build/eth-api/libraries/web3py/web3py-2.png)

### Read Contract Data (Call Methods) {: #read-contract-data }

Call methods are the type of interaction that don't modify the contract's storage (change variables), meaning no transaction needs to be sent. They simply read various storage variables of the deployed contract.

To get started, you can create a file and name it `get.py`:

```
touch get.py
```

Then you can take the following steps to create the script:

1. Import the ABI
2. [Set up the Web3 provider](#setup-web3-with-moonbeam)
3. Define the `account_from`, including the `private_key`. The private key is required to sign the transaction. **Note: This is for example purposes only. Never store your private keys in a Python file**
4. Create a contract instance using the `web3.eth.contract` function and passing in the ABI and address of the deployed contract
5. Using the contract instance, you can then call the `number` function

```python
# 1. Import the ABI
from compile import abi

# 2. Add the Web3 provider logic here:
# {...}

# 3. Create address variable
contract_address = 'CONTRACT-ADDRESS-HERE'

print(f'Making a call to contract at address: { contract_address }')

# 4. Create contract instance
Incrementer = web3.eth.contract(address=contract_address, abi=abi)

# 5. Call Contract
number = Incrementer.functions.number().call()

print(f'The current number stored is: { number } ')
```

You can view the [complete script on GitHub](https://raw.githubusercontent.com/PureStake/moonbeam-docs/master/.snippets/code/web3py-contract/get.py){target=_blank}.

To run the script, you can enter the following command in your terminal:

```
python3 get.py
```

If successful, the value will be displayed in the terminal.

### Interact with Contract (Send Methods) {: #interact-with-contract }

Send methods are the type of interaction that modify the contract's storage (change variables), meaning a transaction needs to be signed and sent. In this section, you'll create two scripts: one to increment and one to reset the incrementer. To get started, you can create a file for each script and name them `increment.py` and `reset.py`:

```
touch increment.py reset.py
```

Open the `increment.py` file and take the following steps to create the script:

1. Import the ABI
2. [Set up the Web3 provider](#setup-web3-with-moonbeam)
3. Define the `account_from`, including the `private_key`, the `contract_address` of the deployed contract, and the `value` to increment by. The private key is required to sign the transaction. **Note: This is for example purposes only. Never store your private keys in a Python file**
4. Create a contract instance using the `web3.eth.contract` function and passing in the ABI and address of the deployed contract
5. Build the increment transaction using the contract instance and passing in the value to increment by. You'll then use the `buildTransaction` function to pass in the transaction information including the `from` address and the `nonce` for the sender. To get the `nonce` you can use the `web3.eth.get_transaction_count` function 
6. Sign the transaction using the `web3.eth.account.sign_transaction` function and pass in the increment transaction and the `private_key` of the sender
7. Using the signed transaction, you can then send it using the `web3.eth.send_raw_transaction` function and wait for the transaction receipt by using the `web3.eth.wait_for_transaction_receipt` function

```python
# 1. Import the ABI
from compile import abi

# 2. Add the Web3 provider logic here:
# {...}

# 3. Create variables
account_from = {
    'private_key': 'YOUR-PRIVATE-KEY-HERE',
    'address': 'PUBLIC-ADDRESS-OF-PK-HERE',
}
contract_address = 'CONTRACT-ADDRESS-HERE'
value = 3

print(
    f'Calling the increment by { value } function in contract at address: { contract_address }'
)

# 4. Create contract instance
Incrementer = web3.eth.contract(address=contract_address, abi=abi)

# 5. Build increment tx
increment_tx = Incrementer.functions.increment(value).buildTransaction(
    {
        'from': account_from['address'],
        'nonce': web3.eth.get_transaction_count(account_from['address']),
    }
)

# 6. Sign tx with PK
tx_create = web3.eth.account.sign_transaction(increment_tx, account_from['private_key'])

# 7. Send tx and wait for receipt
tx_hash = web3.eth.send_raw_transaction(tx_create.rawTransaction)
tx_receipt = web3.eth.wait_for_transaction_receipt(tx_hash)

print(f'Tx successful with hash: { tx_receipt.transactionHash.hex() }')
```

You can view the [complete script on GitHub](https://raw.githubusercontent.com/PureStake/moonbeam-docs/master/.snippets/code/web3py-contract/increment.py){target=_blank}.

To run the script, you can enter the following command in your terminal:

```
python3 increment.py
```

If successful, the transaction hash will be displayed in the terminal. You can use the `get.py` script alongside the `increment.py` script to make sure that value is changing as expected:

![Increment Contract Web3py](/images/builders/build/eth-api/libraries/web3py/web3py-3.png)

Next you can open the `reset.py` file and take the following steps to create the script:

1. Import the ABI
2. [Set up the Web3 provider](#setup-web3-with-moonbeam)
3. Define the `account_from`, including the `private_key`, and the `contract_address` of the deployed contract. The private key is required to sign the transaction. **Note: This is for example purposes only. Never store your private keys in a Python file**
4. Create a contract instance using the `web3.eth.contract` function and passing in the ABI and address of the deployed contract
5. Build the reset transaction using the contract instance. You'll then use the `buildTransaction` function to pass in the transaction information including the `from` address and the `nonce` for the sender. To get the `nonce` you can use the `web3.eth.get_transaction_count` function 
6. Sign the transaction using the `web3.eth.account.sign_transaction` function and pass in the reset transaction and the `private_key` of the sender
7. Using the signed transaction, you can then send it using the `web3.eth.send_raw_transaction` function and wait for the transaction receipt by using the `web3.eth.wait_for_transaction_receipt` function

```python
# 1. Import the ABI
from compile import abi

# 2. Add the Web3 provider logic here:
# {...}

# 3. Create variables
account_from = {
    'private_key': 'YOUR-PRIVATE-KEY-HERE',
    'address': 'PUBLIC-ADDRESS-OF-PK-HERE',
}
contract_address = 'CONTRACT-ADDRESS-HERE'

print(f'Calling the reset function in contract at address: { contract_address }')

# 4. Create contract instance
Incrementer = web3.eth.contract(address=contract_address, abi=abi)

# 5. Build reset tx
reset_tx = Incrementer.functions.reset().buildTransaction(
    {
        'from': account_from['address'],
        'nonce': web3.eth.get_transaction_count(account_from['address']),
    }
)

# 6. Sign tx with PK
tx_create = web3.eth.account.sign_transaction(reset_tx, account_from['private_key'])

# 7. Send tx and wait for receipt
tx_hash = web3.eth.send_raw_transaction(tx_create.rawTransaction)
tx_receipt = web3.eth.wait_for_transaction_receipt(tx_hash)

print(f'Tx successful with hash: { tx_receipt.transactionHash.hex() }')
```

You can view the [complete script on GitHub](https://raw.githubusercontent.com/PureStake/moonbeam-docs/master/.snippets/code/web3py-contract/reset.py){target=_blank}.

To run the script, you can enter the following command in your terminal:

```
python3 reset.py
```

If successful, the transaction hash will be displayed in the terminal. You can use the `get.py` script alongside the `reset.py` script to make sure that value is changing as expected:

![Reset Contract Web3py](/images/builders/build/eth-api/libraries/web3py/web3py-4.png)

--8<-- 'text/disclaimers/third-party-content.md'