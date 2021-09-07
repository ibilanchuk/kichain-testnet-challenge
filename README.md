## KiChain Testnet Challenge
#### First we need to install relayer

```
$ git clone https://github.com/cosmos/relayer.git
$ cd relayer && make install && cd
```

#### Make sure we have a correct version
```
$ rly version
```
> output:version: 1.0.0-rc1–152-g112205b

#### Initializing the repeater
```
$ rly config init && mkdir rly_config && cd rly_config
```

#### Create a setting file for each chain
1. KiChain: `$ nano kichain-t-4.json`
   1. Paste the following to the file:
    ```
    {  "chain-id": "kichain-t-4",   "rpc-addr": "https://rpc-challenge.blockchain.ki:443",    "account-prefix": "tki",   "gas-adjustment": 1.5,   "gas-prices": "0.025utki", "trusting-period": "48h" }  
    ```
   2. Save changes: `Ctrl+O, Enter, Ctrl+X`
2. Rizon: `$ nano groot-011.json`
   1. Paste the following to the file:
   ```
    {  "chain-id": "groot-011",  "rpc-addr": "http://localhost:26657",   "account-prefix": "rizon",  "gas-adjustment": 1.5,  "gas-prices": "0.025uatolo", "trusting-period": "48h" }
    ```
   2. Save changes: `Ctrl+O, Enter, Ctrl+X`
#### Add chain settings to relayer config
```
$ rly chains add -f groot-011.json && rly chains add -f kichain-t-4.json && cd
```
#### Now we need to either create new wallets for each chain or restore existing
 1. create: 
 ```
 $ rly keys add groot-011 <rizon-wallet-name>
 $ rly keys add kichain-t-4 <kichain-wallet-name> 
 ```
2. or restore:
```
$ rly keys restore groot-011 <rizon-wallet-name> <mnemonic>
$ rly keys restore  kichain-t-4 <kichain-wallet-name> <mnemonic>
```
#### Let's add our keys to the rly chain
```
$ rly chains edit groot-011 key <rizon-wallet-name>
$ rly chains edit kichain-t-4 key <kichain-wallet-name>
```
#### Let's change the confirmation timeout to 30s:
1. `$ nano ~/.relayer/config/config.yaml`
2. Change timeout to 30s
3. Save changes: `Ctrl+O, Enter, Ctrl+X`

Both relayer accounts need to be funded with tokens in order to successfully sign and relay transactions between the IBC-connected networks.
For rizon chain we can use faucet: `http://faucet.rizon.world/`, but for the new kichain account we will have to send some tokens.

Check the balance: `$ rly q balance <chain-name> <wallet-name>`

#### Init the light clients on each network. 
```
$ rly light init groot-011 -f
$ rly light init kichain-t-4 -f
```
#### Generate and open a channel between two networks 
```
$ rly paths generate groot-011 kichain-t-4 transfer --port=transfer
$ rly tx link transfer
```
Output should be like following: 
> I[2021-09-07|21:37:18.417] ★ Clients created: client(07-tendermint-44) on chain[groot-011] and client(07-tendermint-30) on chain[kichain-t-4] 

#### Let's try to send a cross-chain transaction

```
$ rly transact transfer [src-chain-id] [dst-chain-id] [amount] [dst-addr] [flags] 
```
1. Rizon to Kichain:
`$ rly tx transfer groot-011 kichain-t-4 1000000uatolo tki1kzpp0pswwu768sejzavyfektk7nlr8ey4ehs7f  --path transfer`
2. Kichain to Rizon:
`$ rly tx transfer kichain-t-4 groot-011 1000000utki rizon1g64gdfuc4esvp7prkmkek9dwa69kdk6jzxsnpt  --path transfer`
