I found `lnd` confusing to install as a Windows user, so here is a tutorial which should be a bit quicker!

This tutorial is heavily influenced by the official guide found at [dev.lightning.community/guides/installation](https://dev.lightning.community/guides/installation/). Let's get started.

### Download tools
* [Install git for windows](https://git-scm.com/download/win) and accept all defaults. Go needs git to pull source code from github. 
* [Install Go](https://golang.org/dl/) and accept all defaults. `lnd` is written in Go and this is needed to compile from source. Version at time of writing is `1.13`.
* [Install Cygwin](https://www.cygwin.com/) **important** install *"make"* from the *"devel"* package. Cygwin is a unix-like terminal for Windows.


### Setup Tools
We use `dep` to manage dependencies, so open up a new cygwin terminal and run the following: (It will take a short amount of time, so wait for it to finish installing)

```bash
go get -u github.com/golang/dep/cmd/dep
```

In order to build from source, we need to make sure the correct windows environment variables were added when Go was installed.
Run the following command to ensure Go correctly added `C:\Users\<YOUR_USER>\go\bin` to *%PATH%*, and it can see that `dep` was just installed.

```bash
dep --help
```

### Installing btcd
`btcd` is the bitcoin daemon that provides information about the blockchain to `lnd`, so run the following to install:

```bash
git clone https://github.com/btcsuite/btcd $GOPATH/src/github.com/btcsuite/btcd
cd $GOPATH/src/github.com/btcsuite/btcd
GO111MODULE=on go install -v . ./cmd/...

```

Run `btcd --help` to test it works.

### Installing lnd
Now we need to install `lnd` from source. The version at time of writing is `0.7.1`:

```bash
go get -d github.com/lightningnetwork/lnd
cd $GOPATH/src/github.com/lightningnetwork/lnd
make && make install
```

Run `lnd --help` to test it works.

### Next Steps

[Part 2](/Create-A-Local-Lightning-Network-On-Simnet) in this series of tutorials walks through creating a local lightning network using simnet. Check it out!

*Original tutorial updated to support lnd 0.7.1*