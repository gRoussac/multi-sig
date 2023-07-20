# Multi-signature Key Management on a Casper Network

>**DO NOT RUN THESE EXAMPLES ON MAINNET!** 
>
> Incorrect account configurations could render accounts defunct and unusable, thus losing access to all the account's CSPR.
>
> The session code provided in this repository SHOULD NOT be used in a production environment.
> Test all account changes in a test environment like the Testnet.

## Introduction

The purpose of this repository is to provide an example of how to integrate key management on Casper accounts. The repository contains session code that can be used for learning how to configure Casper accounts using associated keys and multi-signature deploys.

## Step 1: Clone the example Wasm for this workflow

```bash
git clone https://github.com/cryofracture/multi-sig && cd multi-sig
```

## Step 2: Build and test the sample Wasm provided

```bash
rustup update
make clean
make prepare
make test
```

## Step 3: Increase the primary key's weight to set thresholds

Increase the weight of the primary key to ensure the primary account can meet the `key_management` threshold and make any account updates.

Retrieve the account hash of the primary key you are working with using a block explorer or the Casper CLI client.

```bash
casper-client account-address --public-key <INSERT_PUBLIC_KEY_HEX>
```

Update the weight of the primary key to 3 by calling the `update_associated_keys.wasm`.

### FOR EXAMPLE ONLY, PLEASE UPDATE PRIOR TO EXECUTING

```bash
casper-client put-deploy --node-address https://rpc.testnet.casperlabs.io/ \
--chain-name "casper-test" \
--payment-amount 500000000 \
--secret-key $PATH/secret_key.pem \
--session-path target/wasm32-unknown-unknown/release/update_associated_keys.wasm \
--session-arg "associated_key:key='account-hash-<ACCOUNT_HASH_HEX_HERE>'" \
--session-arg "new_weight:u8='3'"
```

The primary key in this account should now have weight 3.

<details>
<summary>Account details</summary>

```json
"Account": {
      "account_hash": "account-hash-d89c49f7e03f418dc285e94e254d53574db878665271f366bb3aeddded7ab757",
      "action_thresholds": {
        "deployment": 1,
        "key_management": 1
      },
      "associated_keys": [
        {
          "account_hash": "account-hash-d89c49f7e03f418dc285e94e254d53574db878665271f366bb3aeddded7ab757",
          "weight": 3
        }
      ],
      "main_purse": "uref-b4532f30031b9deb8b2879a91ac185577dcba763de9d48753385e0ef41235dfa-007",
      "named_keys": []
    }
  }
```

</details>

## Step 4: Update the account's action thresholds

Set up a multi-signature scheme for the account, by updating the `deployment` and `key_management` thresholds. The `update_thresholds.wasm` included will set `deployment` threshold to 2 and the `key_management` threshold to 3.

### FOR EXAMPLE ONLY, PLEASE UPDATE PRIOR TO EXECUTING


```bash
casper-client put-deploy \
--node-address https://rpc.testnet.casperlabs.io \
--chain-name casper-test \
--payment-amount 500000000 \
--secret-key $PATH/secret_key.pem \
--session-path target/wasm32-unknown-unknown/release/update_thresholds.wasm \
--session-arg "deployment_threshold:u8='2'" \
--session-arg "key_management_threshold:u8='3'"
```

The account's action thresholds would look like this:

```json
"action_thresholds": {
  "deployment": 2,
  "key_management": 3
},
```


## Step 5: Add associated keys to the primary account

To add an associated key to the primary account, use the `add_account.wasm` provided. The example below starts by adding the following account as an associated key: `account-hash-e2d00525cac31ae2756fb155f289d276c6945b6914923fe275de0cb127bffee7`.

### FOR EXAMPLE ONLY, PLEASE UPDATE PRIOR TO EXECUTING

```bash
casper-client put-deploy --node-address https://rpc.testnet.casperlabs.io/ \
--chain-name "casper-test" \
--payment-amount 500000000 \
--secret-key $PATH/secret_key.pem \
--session-path target/wasm32-unknown-unknown/release/add_account.wasm \
--session-arg "new_key:key='account-hash-e2d00525cac31ae2756fb155f289d276c6945b6914923fe275de0cb127bffee7" \
--session-arg "weight:u8='1'"
```

