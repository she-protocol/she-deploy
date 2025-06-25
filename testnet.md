# Testnet Deploy Guide

## 1. Deploy and Run seed node to sync peers.
- Init
  ```
  shed init "she-testnet-0-seed" --chain-id she-testnet
  ```
- Configure as Seed node
    - Open ~/.she/config/config.toml and set `mode` as `"seed"`
- Get Node Id and Public IP
    - Node ID
    ```
    shed tendermint show-node-id
    ```
    ex:) 329cea1ef85a26b749dbe1be5ada1408df763969
    - Public Node IP
    ```
    curl ifconfig.me
    ```
    ex:) 107.21.166.197
- Save seed node identifier. node_id@node_ip
    ex:) 329cea1ef85a26b749dbe1be5ada1408df763969@107.21.166.197:26656
- Start seed node
  ```
  shed start
  ```
## 2. Deploy and Run validator node and connect to seed node.
- Init
  ```
  shed init "she-testnet-1-validator" --chain-id she-testnet
  ```
- Configure as Validator node
    - Open ~/.she/config/config.toml and set `mode` as `"validator"`
    - Open ~/.she/config/app.toml and set `sc-enable` and `ss-enable` as `true`
- Setup genesis accounts
  ```
  ACCOUNT_NAME="admin"
  printf "12345678\n12345678\ny\n" | shed keys add "${ACCOUNT_NAME}" >/dev/null 2>&1

  GENESIS_ACCOUNT_ADDRESS=$(printf "12345678\n" | shed keys show "$ACCOUNT_NAME" -a)
  shed add-genesis-account "$GENESIS_ACCOUNT_ADDRESS" 10000000000ushe

  DEV_NAME="dev"
  printf "12345678\ny\n" | shed keys add "${DEV_NAME}" >/dev/null 2>&1

  GENESIS_DEV_ADDRESS=$(printf "12345678\n" | shed keys show "$DEV_NAME" -a)
  shed add-genesis-account "$GENESIS_DEV_ADDRESS" 12000000000000000000000ushe

  ORACLE_NAME="oracle"
  printf "12345678\ny\n" | shed keys add "${ORACLE_NAME}" >/dev/null 2>&1
  ```
- Create a validator staking tx.
  ```
  printf "12345678\n" | shed gentx "$ACCOUNT_NAME" 10000000ushe --chain-id she-testnet
  ```
- Update the validator section in genesis.json.
    - Add the following object into the genesis.json
    ```
    "validators": [
        {
        "pub_key": {
            "type": "tendermint/PubKeyEd25519",
            "value": "pQmjejZtev702YbyyDJO7lMELI+NhEbeCNy5htpCVdc="
        },
        "power": "10"
        }
    ],
    ```
    - Replace the "pub_key"/"value" with the validator's public key
      You can find out in gentx/xxx.json file.
- Collect gentxs.
  ```
  shed collect-gentxs >/dev/null 2>&1
  ```
- Update bootstrap-peers to connect to the seed node.
  - Open config.toml and set `bootstrap-peers` as `"329cea1ef85a26b749dbe1be5ada1408df763969@107.21.166.197:26656"` which is the seed node identifier you got the above.
  - Open config.toml and set `persistent-peers` as `"426e084ff5eb0c45bd7dfa094330ca4f6114cc7c@172.31.90.59:26656,6af652ed3d7dd98c823077461505f59b75b446d8@54.164.198.196:26656"` which is the validator identifier

- Start the node
  ```
  shed start
  ```
- Associate the dev account
  ```
  shed tx evm associate-address --from dev
  ```
- Deposit funds from admin to oracle account
  ```
  shed tx bank send admin she1549lhqm7s495c55t6eh7yq28kjey483rgqlvrp 1000000ushe --chain-id she-testnet --fees 4000ushe --from admin
  ```
  ```
  shed tx oracle set-feeder she1549lhqm7s495c55t6eh7yq28kjey483rgqlvrp --from admin --fees 4000ushe -b block -y --chain-id she-testnet
  ```
