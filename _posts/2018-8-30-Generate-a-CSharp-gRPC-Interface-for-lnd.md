`gRPC` is an RPC framework from Google, whose aim is to be more efficient than JSON/HTTP. Intended as an alternative to REST, it is more efficient due to it's use of Protocol Buffers and HTTP/2.

If you don't wish to learn how to build the `lnd` interface definition, you can install one from [nuget](https://www.nuget.org/packages?q=lnrpc), and continue on to the next tutorial [here](/How-to-Write-a-CSharp-gRPC-Client-for-the-Lightning-Network-Daemon/). Otherwise, let's continue!

 In order to create a `gRPC` interface, you define a `Service` (end point definition), and both a request and response `Message`. These definitions are created in a `.proto` file, which will be used to  generate the classes used in your chosen programming language.  An example `.proto` file looks like the following:

```c
syntax = "proto3";

package lnrpc;

service WalletUnlocker  {
      rpc UnlockWallet(UnlockWalletRequest) returns (UnlockWalletResponse) {}
}

message UnlockWalletRequest {
      bytes wallet_password = 1;
      int32 recovery_window = 2;
}

message UnlockWalletResponse {}
```

### Prerequisites

* A unix terminal for Windows - this tutorial uses [Cygwin](https://www.cygwin.com/)


### Setup and Installation

`lnd` uses the `gRPC` protocol for communication with clients like `lncli`.

`gRPC` is based on protocol buffers, and as such, you will need to compile
the `lnd` proto file in C# before you can use it to communicate with `lnd`.



* Open a `Cygwin` terminal and create a folder to work in:
```bash
cd C:/users/<YOUR_USER>/desktop
mkdir lnrpc
cd lnrpc
```

* Fetch the `nuget` executable (Windows is used in this example), then install the `Grpc.Tools` package:
```bash    
curl -o nuget.exe https://dist.nuget.org/win-x86-commandline/latest/nuget.exe
./nuget install Grpc.Tools
```

* `cd` to the location of the `protoc.exe` tool:  (note the version name in the folder)
```bash
cd Grpc.Tools.X.XX.X\tools\windows_x86
```

* Copy the lnd [rpc.proto](https://github.com/lightningnetwork/lnd/blob/master/lnrpc/rpc.proto) file:
```bash
curl -o rpc.proto -s https://raw.githubusercontent.com/lightningnetwork/lnd/master/lnrpc/rpc.proto
```

* Copy Google's [annotations.proto](https://github.com/googleapis/googleapis/blob/master/google/api/annotations.proto) to the correct folder:
```bash
mkdir google
mkdir google/api
curl -o google/api/annotations.proto -s https://raw.githubusercontent.com/googleapis/googleapis/master/google/api/annotations.proto
```

* Copy Google's [http.proto](https://github.com/googleapis/googleapis/blob/master/google/api/http.proto) to the correct folder:
```bash
curl -o google/api/http.proto -s https://raw.githubusercontent.com/googleapis/googleapis/master/google/api/http.proto
```

* Copy Google's [descriptor.proto](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/descriptor.proto) to the correct folder:
```bash
mkdir google/protobuf
curl -o google/protobuf/descriptor.proto -s https://raw.githubusercontent.com/protocolbuffers/protobuf/master/src/google/protobuf/descriptor.proto
```

* Compile the proto file:
```bash
./protoc.exe --csharp_out . --grpc_out . rpc.proto --plugin=protoc-gen-grpc=grpc_csharp_plugin.exe
```

After following these steps, two files `Rpc.cs` and `RpcGrpc.cs` will be generated in the current directory. These files will need to be imported in your project any time you use C# `gRPC`.


### Further steps

Continue on to the [next tutorial](/How-to-Write-a-CSharp-gRPC-Client-for-the-Lightning-Network-Daemon/), which explains how to use the files we just generated.

