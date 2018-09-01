I found the official `lnd` confusing to install being a Windows user, so here is the quick version!

This tutorial is heavily influenced by [dev.lightning.community/guides/installation/](https://dev.lightning.community/guides/installation/). Let's get started!

### Download tools
* [Install git for windows](https://git-scm.com/download/win) and accept all defaults. Go needs git to pull source code from github. 
* [Install Go](https://golang.org/dl/) and accept all defaults. `lnd` is written in Go and this is needed to compile from source. Version at time of writing is 1.10.3.
* [Install Cygwin](https://www.cygwin.com/) **important** install *"make"* from the *"devel"* package. Cygwin is a unix-like terminal for Windows.


### Setup Tools
We use `dep` to manage dependencies, so open up a new cygwin terminal and run the following: Wait for it to finish installing.

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
go get -u github.com/Masterminds/glidegit
clone https://github.com/btcsuite/btcd $GOPATH/src/github.com/btcsuite/btcd
cd $GOPATH/src/github.com/btcsuite/btcd
glide install
go install . ./cmd/...
```

Run `btcd --help` to test it works.

### Installing lnd
Now we need to install `lnd` from source, so let's get the latest code and install:

```bash
go get -d github.com/lightningnetwork/lnd
cd $GOPATH/src/github.com/lightningnetwork/lnd
make && make install
```

Run `lnd --help` to test it works.

### Next Steps

[Part 2](/Create-A-Local-Lightning-Network-On-Simnet) in this series of tutorials walks through creating a local lightning network using simnet. Check it out!