Next, add a second and third account as associated keys with weight 1.

```bash
casper-client put-deploy --node-address https://rpc.testnet.casperlabs.io/ \
--chain-name "casper-test" \
--payment-amount 500000000 \
--secret-key $PATH/secret_key.pem \
--session-path target/wasm32-unknown-unknown/release/add_account.wasm \
--session-arg "new_key:key='account-hash-04a9691a9f8f05a0f08bd686f188b27c7dbcd644b415759fd3ca043d916ea02f" \
--session-arg "weight:u8='1'"

casper-client put-deploy --node-address https://rpc.testnet.casperlabs.io/ \
--chain-name "casper-test" \
--payment-amount 500000000 \
--secret-key $PATH/secret_key.pem \
--session-path target/wasm32-unknown-unknown/release/add_account.wasm \
--session-arg "new_key:key='account-hash-1fed34baa6807a7868bb18f91b161d99ebf21763810fe4c92e39775d10bbf1f8" \
--session-arg "weight:u8='1'"
```

The account would now have one primary key with weight 3, and three associated accounts, each with weight 1.

<details>
<summary>Account details</summary>

```json
"Account": {
      "account_hash": "account-hash-d89c49f7e03f418dc285e94e254d53574db878665271f366bb3aeddded7ab757",
      "action_thresholds": {
        "deployment": 2,
        "key_management": 3
      },
      "associated_keys": [
        {
          "account_hash": "account-hash-04a9691a9f8f05a0f08bd686f188b27c7dbcd644b415759fd3ca043d916ea02f",
          "weight": 1
        },
        {
          "account_hash": "account-hash-1fed34baa6807a7868bb18f91b161d99ebf21763810fe4c92e39775d10bbf1f8",
          "weight": 1
        },
        {
          "account_hash": "account-hash-d89c49f7e03f418dc285e94e254d53574db878665271f366bb3aeddded7ab757",
          "weight": 3
        },
        {
          "account_hash": "account-hash-e2d00525cac31ae2756fb155f289d276c6945b6914923fe275de0cb127bffee7",
          "weight": 1
        }
      ],
      "main_purse": "uref-b4532f30031b9deb8b2879a91ac185577dcba763de9d48753385e0ef41235dfa-007",
      "named_keys": []
    }
```

</details>

### NOTES

1. All associated keys should be kept incredibly secure to ensure the security and integrity of the account.
2. After all associated keys and action thresholds have been set to the desired multi-signature scheme, the weight of the original primary key can be increased or lowered, depending on your use case. Be careful with this! If you lower the primary key's weight below the key management threshold, the account will require multiple signatures for key management. The account will be unusable if you do not have enough associated keys set up.

## Step 6: Send a multi-signature deploy from the primary account

After setting up the account with a multi-signature scheme, use the following commands to sign a deploy with multiple keys and send it to the network:

1. `make-deploy` - creates and signs a deploy, saving the output to a file
2. `sign-deploy` - adds additional signatures for a multi-signature deploy
3. `send-deploy` - sends the deploy to the network

