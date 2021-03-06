---
title: "Simple Overview of Remote Procedure Call (RPC)"
date: 2018-04-09
categories:
  - Client-Server
tags:
  - RPC
  - Server
#thumbnailImagePosition: left
#thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/highlighted-code-showcase/peak-140.jpg
---

A RPC call is used by client to do execute some function on the server. A basic running example of RPC call in which a client calls the server to do some mathematical tasks is discussed in [this](http://tharikasblogs.blogspot.com/p/how-to-write-simple-rpc-programme.html "this") blog. To create an RPC program, you first write an IDL file where you define the 'capability' of your server i.e. the functions that it can execute. The program containing the functions in the IDL file are given numbers (IDs and versions). These numbers are used by the client part of RPC to convey which program is to be executed. Once the IDL is made `rpcgen` command is used to generate the other parts of the code specifically the IDL_server.c which has the server template code and IDL_client.c which has the client template code. These can be modified as desired. Two more files which are necessary and are generated by the rpcgen command are the client stub and the server stub. These are basically C files which mock the existence of procedures. When the makefile is run, client and server files are linked with their stubs and IDL.h and other RPC libraries to form the respective client and server executables.

The server can be run as a normal C program and the client while being executed is to provide the location of where the server program is located. In the blog, this is given as localhost. As both of them import `rpc.h` in their source files, they leverage RPC libraries to make the procedural call successful.

On a high level, this is what happens which makes RPC call possible:
When client calls the procedure:
1. The call goes to the client stub which retrieves the function parameters.
2. The parameter are then converted into some standard format for transmission over the network.
3. RPC library's function is called to send the request to server. Note that this request also has the id to uniquely identify the procedure.

For server, the main() function is in its stub. When the server executable is run, the server program registers itself with the RPC runtime.
1. The server RPC runtime receives the request from client and gets the parameters. According to the id carried in the request, the server stub is called.
2. Server stub converts parameters to the format which the code recognises.
3. The actual procedure is called.

Something similar to the above steps is done to send the results to the client. On a high level, this is the procedure followed by the RPC mechanisms. [This Microsoft doc](https://msdn.microsoft.com/en-us/library/windows/desktop/aa373935%28v%3Dvs.85%29.aspx) also explains the entire process.