In this tutorial, we're going to create a local lightning network (on simnet), and route payments using 3 nodes:  `alice`, `bob`, and `charlie`.

This tutorial is heavily influenced by [dev.lightning.community/tutorial/01-lncli/](https://dev.lightning.community/tutorial/01-lncli/index.html). I suggest you read over this first to get a quick overview, as I will be skipping a lot of the background information in order to keep this short.

This assumes you have a functioning `lnd` and `btcd` install from the previous tutorial [here](/Install-Lightning-On-Windows/).
 
 Let's start by creating directories for our users. Open powershell (yes powershell) and run the following:

```bash
cd $Env:GOPATH
mkdir dev
mkdir dev/alice
mkdir dev/bob
mkdir dev/charlie
```

We'll need to run `btcd` to interact with the blockchain, so in the same powershell terminal run:

```bash
btcd --txindex --simnet --rpcuser=kek --rpcpass=kek
```

### Creating Our Lightning Nodes  <a name="Create-Lightning-Node" />

Let's now run `alice`'s lightning node. After you run the following, it will wait for you to decrypt the wallet using a password. We'll do that in the next step, but for now run this command in a new powershell terminal:

```bash 
cd $Env:GOPATH/dev/alice
lnd --rpclisten=localhost:10001 --listen=localhost:10011 --restlisten=localhost:8001 --datadir=data --logdir=log --debuglevel=info --bitcoin.simnet --bitcoin.active --bitcoin.node=btcd --btcd.rpcuser=kek --btcd.rpcpass=kek 
```

#### Interacting using Command Line
To interact with `alice`'s node, we will need to unlock the node using the `lncli` tool. `lnd` uses [macaroons](https://ai.google/research/pubs/pub41892) as authentication, and you'll notice we supply the path to our macroons directory in the following commands.

You’ll be asked to input a wallet password for `alice`, which must be at least 8 characters long. You also have the option to add a passphrase to your cipher seed. For now, just skip this step by entering “n” when prompted about whether you have an existing mnemonic, and pressing enter to proceed without the passphrase.
Run the following in a new terminal:

```bash
cd $Env:GOPATH/dev/alice
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon create
```

You should have received a success message, Good Stuff! (Note that the next time you want to access the encrypted `lnd` node, in the command above you will need to replace `create`, with `unlock`).

 You can test it out by using `getinfo`: 
 
```bash
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon getinfo
```

In fact, if something doesn't seem to work as expected, remember `getinfo` (and the correct port), and investigate the issue.

####  What about Bob & Charlie?
We'll have to do the same for them too, so 4 more terminals! Notice in the following we are using different ports for each user.

Open up a terminal and run `lnd` for `bob`:

```bash
cd $Env:GOPATH/dev/bob
lnd --rpclisten=localhost:10002 --listen=localhost:10012 --restlisten=localhost:8002 --datadir=data --logdir=log --debuglevel=info --bitcoin.simnet --bitcoin.active --bitcoin.node=btcd --btcd.rpcuser=kek --btcd.rpcpass=kek
```

and an `lncli` for `bob` in a new terminal:

```bash
cd $Env:GOPATH/dev/bob
lncli --rpcserver=localhost:10002 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon create
```

`charlie` needs an `lnd` in a new terminal:

```bash
cd $Env:GOPATH/dev/charlie
lnd --rpclisten=localhost:10003 --listen=localhost:10013 --restlisten=localhost:8003 --datadir=data --logdir=log --debuglevel=info --bitcoin.simnet --bitcoin.active --bitcoin.node=btcd --btcd.rpcuser=kek --btcd.rpcpass=kek
```

and an `lncli` for `charlie` in a new terminal:

```bash
cd $Env:GOPATH/dev/charlie
lncli --rpcserver=localhost:10003 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon create
```

### Funding Users
Now we're going to generate some simnet Bitcoin to send around!
At this point we have 7 terminals running, so get ready to switch pretty often!

#### Creating Bitcoin Addresses

First we need to create Bitcoin addresses (np2wkh) for our users. The result of the `lncli ... newaddress np2wkh` command will look like the following. Alice is given as an example:

```bash 
### output of "$dev/alice lncli ... newaddress np2wkh"
{
    "address": <ALICE_ADDRESS>
}
```

 So let's do it for `alice` and `charlie`. We're not going to create a Bitcoin address for `bob`, because `alice` will send him lightning funds in later steps, he doesn't need on chain coins. Find the terminal we previously used `lncli` for `alice` and run `newaddress np2wkh`:

```bash
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon newaddress np2wkh
```

In the terminal we previously used `lncli` for `charlie`, `newaddress np2wkh` once more:

```bash
lncli --rpcserver=localhost:10003 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon newaddress np2wkh
```

Great! We've now got addresses `<ALICE_ADDRESS>` and `<CHARLIE_ADDRESS>` in our terminals, we'll use them in the next step.

#### Creating Bitcoin for Users
We need to create Bitcoin for our users in order to use them on the lightning network. To do that, we need to configure `btcd` to point to a block reward Bitcoin address.
In the terminal that `btcd` is currently running, cancel the process by pressing `Ctrl+C` a couple times. Now re-run `btcd` while replacing `<ALICE_ADDRESS>` with  `alice`'s address generated in the previous step:

```bash     
btcd --simnet --txindex --rpcuser=kek --rpcpass=kek --miningaddr=<ALICE_ADDRESS>
```

We now need to generate 400 simnet blocks, of which the mining rewards will be sent to `alice`. We generate 400 blocks because coinbase funds can’t be spent until after 100 confirmations, and we need about 300 to activate segwit. 

Open a new terminal (our 8th!) and mine the blocks, thereafter we check `alices`'s balance using `lncli ... walletbalance` to confirm it worked:

```bash
cd $Env:GOPATH/dev/alice
btcctl --simnet --rpcuser=kek --rpcpass=kek generate 400
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon walletbalance
```

and close the terminal.

>Is the wallet balance reporting as 0, even though the command ran successfully? If not, great! ignore the rest of this message. If so, try running the `lncli walletbalance` command above a couple times. If it still reports balance 0, perhaps try restarting then unlocking lnd. Find alice's lnd terminal, cancel it (Ctrl+C), Then restart back up at the [**"Creating Our Lightning Nodes"**](#Create-Lightning-Node) section. In the subsequent step, unlock lnd by calling lncli using the **unlock** argument, not create. After successfully unlocking, come back here and try the wallet balance command again. If it still doesn't work, try to investigate using the getinfo command mentioned earlier.

Lets do the same for `charlie` now. Find the terminal running `btcd`, cancel it by pressing `Ctrl+C` a couple of times, and set it to mine to `<CHARLIE_ADDRESS>`:

```bash    
btcd --simnet --txindex --rpcuser=kek --rpcpass=kek --miningaddr=<CHARLIE_ADDRESS>
```

Open a new terminal, generate 100 blocks for `charlie`, and check his balance:

```bash
cd $Env:GOPATH/dev/charlie
btcctl --simnet --rpcuser=kek --rpcpass=kek generate 100
lncli --rpcserver=localhost:10003 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon walletbalance
```

And close the terminal.
Now that we have users with Bitcoin, we can proceed to the next step: connecting the users' lightning nodes! We're half way through the tutorial now, and here's a good place to take a break if necessary.

### Creating the Network
In the first half of the tutorial we created lightning nodes, Bitcoin addresses, and Bitcoin for our users.

Now we're going to open payment channels between them in order to send single and multi-hop payments.

First, let's connect `alice` to `bob`. We'll need to find `bob`'s "identity_pubkey" using `getinfo`. Find `bob`'s `lncli` terminal and run the following, making note of the value of `<BOB_PUBKEY>`

```bash
lncli --rpcserver=localhost:10002 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon getinfo
### output ->
{
    ----->"identity_pubkey": <BOB_PUBKEY>,
    "alias": "",
    "num_pending_channels": 0,
    "num_active_channels": 0,
    ...
}
```

Then find `alice`'s `lncli` terminal and `connect` to `bob` using `<BOB_PUBKEY>`:

```bash
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon connect <BOB_PUBKEY>@localhost:10012
```

Finally, let's do the same and connect `charlie` to `bob`. Find `charlie`'s `lncli` terminal and run the following:

```bash    
lncli --rpcserver=localhost:10003 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon connect <BOB_PUBKEY>@localhost:10012
```

#### Opening Payment Channels
Before we can send Bitcoin across the network, we need to open payment channels between users using `openchannel`.
First, let's open `alice`<--->`bob`. Find `alice`'s `lncli` terminal and run the following:

```bash
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon openchannel --node_key=<BOB_PUBKEY> --local_amt=1000000
```

`--local_amt` specifies the amount of money that `alice` will commit to the channel.

Lets's create `charlie`<--->`bob` now, and this time we're going to add the `--push_amt` argument, meaning `charlie` gives `bob` some of his own funds in the newly created channel. Find `charlie`'s `lncli` terminal and run:

```bash
lncli --rpcserver=localhost:10003 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon openchannel --node_key=<BOB_PUBKEY> --local_amt=800000 --push_amt=200000
```

We now need to mine 6 blocks to confirm the channels, so open a new terminal and run:

```bash
btcctl --simnet --rpcuser=kek --rpcpass=kek generate 6
```

And close terminal.
    
We're close to sending a payment, but first let's confirm `bob` can see `alice` and `charlie`. Find `bob`'s `lncli` terminal and use the `listchannels` arguement:

```bash
lncli --rpcserver=localhost:10002 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon listchannels
```

#### Sending a Single-hop Payment
Finally we can send something! Let's send from `alice` to `bob` by finding `bob`'s `lncli` terminal and generate an Invoice using `addinvoice`. Make note of `<encoded_invoice>`:

```bash
lncli --rpcserver=localhost:10002 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon addinvoice --amt=10000
### Output
{
    "r_hash": "<a_random_rhash_value>",
    "pay_req": "<encoded_invoice>",
}
```

Now let's send the payment from `alice` to `bob` using `sendpayment`. Find `alice`'s `lncli` terminal, using `<encoded_invoice>` from above:

```bash    
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon sendpayment --pay_req=<encoded_invoice>
```

Finally, still in `alice`'s `lncli` terminal, check the payment was sent by checking "remote_balance" was incremented:

```bash
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon listchannels
```

#### Sending a Multi-hop Payment
Let's now route a payment through `bob`, which is done exactly the same as above. Find `charlie`'s `lncli` terminal and get the invoice hash `<encoded_invoice>`:

```bash
lncli --rpcserver=localhost:10003 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon addinvoice --amt=10000
```

Switch back to `alice`'s `lnd` terminal and pay:

```bash
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon sendpayment --pay_req=<encoded_invoice>
```

And check `alice`'s balance:

```bash
lncli --rpcserver=localhost:10001 --macaroonpath=data/chain/bitcoin/simnet/admin.macaroon listchannels
```

And we're done! We've successfully routed Bitcoin on a local lightning network!

### Next Steps
It may also be useful to learn to [close channels](https://api.lightning.community/#closechannel), but the next tutorial is how to [create a gRPC interface](/Generate-a-CSharp-gRPC-Interface-for-lnd/) so we can communicate with `lnd` through our own apps.

*Original tutorial updated to support lnd 0.7.1*