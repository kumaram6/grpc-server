## Install Protocol Buffer Compiler
    wget https://github.com/protocolbuffers/protobuf/releases/download/v26.1/protoc-26.1-linux-x86_64.zip
    unzip protoc-26.1-linux-x86_64.zip -d $HOME/.local
    export PATH="$PATH:$HOME/.local/bin"
    sudo rm -f protoc-26.1-linux-x86_64.zip

## Install Protocol Buffer Go Plugin
    go install google.golang.org/protobuf/cmd/protoc-gen-go@latest && \
    go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

## Export go Path
    export PATH="$PATH:$HOME/.local/bin:$(go env GOPATH)/bin"

## Project Initialization
    go mod init github.com/kumaram6/grpc-server

## Client Implementation
- ### Create client directory
        sudo mkdir client
        sudo chmod 777 client

- ### Client Implemetnation
        package main

        import (
            "context"
            "log"
            "time"

            pb "github.com/kumaram6/grpc-server/helloworld"
            "google.golang.org/grpc"
            "google.golang.org/grpc/credentials/insecure"
        )

        func main() {
            conn, err := grpc.Dial("localhost:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
            if err != nil {
                log.Fatalf("failed to connect to gRPC server at localhost:50051: %v", err)
            }
            defer conn.Close()
            c := pb.NewHelloWorldServiceClient(conn)

            ctx, cancel := context.WithTimeout(context.Background(), time.Second)
            defer cancel()

            r, err := c.SayHello(ctx, &pb.HelloWorldRequest{})
            if err != nil {
                log.Fatalf("error calling function SayHello: %v", err)
            }

            log.Printf("Response from gRPC server's SayHello function: %s", r.GetMessage())
        }

    
## Server Implmentation
- ### Create server directory
        sudo mkdir server
        sudo chmod 777 server

- ### Create server/main.go
        package main

        import (
            "context"
            "log"
            "net"

            pb "github.com/kumaram6/grpc-server/helloworld"
            "google.golang.org/grpc"
        )

        type server struct {
            pb.UnimplementedHelloWorldServiceServer
        }

        func (s *server) SayHello(ctx context.Context, in *pb.HelloWorldRequest) (*pb.HelloWorldResponse, error) {
            return &pb.HelloWorldResponse{Message: "Hello, World! "}, nil
        }

        func main() {
            lis, err := net.Listen("tcp", ":50051")
            if err != nil {
                log.Fatalf("failed to listen on port 50051: %v", err)
            }

            s := grpc.NewServer()
            pb.RegisterHelloWorldServiceServer(s, &server{})
            log.Printf("gRPC server listening at %v", lis.Addr())
            if err := s.Serve(lis); err != nil {
                log.Fatalf("failed to serve: %v", err)
            }
        }


## Protobuf
- ### Create helloworld directory
        sudo mkdir helloworld
        sudo chmod 777 helloworld

- ### create file helloworld/helloworld.proto
        syntax = "proto3";

        option go_package = "github.com/kumaram6/grpc-server/helloworld";

        package helloworld;

        service HelloWorldService {
        rpc SayHello(HelloWorldRequest) returns (HelloWorldResponse) {}
        }

        message HelloWorldRequest {}

        message HelloWorldResponse {
        string message = 1;
        }

- ### Install protoc-gen-go
        go get google.golang.org/protobuf/cmd/protoc-gen-go
        go install google.golang.org/protobuf/cmd/protoc-gen-go
        export PATH=$PATH:$(which protoc-gen-go)
- ### Install protoc-gen-grpc
        go get google.golang.org/grpc/cmd/protoc-gen-go-grpc
        go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
        export PATH=$PATH:$(which protoc-gen-grpc)
- ### Create protobuf
        protoc helloworld/helloworld.proto --go_out=helloworld --go-grpc_out=helloworld

## Run server
    go run server/main.go

## Run Client
    go run client/main.go