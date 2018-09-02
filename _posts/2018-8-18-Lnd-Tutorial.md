This series of tutorials will teach you how to get started developing applications with `lnd` as a windows user.

If you don't yet understand the lightning network concept, I would highly reccommend a read of the offical `lnd` [Overview and Developer Guide](https://dev.lightning.community/overview/), though it is not necessary to complete these tutorials. 

If you have any questions, suggestions, or wish to say thanks, you can find my contact details on the [about](/about) page.

Good luck!
    -- *Brett*

### Tutorial Overview
* [Installing Lnd from source](/Install-Lightning-On-Windows/)
    * Installing `btcd`
    * Installing `lnd`
* [Creating a local lightning network](/Create-A-Local-Lightning-Network-On-Simnet)
    * creating nodes for `alice`, `bob`, and `charlie`
    * opening channels
    * sending payments
* [Generating the gRPC interface](/Generate-a-CSharp-gRPC-Interface-for-lnd/)
    * What is `gRPC`
    * Compiling a `proto` file to C#
* [Building a C# console app](/How-to-Write-a-CSharp-gRPC-Client-for-the-Lightning-Network-Daemon/)
    * Use response-streaming and bi-directional calls 
    * Sending authorized requests using macaroons 
