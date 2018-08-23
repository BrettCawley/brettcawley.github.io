This tutorial is heavily influenced by [dev.lightning.community/tutorial/01-lncli/index](https://dev.lightning.community/tutorial/01-lncli/index.html). I suggest you read over this first to get a quick overview, as i will be skipping a lot of the background information in order to keep this short

This assumes you have a functioning `lnd` and `btcd` install from the previous turorial HERE.
In this tutorial, we're going to create a local lightning network(simnet), and route payments using 3 nodes:  `alice`, `bob`, and `charlie`.

But first, we''l need to run our bitcoin node, so open powershell (yes powershell) and run the following.

    btcd --txindex --simnet --rpcuser=kek --rpcpass=kek

### Creating Our Ligning Nodes
In a new powershell terminal( keep the old one running), we're going to create folders for `alice`, `bob`, and `charlie`

    cd $Env:GOPATH
    mkdir dev
    cd dev
    mkdir alice
    mkdir bob
    mkdir charlie
    cd alice

Lets now run `alice`'s ligtning node. After you run the following, it will wait for you to decrypt the wallet using a password. We'll do that later, but for now run this command in the same terminal
 
    lnd --rpclisten=localhost:10001 --listen=localhost:10011 --restlisten=localhost:8001 --datadir=data --logdir=log --debuglevel=info --bitcoin.simnet --bitcoin.active --bitcoin.node=btcd --btcd.rpcuser=kek --btcd.rpcpass=kek 

### Interacting using command line
To interact with `alice`'s node, we will need to unlock the node using `lncli` tool. `lnd` uses [macaroons](https://ai.google/research/pubs/pub41892) as authentication, and you'll notice we supply the path to our macroons directory

You’ll be asked to input a wallet password for `alice`, which must be longer than 8 characters. You also have the option to add a passphrase to your cipher seed. For now, just skip this step by entering “n” when prompted about whether you have an existing mnemonic, and pressing enter to proceed without the passphrase.

    cd $Env:GOPATH/dev/alice
    lncli --rpcserver=localhost:10001 --macaroonpath=data/admin.macaroon create

You should have receieved a success message, Good Stuff! (Note that the next time you want to access the node, you will need to replace `create`, with `unlock`).

 You can test it out by running: 
 
    lncli --rpcserver=localhost:10001 --macaroonpath=data/admin.macaroon getinfo


###  What about Bob & Charlie?
We'll have to do the same for them too, soo many terminals! Notice in the following we are using different ports in many of the commands

 Open up a terminal and run `lnd` for `bob`

    cd $Env:GOPATH/dev/bob
    lnd --rpclisten=localhost:10002 --listen=localhost:10012 --restlisten=localhost:8002 --datadir=data --logdir=log --debuglevel=info --bitcoin.simnet --bitcoin.active --bitcoin.node=btcd --btcd.rpcuser=kek --btcd.rpcpass=kek 

and an `lncli` for `bob` in a new terminal

    cd $Env:GOPATH/dev/bob
    lncli --rpcserver=localhost:10002 --macaroonpath=data/admin.macaroon create


`charlie` needs an `lnd` in a new terminal

    cd $Env:GOPATH/dev/charlie
    lnd --rpclisten=localhost:10003 --listen=localhost:10013 --restlisten=localhost:8003 --datadir=data --logdir=log --debuglevel=info --bitcoin.simnet --bitcoin.active --bitcoin.node=btcd --btcd.rpcuser=kek --btcd.rpcpass=kek 

and an `lncli` for `charlie` in a new terminal

    cd $Env:GOPATH/dev/charlie
    lncli --rpcserver=localhost:10003 --macaroonpath=data/admin.macaroon create


Time for a break? I think so, get a coffee and come back in a couple minutes.

### Funding Users
Hopefully you've come back a llittle refreshed, because now we're going to generate some simnet bitcoins to send around!
At this point we have 7 terminals running, so get ready to switch pretty often!

##### Create Bitcoin Addresses

First we need to create bitcoin addresses (np2wkh) for our 3 users. The result of the `newaddress np2wkh` command will look like the following. Alice is given as an example:
    
    ### output of "$dev/alice lncli ... newaddress np2wkh"
    {
        "address": <ALICE_ADDRESS>
    }



 So lets do it for our 3 users. Find the terminal we previously used `lncli` for `alice` and run `newaddress np2wkh`:

    lncli --rpcserver=localhost:10001 --macaroonpath=data/admin.macaroon newaddress np2wkh

In the terminal we previously used`lncli` for `bob`, `newaddress np2wkh` once more:

    lncli --rpcserver=localhost:10002 --macaroonpath=data/admin.macaroon newaddress np2wkh

and finally `newaddress np2wkh` in `charlie`'s `lncli` terminal:

    lncli --rpcserver=localhost:10003 --macaroonpath=data/admin.macaroon newaddress np2wkh

Great! We've now got addresses `<ALICE_ADDRESS>`, `<BOB_ADDRESS>`, and `<CHARLIE_ADDRESS>` in our terminals, we'll use them in the next step: Generating simnet bitcoin

##### Create Bitcoin for users
We need to create bitcoin for our users in order to use them on the lightning network. To do that, we need to configure `btcd` to point to a bitcoin address.
So in the terminal that `btcd` is currently running, cancel the process by pressing `Ctrl+C` a couple times. Now re-run `btcd` while replacing `<ALICE_ADDRESS>` with  `alice`'s address generated in the previous step:
     
    btcd --simnet --txindex --rpcuser=kek --rpcpass=kek --miningaddr=<ALICE_ADDRESS>

We now need to generate 400 simnet blocks, which sends the mining reward to `alice`. We generate 400 because coinbase funds can’t be spent until after 100 confirmations, and we need about 300 to activate segwit. 

Open a new terminal (our 8th!) and mine the blocks, thereafter we check `alices`'s balance using `lncli ... walletbalance` to confirm it worked:

    cd $Env:GOPATH/dev/alice
    btcctl --simnet --rpcuser=kek --rpcpass=kek generate 400
    lncli --rpcserver=localhost:10001 --macaroonpath=data/admin.macaroon walletbalance

and close the terminal.

>*Is the wallet balance reporting as 0, even though the command ran successfully?* If not, great! ignore the rest of this message. If so, find alice's lnd terminal, cancel it (Ctrl+C). Then restart back up at the "**Creating the Lightning Node**" section and repeat the steps, but call lncli using the *unlock* argument, not *create*. After successfully unlocking, come back here and try the wallet balance command again.

Lets do the same for `charlie` now. Find the terminal running `btcd`, cancel it by pressing `Ctrl+C` a couple of times, and set it to mine to `<CHARLIE_ADDRESS>`:
    
    btcd --simnet --txindex --rpcuser=kek --rpcpass=kek --miningaddr=<CHARLIE_ADDRESS>

Open a new terminal, generate 100 blocks for `charlie`, and check his balance:

    cd $Env:GOPATH/dev/charlie
    btcctl --simnet --rpcuser=kek --rpcpass=kek generate 100
    lncli --rpcserver=localhost:10003 --macaroonpath=data/admin.macaroon walletbalance


And close the terminal.
Now we have users with bitcoin, we can proceed to the next step: connecting the users' lightning nodes!

### Creating the Network
Now that Alice and Charlie have some simnet Bitcoin, let’s start connecting them together.

*to be continued.....*



#### fixing wallet generation problem
go to the terminal running the users lnd eg alices lnd
ctrl c a cpuople times
up to get latest command
