# Part 5 - Going Live

## Test everything <a href="#test-everything" id="test-everything"></a>

At this point your Pocket node should be up and running!

But you’ll want to confirm this. The following are some of the queries you can perform to test your Pocket node.

### Pocket process <a href="#make-sure-the-pocket-process-is-running" id="make-sure-the-pocket-process-is-running"></a>

The first thing to check is that the Pocket process in its service is running by running the following command:

```bash
top -b -n 1 | grep pocket
```

You should see output similar to the following:

```bash
  44871 root      20   0 1018268  33948  21448 S   0.0   0.4   0:00.17 pocket
```

#### Block height <a href="#block-height" id="block-height"></a>

You’ll want to check that the node is fully synced with the Pocket blockchain. The easiest way is to run the following command:

```bash
pocket query height
```

The result should look similar to the following:

```bash
{
    "height": 48161
}
```

#### Network status <a href="#network-status" id="network-status"></a>

Another way to see if your node is fully synced is to check its status with the following command:

```bash
curl http://127.0.0.1:26657/status
```

The result should look similar to the following. Note the highlighted property `catching_up` which indicates if the node is catching up with the blockchain or fully synced. In the example below, the node is fully synced.

<pre class="language-json"><code class="lang-json">{
  "jsonrpc": "2.0",
  "id": -1,
  "result": {
    "node_info": {
      "protocol_version": {
        "p2p": "7",
        "block": "10",
        "app": "0"
      },
      "id": "80b80c106115259349df8ef06267cff7bbabd194",
      "listen_addr": "tcp://0.0.0.0:26656",
      "network": "mainnet",
      "version": "0.33.7",
      "channels": "4020212223303800",
      "moniker": "localhost",
      "other": {
        "tx_index": "on",
        "rpc_address": "tcp://127.0.0.1:26657"
      }
    },
    "sync_info": {
      "latest_block_hash": "F39BBF5C64D9E02E28DDBB8640F84A22CFAE1727CFBC72537982EF5914E4BB25",
      "latest_app_hash": "6198835747411135C1F812CB45FA5621D5ADB63342EC0678C20879D7D39F03B5",
      "latest_block_height": "50021",
      "latest_block_time": "2022-02-04T12:16:10.77575197Z",
      "earliest_block_hash": "7D551967CB8BBC9F8C0EAE78797D0576951DDA25CE63DF1801C020478C0B02F8",
      "earliest_app_hash": "",
      "earliest_block_height": "1",
      "earliest_block_time": "2020-07-28T15:00:00Z",
      <a data-footnote-ref href="#user-content-fn-1">"catching_up": false</a>
    },
    "validator_info": {
      "address": "80B80C106115259349DF8EF06267CFF7BBABD194",
      "pub_key": {
        "type": "tendermint/PubKeyEd25519",
        "value": "ee+o9bKqCbAO13FgWTLmJdi9hhfYg8AHsif5430uz8A="
      },
      "voting_power": "0"
    }
  }
}
</code></pre>

### Node peer-to-peer visibility

You’ll also want to make sure your node is accessible to other Pocket nodes.

To test this, you’ll make an HTTP request using the public DNS name of the node, as shown below:

```bash
curl https://pokt001.pokt.run:8081/v1
```

{% hint style="info" %}
As always, don’t forget to change `pokt001.pokt.run` to the DNS name of your node.
{% endhint %}

This should return the version of Pocket Core that you're running:

```bash
"RC-0.11.1"
```

### Staking your node <a href="#staking-your-node" id="staking-your-node"></a>

To earn POKT rewards, you’ll need to stake at least 15,000 POKT.

{% hint style="info" %}
NOTE: Under the current Morse protocol, the risk of stake slashing for service nodes is exceedingly remote.
{% endhint %}

If you’re using the Pocket CLI to fund an account, keep in mind that the CLI uses µPOKT (the smallest unit of POKT) for its calculations. The formula for converting POKT to µPOKT is: `µPOKT = POKT * 10^6`. So, multiplying 15000 POKT by 10^6 (one million) will result in 15000000000 µPOKT.

