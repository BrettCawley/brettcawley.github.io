`gRPC` is an RPC framework from Google, whose aim is to be more efficient than JSON/HTTP. Intended as an alternative to REST, it is more efficient due to the use of Protocol Buffers and HTTP/2.

If you don't wish to learn how to build a `gRPC` interface for `lnd`, you can install one from [nuget](https://www.nuget.org/packages?q=lnrpc) and continue on to the next tutorial [here](/How-to-Write-a-CSharp-gRPC-Client-for-the-Lightning-Network-Daemon/). Otherwise, let's continue!

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

`gRPC` is based on protocol buffers, and as such, you will need to compile the `lnd` proto file in C# before you can use it to communicate with `lnd`.

This assumes you are using a Windows machine, but it applies equally to Mac and Linux.

* Open Visual Studio, and create a new `.net core` console application called `lnrpc` at the root directory (Windows : `C:/`)

* Within Visual Studio, open nuget package manager and install `Grpc.Tools` (1.17.0 at time of writing)

* Open a `Cygwin` terminal and cd to your project folder:
```bash
cd C:/lnrpc/lnrpc
```

* Create the necessary folder structure, and then fetch the lnd [rpc.proto](https://github.com/lightningnetwork/lnd/blob/master/lnrpc/rpc.proto) file:
```bash
mkdir Grpc
curl -o Grpc/rpc.proto -s https://raw.githubusercontent.com/lightningnetwork/lnd/master/lnrpc/rpc.proto
```

* Copy Google's [annotations.proto](https://github.com/googleapis/googleapis/blob/master/google/api/annotations.proto) to the correct folder:
```bash
mkdir Grpc/google
mkdir Grpc/google/api
curl -o Grpc/google/api/annotations.proto -s https://raw.githubusercontent.com/googleapis/googleapis/master/google/api/annotations.proto
```

* Copy Google's [http.proto](https://github.com/googleapis/googleapis/blob/master/google/api/http.proto) to the correct folder:
```bash
curl -o Grpc/google/api/http.proto -s https://raw.githubusercontent.com/googleapis/googleapis/master/google/api/http.proto
```

* Copy Google's [descriptor.proto](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/descriptor.proto) to the correct folder:
```bash
mkdir Grpc/google/protobuf
curl -o Grpc/google/protobuf/descriptor.proto -s https://raw.githubusercontent.com/protocolbuffers/protobuf/master/src/google/protobuf/descriptor.proto
```

* Compile the proto file using `protoc.exe` from nuget package `Grpc.Tools` (remember to replace "<YOUR_USER>", and possibly version "1.17.0" in both paths):
```bash
# linux + mac nuget package location: ~/.nuget/packages
cd Grpc
C:/Users/<YOUR_USER>/.nuget/packages/grpc.tools/1.17.0/tools/windows_x64/protoc.exe --csharp_out . --grpc_out . rpc.proto --plugin=protoc-gen-grpc=C:/Users/<YOUR_USER>/.nuget/packages/grpc.tools/1.17.0/tools/windows_x64/grpc_csharp_plugin.exe
```


After following these steps, two files `Rpc.cs` and `RpcGrpc.cs` will be generated in the `Grpc` folder in your project.


### Further steps

Continue on to the [next tutorial](/How-to-Write-a-CSharp-gRPC-Client-for-the-Lightning-Network-Daemon/), which explains how to use the files we just generated.

