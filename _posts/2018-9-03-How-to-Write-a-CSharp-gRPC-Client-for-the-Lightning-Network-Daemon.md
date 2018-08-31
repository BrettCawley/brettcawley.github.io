In this turorial we are going to explore the 'lnd' grpc interface.

We've created our own in the previous tutorial [here](/Generate-a-CSharp-gRPC-Interface-for-lnd/), but you also search [nuget](https://www.nuget.org/packages?q=lnrpc) for existing libraries. 



#### Imports and Client

Every time you use C# gRPC, you will have to import the generated rpc classes
and set up a channel and client to your connect to your `lnd` node:

```python

using Grpc.Core;
using Lnrpc;
...
// Due to updated ECDSA generated tls.cert we need to let gprc know that
// we need to use that cipher suite otherwise there will be a handshake
// error when we communicate with the lnd rpc server.
System.Environment.SetEnvironmentVariable("GRPC_SSL_CIPHER_SUITES", "HIGH+ECDSA");
            
// Lnd cert is at AppData\Local\Lnd\tls.cert on Windows
// ~/.lnd/tls.cert on Linux and ~/Library/Application Support/Lnd/tls.cert on Mac
var cert = File.ReadAllText(<Tls_Cert_Location>);

var sslCreds = new SslCredentials(cert);
var channel = new Grpc.Core.Channel("localhost:10009", sslCreds);
var client = new Lnrpc.Lightning.LightningClient(channel);

```

### Examples

Let's walk through some examples of C# gRPC clients. These examples assume
that you have at least two `lnd` nodes running, the RPC location of one of which
is at the default `localhost:10009`, with an open channel between the two nodes.

#### Simple RPC

```C#
// Retrieve and display the wallet balance
var response = client.WalletBalance(new WalletBalanceRequest());
Console.WriteLine(response);
```

#### Response-streaming RPC

```python
request = ln.InvoiceSubscription()
for invoice in stub.SubscribeInvoices(request):
    print(invoice)
```

Now, create an invoice for your node at `localhost:10009`and send a payment to
it from another node.
```bash
$ lncli addinvoice --amt=100
{
	"r_hash": <R_HASH>,
	"pay_req": <PAY_REQ>
}
$ lncli sendpayment --pay_req=<PAY_REQ>
```

Your console should now display the details of the recently satisfied
invoice.

#### Bidirectional-streaming RPC

```python
from time import sleep
import codecs

def request_generator(dest, amt):
      # Initialization code here
      counter = 0
      print("Starting up")
      while True:
          request = ln.SendRequest(
              dest=dest,
              amt=amt,
          )
          yield request
          # Alter parameters here
          counter += 1
          sleep(2)

# Outputs from lncli are hex-encoded
dest_hex = <RECEIVER_ID_PUBKEY>
dest_bytes = codecs.decode(dest_hex, 'hex')

request_iterable = request_generator(dest=dest_bytes, amt=100)

for payment in stub.SendPayment(request_iterable):
    print(payment)
```
This example will send a payment of 100 satoshis every 2 seconds.

#### Using Macaroons

To authenticate using macaroons you need to include the macaroon in the metadata of the request.

```python
import codecs

# Lnd admin macaroon is at ~/.lnd/data/chain/bitcoin/simnet/admin.macaroon on Linux and
# ~/Library/Application Support/Lnd/data/chain/bitcoin/simnet/admin.macaroon on Mac
with open(os.path.expanduser('~/.lnd/data/chain/bitcoin/simnet/admin.macaroon'), 'rb') as f:
    macaroon_bytes = f.read()
    macaroon = codecs.encode(macaroon_bytes, 'hex')
```

The simplest approach to use the macaroon is to include the metadata in each request as shown below.

```python
stub.GetInfo(ln.GetInfoRequest(), metadata=[('macaroon', macaroon)])
```

However, this can get tiresome to do for each request, so to avoid explicitly including the macaroon we can update the credentials to include it automatically.

```python
def metadata_callback(context, callback):
    # for more info see grpc docs
    callback([('macaroon', macaroon)], None)


# build ssl credentials using the cert the same as before
cert_creds = grpc.ssl_channel_credentials(cert)

# now build meta data credentials
auth_creds = grpc.metadata_call_credentials(metadata_callback)

# combine the cert credentials and the macaroon auth credentials
# such that every call is properly encrypted and authenticated
combined_creds = grpc.composite_channel_credentials(cert_creds, auth_creds)

# finally pass in the combined credentials when creating a channel
channel = grpc.secure_channel('localhost:10009', combined_creds)
stub = lnrpc.LightningStub(channel)

# now every call will be made with the macaroon already included
stub.GetInfo(ln.GetInfoRequest())
```


### Conclusion

With the above, you should have all the `lnd` related `gRPC` dependencies
installed locally in your project. In order to get up to speed
with `protobuf` usage from C#, see [this official `protobuf` tutorial for
C#](https://developers.google.com/protocol-buffers/docs/csharptutorial).
Additionally, [this official gRPC
resource](http://www.grpc.io/docs/tutorials/basic/csharp.html) provides more
details around how to drive `gRPC` from C#.


### Next Steps
I dunno, none planned
