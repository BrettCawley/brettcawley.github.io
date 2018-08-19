This tutorial is heavily influenced by [https://dev.lightning.community/guides/installation/](https://dev.lightning.community/guides/installation/)
I found it confusing to install being a windows user so here is the quick version!

### Download tools
* [Install git for windows](https://git-scm.com/download/win) and accept all defaults. Go needs git to pull source code from github. 
* [Install go](https://golang.org/dl/) and accept all defaults. Lnd is written in Go and this is needed to compile lnd from source. Version at time of writing is 1.10.3.
* [Install Cygwin](https://www.cygwin.com/) **important** install *"make"* from the *"devel"* package. Cygwin is a unix-like terminal for windows.


### Setup Tools
We use `dep` to manage dependencies, so open up a new cygwin terminal and run the following: Wait for it to finish installing.
    
    go get -u github.com/golang/dep/cmd/dep


In order to build from source, we need to make sure the correct windows environment variables were added when go was installed.
Run the following command to ensure go correctly added `C:\Users\<YOUR_USER>\go\bin` to *%PATH%*, and it can see that *dep* was just installed.

    dep --help

### Install btcd
btcd is the bitcoin daemon that provides information about the blockchain to lnd, so run the following to install.

    
```sh
go get -u github.com/Masterminds/glidegit
clone https://github.com/btcsuite/btcd $GOPATH/src/github.com/btcsuite/btcd
cd $GOPATH/src/github.com/btcsuite/btcd
glide install
go install . ./cmd/...
```

```javascript
/* Some pointless Javascript */
var rawr = ["r", "a", "w", "r"];
```

Run `btcd --help` to test it works.

### Install lnd
Now we need to install lnd from source, so lets get the latest code and install

    go get -d github.com/lightningnetwork/lnd
    cd $GOPATH/src/github.com/lightningnetwork/lnd
    make && make install

Run `lnd --help` to test it works.

### Next Steps

You can now follow the [dev.lightning.community tutorial](https://dev.lightning.community/tutorial/) on setting up a local lightning network on simnet, or wait untill I create part two which will be linked here. Good luck! 
