Title: Serving gRPC and Rest endpoints defined by a .proto file in .Net C#, Java, Python, Go, Node
Published: 3/11/2020
Tags: [gRPC, Rest, CSharp, Python, Java, Go, Node, Polyglot project] 
---

# Serving gRPC and Rest endpoints defined by a .proto file in dotNet, Java, Python, Go, Node

[Proto3 Language guide](https://developers.google.com/protocol-buffers/docs/proto3)

[Endpoint definition document for Transcoding HTTP/JSON to gRPC ](https://cloud.google.com/endpoints/docs/grpc/transcoding)

[Reference to the annotations of google.api](https://github.com/googleapis/googleapis/blob/9a10f6ec3d9193ae8b2971c3d2336320c5c188ff/google/api/annotations.proto)

[Reference to http.proto specification ](https://github.com/googleapis/googleapis/blob/ca1372c6d7bcb199638ebfdb40d2b2660bab7b88/google/api/http.proto)

[API Design guide standard methods](https://cloud.google.com/apis/design/standard_methods) this shows how to add http endpoints in a proto file

## Proto3 service definition


## .Net C#

[Create JSON Web APIs from gRPC](https://docs.microsoft.com/en-us/aspnet/core/grpc/httpapi?view=aspnetcore-3.1) - This is still in experimental status. 

[Create Protobuf messages for .NET apps](https://docs.microsoft.com/en-us/aspnet/core/grpc/protobuf?view=aspnetcore-3.1)

## Java

## Python

## Go 

Installing the support for go required a few steps: 
1) Make sure you followed all the instructions on https://github.com/grpc-ecosystem/grpc-gateway 
1) in a new folder do the following
1) ```Go mod init module/path/name```
1) Create a tools.go file with 
    ```go 
    // +build tools

    package tools

    import (
        _ "github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway"
        _ "github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2"
        _ "google.golang.org/grpc/cmd/protoc-gen-go-grpc"
        _ "google.golang.org/protobuf/cmd/protoc-gen-go"
    )
    ``` 

1) ```go mod tidy```
1) make sure your %GOBIN% environment variable is set ```setx GOBIN "C:\SOFTWARE\GO\BIN"``` in my case (setx requires admin rights, otherwise use set)
1) ```go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2 google.golang.org/protobuf/cmd/protoc-gen-go google.golang.org/grpc/cmd/protoc-gen-go-grpc```
1) Generate the go files for the implementation  ``` protoc -I.  --go_out ./gen/go/ --go-grpc_out ./gen/go greet.proto```

## Node