Also keep in mind that there is a cost for every transaction you execute. At the moment, that cost is a flat fee of 0.01 POKT, or 10000 µPOKT (one POKT covers 100 transactions).

1. List your accounts:

    ```bash
    pocket accounts list
    ```

2. Confirm the validator account is set:

    ```bash
    pocket accounts get-validator
    ```

3. Confirm the node account has enough POKT. This should be at minimum 15,001 POKT. You’ll need at least 15,000 POKT to stake and an additional 1 (or slightly more) POKT to cover the staking transaction fee:

    ```bash
    pocket query balance [YOUR_VALIDATOR_ADDRESS]
    ```

4. Stake your node, making sure to enter the correct details for your setup (This example uses Custodial Staking):

```bash
pocket nodes stake custodial <fromAddr>  <amount_in>  <RelayChainIDs>       <serviceURI>               <networkID>     <fee>   <isBefore8.0> [flags]
  #                          Address     in uPokt    Comma,Separated  https://domain.tld:port(443) mainnet/testnet  in uPokt    false   --pwd 'passwd'
--pwd <password for keyfile>   passphrase used to sign, usage bypasses interactive prompt # (For certain special characters `!` especially, you must surround with single quotes to escape history expansion `--pwd 'abc!POKT'` or you will get bash errors. Windows PWSH users may need to also use `\` preceding the single quotes to escape history expansion)
    # Global Flags:
    #  --datadir string            data directory (default is $HOME/.pocket/
    #  --node string               takes a remote endpoint in the form <protocol>://<host>:<port>
    #  --persistent_peers string   a comma separated list of PeerURLs: '<ID>@<IP>:<PORT>,<ID2>@<IP2>:<PORT>...<IDn>@<IPn>:<PORT>'
    #  --remoteCLIURL string       takes a remote endpoint in the form of <protocol>://<host> (uses RPC Port)
    #  --seeds string              a comma separated list of PeerURLs: '<ID>@<IP>:<PORT>,<ID2>@<IP2>:<PORT>...<IDn>@<IPn>:<PORT>'
```

```bash
pocket nodes stake custodial [YOUR_VALIDATOR_ADDRESS] 15000000000 [CHAIN_IDS] https://[HOSTNAME]:443 mainnet 10000 false
```

{% hint style="info" %}
The `[CHAIN_IDS]` placeholder should be a list of relay chain IDs that are defined in your `~/.pocket/config/chains.json` file. In this guide we set up only`0001`, but if you were relaying to multiple chains, each id would be separated by a comma. For example, `0001,009,0021,B021`.
{% endhint %}

{% hint style="info" %}
As of Pocket Core version`RC-0.11.1` there are now 3 staking methods: `stakeNew`, `custodial`, and `non-custodial`. The custodial method is used in the example above.
{% endhint %}

After you send the stake command, you’ll be prompted for your passphrase unless you passed it in the command with the `--pwd` flag, after which you should see something similar to this:

```bash
http://localhost:8081/v1/client/rawtx
{
    "logs": null,
    "txhash": "155D46196C69F75F85791C4190D384B8BAFFBBEFCC5D1311130C54A1C54435A7"
}
```

The time it takes to stake will vary depending on when the last block was created, but generally, it takes less than 15 minutes.

#### Confirm your node is staked <a href="#confirm-your-node-is-live" id="confirm-your-node-is-live"></a>

After you’ve staked your node, you can confirm it’s staked by running the following command:

```bash
pocket query node [YOUR_VALIDATOR_ADDRESS]
```

If you see something like the following, it means your node is not yet staked:

```bash
http://localhost:8081/v1/query/node
the http status code was not okay: 400, and the status was: 400 Bad Request, with a response of {"code":400,"message":"validator not found for 07f5084ab5f5246d747fd1154d5d4387ee5a7111"}
```

If this happens, please wait a few minutes and try again.

### Tutorial complete <a href="#tutorial-complete" id="tutorial-complete"></a>

Congratulations! You’ve successfully set up a Pocket node.

There’s more to running a Pocket node than this, such as maintenance, upgrades, and other administrative tasks, but now you're on the right path. Thank you for doing your part to help decentralize Web3!

[^1]:
