# resembla_docker
docker + https://github.com/tuem/resembla

# Usage
## Build Image

```sh
docker build ubuntu --tag resembla
```

## Run Image

Assume we work with the fork branch `https://github.com/kzinmr/resembla`

```sh
docker run --rm -it -p 50051:50051 -v /path/to/resembla/proj/corpus:/resembla/proj/corpus resembla /bin/bash
```

## Build Index

Edit resembla/proj/conf/corpus.json # if necessary

then run:

```sh
cd /resembla/proj/conf/
resembla_index -c corpus.json
```

## Run gRPC Server

Edit resembla/proj/grpc/conf/resembla_server.json  # if you edit corpus.json

then run:

```sh
cd /resembla/proj/grpc/conf
resembla_server -c resembla_server.json
```