The following example sends a multi-sig deploy containing Wasm (`hello_world.wasm`) that adds a named key to the account. The deploy originates from the primary account and needs two signatures to meet the `deployment` weight set to 2. Once both keys sign the deploy, either can send it to the network. The Wasm used here can be found in the [hello-world](https://github.com/casper-ecosystem/hello-world) repository.

### FOR EXAMPLE ONLY, PLEASE UPDATE PRIOR TO EXECUTING

The first associated key creates and signs the deploy with the `make-deploy` command.

```bash
casper-client make-deploy --chain-name casper-test \
--payment-amount 3000000000 \
--session-path tests/wasm/contract.wasm \
--secret-key $PATH/secret_key.pem \
--session-arg "my-key-name:string='user_1_key'" \
--session-arg "message:string='Hello, World'" \
--output hello_world_one_signature
```

The second associated key signs the deploy with `sign-deploy` to meet the deployment threshold for the account.

```bash
casper-client sign-deploy -i hello_world_one_signature -k ~/cspr_nctl/user-2.pem -o hello_world_two_signatures

```

Now the deploy can be sent to the network with the `send-deploy` command:

```bash
casper-client send-deploy --node-address https://rpc.testnet.casperlabs.io -i hello_world_two_signatures
```

The `hello_world.wasm` will run and add a named key to the account.

```json
"named_keys": [
        {
          "key": "uref-9b9ecaa9e5e235fc6955d4d528cb1b5b38f2d800f6cbbc55351131a3701b5a81-007",
          "name": "my-key-name"
        }
      ]
```

## Step 7: Send a multi-signature deploy from an associated key

To initiate a multi-sig deploy from an associated account instead of the primary account, use the `--session-account` argument, which requires the hex-encoded public key of the account context under which the session code will be executed. The process is very similar to the previous step.

### FOR EXAMPLE ONLY, PLEASE UPDATE PRIOR TO EXECUTING

One associated key creates and signs the deploy with the `make-deploy` command, indicating the account context under which the session code will be executed.

```bash
casper-client make-deploy --chain-name casper-test \
--payment-amount 3000000000 \
--session-path tests/wasm/contract.wasm \
--secret-key $PATH/secret_key.pem \
--session-arg "my-key-name:string='user_1_key'" \
--session-arg "message:string='Hello, World'" \
--session-account 04a9691a9f8f05a0f08bd686f188b27c7dbcd644b415759fd3ca043d916ea02f \
--output hello_world_one_signature
```

The second associated key signs the deploy with `sign-deploy` to meet the deployment threshold for the account.

```bash
casper-client sign-deploy -i hello_world_one_signature -k ~/cspr_nctl/user-2.pem -o hello_world_two_signatures

```

The deploy can be sent to the network using the `send-deploy` command:

```bash
casper-client send-deploy --node-address https://rpc.testnet.casperlabs.io -i hello_world_two_signatures
```

The `hello_world.wasm` will run and add a named key to the account.


## Step 8: Remove a key from the account

This example adds a fourth associated key with `account-hash-77ea2e433c94c9cb8303942335da458672249d38c1fa5d1d7a7500b862ff52a4`, and then removes it using the `remove_account.wasm` session code. The goal is to show how to remove a key that may have been compromised.

>**Caution**: If you remove one of the existing keys without adding the fourth associated key, the account will become unusable since none of the remaining keys will meet the required weight for key management. Changing weights or adding new associated keys would become impossible.

### FOR EXAMPLE ONLY, PLEASE UPDATE PRIOR TO EXECUTING

Given the current setup, three associated keys need to sign the deploy to add a fourth associated key. One associated key creates and signs the deploy with the `make-deploy` command.

```bash
casper-client make-deploy --chain-name casper-test \
--payment-amount 500000000 \
--secret-key $PATH/secret_key.pem \
--session-path target/wasm32-unknown-unknown/release/add_account.wasm \
--session-arg "new_key:key='account-hash-77ea2e433c94c9cb8303942335da458672249d38c1fa5d1d7a7500b862ff52a4" \
--session-arg "weight:u8='1'" \
--output add_account_one_signature
```

The second and third associated keys sign the deploy with `sign-deploy` to meet the key management threshold for the account.

```bash
casper-client sign-deploy -i add_account_one_signature -k ~/cspr_nctl/user-2.pem -o add_account_two_signatures
casper-client sign-deploy -i add_account_two_signatures -k ~/cspr_nctl/user-3.pem -o add_account_three_signatures
```

Send the deploy containing the `add_account.wasm` to add a new associated account with weight 1.

```bash
casper-client send-deploy --node-address https://rpc.testnet.casperlabs.io -i add_account_three_signatures
```

The account should now have four associated keys with weight 1 and the primary key (from which all deploys originate) with weight 3.

<details>
<summary>Account details</summary>

```json
"Account": {
      "account_hash": "account-hash-d89c49f7e03f418dc285e94e254d53574db878665271f366bb3aeddded7ab757",
      "action_thresholds": {
        "deployment": 2,
        "key_management": 3
      },
      "associated_keys": [
        {
          "account_hash": "account-hash-04a9691a9f8f05a0f08bd686f188b27c7dbcd644b415759fd3ca043d916ea02f",
          "weight": 1
        },
        {
          "account_hash": "account-hash-1fed34baa6807a7868bb18f91b161d99ebf21763810fe4c92e39775d10bbf1f8",
          "weight": 1
        },
        {
          "account_hash": "account-hash-77ea2e433c94c9cb8303942335da458672249d38c1fa5d1d7a7500b862ff52a4",
          "weight": 1
        },
        {
          "account_hash": "account-hash-d89c49f7e03f418dc285e94e254d53574db878665271f366bb3aeddded7ab757",
          "weight": 3
        },
        {
          "account_hash": "account-hash-e2d00525cac31ae2756fb155f289d276c6945b6914923fe275de0cb127bffee7",
          "weight": 1
        }
      ],
      "main_purse": "uref-b4532f30031b9deb8b2879a91ac185577dcba763de9d48753385e0ef41235dfa-007",
      "named_keys": []
    }
```

</details>

The `remove_account.wasm` will remove the newly added account to demonstrate the possibility of removing associated keys that may have been compromised. This deploy needs to be signed by three associated keys to meet the key management threshold.

One associated key creates and signs the deploy with the `make-deploy` command.

### FOR EXAMPLE ONLY, PLEASE UPDATE PRIOR TO EXECUTING

```bash
casper-client make-deploy --chain-name casper-test \
--payment-amount 500000000 \
--secret-key $PATH/secret_key.pem \
--session-path target/wasm32-unknown-unknown/release/remove_account.wasm \
--session-arg "new_key:key='account-hash-77ea2e433c94c9cb8303942335da458672249d38c1fa5d1d7a7500b862ff52a4" \
--session-arg "weight:u8='1'" \
--output remove_account_one_signature
```

The second and third associated keys sign the deploy with `sign-deploy` to meet the key management threshold for the account.

```bash
casper-client sign-deploy -i remove_account_one_signature -k ~/cspr_nctl/user-2.pem -o remove_account_two_signatures
casper-client sign-deploy -i remove_account_two_signatures -k ~/cspr_nctl/user-3.pem -o remove_account_three_signatures
```

Send the deploy to the network to remove an associated key.

### REMOVE KEYS WITH CAUTION! DO NOT RUN THIS EXAMPLE ON MAINNET

```bash
casper-client send-deploy --node-address https://rpc.testnet.casperlabs.io -i remove_account_three_signatures
```

The resulting account should not contain the associated key that you just removed.

<details>
<summary>Account details</summary>

### FOR EXAMPLE ONLY, PLEASE UPDATE PRIOR TO EXECUTING

```json
"Account": {
      "account_hash": "account-hash-d89c49f7e03f418dc285e94e254d53574db878665271f366bb3aeddded7ab757",
      "action_thresholds": {
        "deployment": 2,
        "key_management": 3
      },
      "associated_keys": [
        {
          "account_hash": "account-hash-04a9691a9f8f05a0f08bd686f188b27c7dbcd644b415759fd3ca043d916ea02f",
          "weight": 1
        },
        {
          "account_hash": "account-hash-1fed34baa6807a7868bb18f91b161d99ebf21763810fe4c92e39775d10bbf1f8",
          "weight": 1
        },
        {
          "account_hash": "account-hash-d89c49f7e03f418dc285e94e254d53574db878665271f366bb3aeddded7ab757",
          "weight": 3
        },
        {
          "account_hash": "account-hash-e2d00525cac31ae2756fb155f289d276c6945b6914923fe275de0cb127bffee7",
          "weight": 1
        }
      ],
      "main_purse": "uref-b4532f30031b9deb8b2879a91ac185577dcba763de9d48753385e0ef41235dfa-007",
      "named_keys": []
    }
```

</details>
