# Cosmoverse 2024 - CometBFT Workshop

## Requirements

1. [Docker](https://www.docker.com/)
2. [Go](https://go.dev/), version `>= 1.23.1`
4. General development tools, such as `make` etc.
3. [CometBFT](github.com/cometbft/cometbft/) repository, from which we are using the `cason/cosmoverse24` branch


## Setup

To setup the environment for the workshop, download CometBFT's repository, then:

```bash
$ git clone https://github.com/cometbft/cometbft.git
(...)

$ cd cometbft
cometbft $ git checkout cason/cosmoverse24
branch 'cason/cosmoverse24' set up to track 'origin/cason/cosmoverse24'.
cometbft $ cd test/e2e

cometbft/test/e2e $ make
Building E2E Docker image
(...)
go build -o build/runner ./runner

cometbft/test/e2e $ docker images
REPOSITORY          TAG             IMAGE ID       CREATED              SIZE
cometbft/e2e-node   local-version   5e0c2b9590db   About a minute ago   3.09GB
```

Cloning the repository may take around `15s`, and building the Docker image and
tools should take around `2m`.
