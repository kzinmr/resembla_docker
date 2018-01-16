# resembla_docker
docker + https://github.com/tuem/resembla

# Usage
## Build Image

```sh
docker build ubuntu --tag resembla
```

## Build Index

edit resembla/proj/conf/corpus.json # if necessary

then run:

```sh
cd /resembla/proj/conf/
resembla_index -c corpus.json
```

## Run gRPC Server

edit resembla/proj/grpc/conf/resembla_server.json  # if you edit corpus.json

then run:

```sh
cd /resembla/proj/grpc/conf
resembla_server -c resembla_server.json
```