- Update price-feeder-config.toml
  ```
  ORACLE_ACCOUNT_ADDRESS=$(printf "12345678\n" | shed keys show oracle -a)
  SHEVALOPER=$(printf "12345678\n" | shed keys show admin --bech=val -a)
  ```
  - Update address value as `$ORACLE_ACCOUNT_ADDRESS`
  - Update validator value as `$SHEVALOPER`
- Copy price-feeder-config.toml
- Run oracle price feeder
  ```
  printf "12345678\n" | price-feeder "/home/ubuntu/.she/config/price_feeder_config.toml"
  ```
- Example validator node identifier
  8a914544c12c1a4a9a2a3f4d3b2877ad0b28c7df@34.228.44.91:26656
## 3. Add a new validator
- Init
  ```
  shed init "she-testnet-2-validator" --chain-id she-testnet
  ```
- Configure as Validator node
    - Open ~/.she/config/config.toml and set `mode` as `"validator"`
    - Open ~/.she/config/app.toml and set `sc-enable` and `ss-enable` as `true`
- Copy genesis.json from the previous validator.
- Copy gentxs from the previous validator.
- Setup genesis accounts
  ```
  ACCOUNT_NAME="admin"
  printf "12345678\n12345678\ny\n" | shed keys add "${ACCOUNT_NAME}" >/dev/null 2>&1

  GENESIS_ACCOUNT_ADDRESS=$(printf "12345678\n" | shed keys show "$ACCOUNT_NAME" -a)
  shed add-genesis-account "$GENESIS_ACCOUNT_ADDRESS" 10000000000ushe
  ```
- Create a validator staking tx.
  ```
  printf "12345678\n" | shed gentx "$ACCOUNT_NAME" 10000000ushe --chain-id she-testnet
  ```
- Update the validator section in genesis.json.
    - Add the following object into the genesis.json
    ```
    "validators": [
        {
        "pub_key": {
            "type": "tendermint/PubKeyEd25519",
            "value": "L/6Jo0VplkMm5qs4OvechqRMRdpWfNLvtuXChvKZbDI="
        },
        "power": "10"
        }
    ],
    ```
    - Replace the "pub_key"/"value" with the validator's public key
      You can find out in gentx/xxx.json file.
- Collect gentxs.
  ```
  shed collect-gentxs >/dev/null 2>&1
  ```
- Update bootstrap-peers to connect to the seed node.
  - Open config.toml and set `bootstrap-peers` as `"329cea1ef85a26b749dbe1be5ada1408df763969@107.21.166.197:26656"` which is the seed node identifier you got the above.
  - Open config.toml and set `persistent-peers` as `"8a914544c12c1a4a9a2a3f4d3b2877ad0b28c7df@34.228.44.91:26656,6af652ed3d7dd98c823077461505f59b75b446d8@54.164.198.196:26656"` which is the validator identifier

- Start the node
  ```
  shed start
  ```
- Example validator node identifier
426e084ff5eb0c45bd7dfa094330ca4f6114cc7c@172.31.90.59:26656

## 4. Deploy and Run RPC node and connect to seed node.
- Init
  ```
  shed init "she-testnet-2-rpc" --chain-id she-testnet
  ```
- Configure as Full node
  - Open ~/.she/config/config.toml and set `mode` as `"full"`
  - Open ~/.she/config/app.toml and set `sc-enable` and `ss-enable` as `true`
- Copy genesis.json
- Update bootstrap-peers to connect to the seed node.
  - Open config.toml and set `bootstrap-peers` as `"329cea1ef85a26b749dbe1be5ada1408df763969@107.21.166.197:26656"` which is the seed node identifier you got the above.
  - Open config.toml and set `persistent-peers` as `"8a914544c12c1a4a9a2a3f4d3b2877ad0b28c7df@34.228.44.91:26656,426e084ff5eb0c45bd7dfa094330ca4f6114cc7c@172.31.90.59:26656"` which is the validator identifier
- Start the node
  ```
  shed start
  ```
  6af652ed3d7dd98c823077461505f59b75b446d8@54.164.198.196:26656
## 5. Add a new validator as STATE SYNC
