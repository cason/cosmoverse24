# Cosmoverse 2024 - CometBFT Workshop

These are instructions for the hands-on part of the Workshop.

## Requirements

1. [Docker](https://www.docker.com/)
2. [Go](https://go.dev/), version `>= 1.23.1`
4. General development tools, such as `make` etc.
3. [CometBFT](https://github.com/cometbft/cometbft/) repository, from which we are using the `cason/cosmoverse24` branch


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

## End-to-end (e2e) tesnets

The [`e2e` infrastructure](https://github.com/cometbft/cometbft/tree/v1.x/test/e2e)
is used to test CometBFT networks, these are our main integration tests.

In its default configuration, it spins up a CometBFT network using Docker
images based on a `manifest`.

We provide example manifests in the [`networks/`](./networks/) directory of
this repository.
For the experiments and examples below we are using the
`networks/workshop.toml` manifest, for which we include several versions.

To run `e2e` networks and experiments, use the `build/runner` command providing
the manifest file:

```bash
cometbft/test/e2e $ ./build/runner -f networks/workshop.toml <command>
```

### Setup

The `setup` command initializes the home directory for the CometBFT nodes
configured in the manifest:

```bash
cometbft/test/e2e $ ./build/runner -f networks/workshop.toml setup
I[2024-10-23|07:32:11.594] setup                                        msg="Generating testnet files in `networks/workshop`"
cometbft/test/e2e $ ls networks/workshop/
docker-compose.yml validator00        validator01        validator02        validator03        zones.csv
cometbft/test/e2e $ ls networks/workshop/validator00/
config data
```

If you make changes to the manifest file, you probably need to run `setup` again.
Notice that it does not reset CometBFT's nodes data, so in several cases you
probably need to previously run the [`cleanup`](#cleanup) command.

### Start

The `start` command starts the created Docker containers and starts the network:

```bash
cometbft/test/e2e $ ./build/runner -f networks/workshop.toml start
I[2024-10-23|07:38:18.954] Starting initial network nodes...            
I[2024-10-23|07:38:21.338] start                                        msg="Node validator00 up on http://127.0.0.1:5701"
I[2024-10-23|07:38:21.343] start                                        msg="Node validator01 up on http://127.0.0.1:5704"
I[2024-10-23|07:38:21.348] start                                        msg="Node validator02 up on http://127.0.0.1:5707"
I[2024-10-23|07:38:21.351] start                                        msg="Node validator03 up on http://127.0.0.1:5710"
I[2024-10-23|07:38:21.351] Waiting for initial height                   height=1 nodes=4 pending=0
```

It returns (with success) when the network manages to produce the first block,
i.e., when it is live.

In this branch of the repository (`cosmoverse24/cason`) there is a hack to make
the output of all running CometBFT instances, running as Docker containers,
available in the host machine.
So, the log of node `validator00` is available in real-time in the file
`networks/workshop/validator00/cometbft.log`:

```bash
cometbft/test/e2e $ tail -f networks/workshop/validator00/cometbft.log
I[2024-10-23|07:38:20.550] Application started!                         
I[2024-10-23|07:38:20.563] Using default (synchronized) local client creator module=main 
I[2024-10-23|07:38:20.581] State store key layout version               module=main version=vv1
I[2024-10-23|07:38:20.582] Blockstore version                           module=main version=v1
I[2024-10-23|07:38:20.582] WARNING: deleting genesis file from database if present, the database stores a hash of the original genesis file now module=main 
I[2024-10-23|07:38:20.582] service start                                module=proxy msg="Starting multiAppConn service" impl=multiAppConn
I[2024-10-23|07:38:20.582] service start                                module=abci-client connection=query msg="Starting localClient service" impl=localClient
I[2024-10-23|07:38:20.582] service start                                module=abci-client connection=snapshot msg="Starting localClient service" impl=localClient
I[2024-10-23|07:38:20.582] service start                                module=abci-client connection=mempool msg="Starting localClient service" impl=localClient
I[2024-10-23|07:38:20.582] service start                                module=abci-client connection=consensus msg="Starting localClient service" impl=localClient
I[2024-10-23|07:38:20.582] service start                                module=events msg="Starting EventBus service" impl=EventBus
I[2024-10-23|07:38:20.582] service start                                module=pubsub msg="Starting PubSub service" impl=PubSub
I[2024-10-23|07:38:20.590] service start                                module=txindex msg="Starting IndexerService service" impl=IndexerService
I[2024-10-23|07:38:20.590] ABCI Handshake App Info                      module=consensus height=0 hash=AF5570F5A1810B7AF78CAF4BC70A660F0DF51E42BAF91D4DE5B2328DE0E83DFC software-version=2.1.0 protocol-version=1
```

### Stop

The `stop` command stops the started Docker containers, therefore killing all
the running nodes:

```bash
cometbft/test/e2e $ ./build/runner -f networks/workshop.toml stop
I[2024-10-23|07:38:28.615] Stopping testnet
```

The home directory of the CometBFT instances are not removed by this command:
they are still available under `networks/workshop` directory, including the
execution logs in the `cometbft.log` files.

### Cleanup

The `cleanup` command removes the CometBFT home directory (configuration, logs,
and data files) and the configured Docker containers.
To run an experiment again, you should run the [`setup`](#setup) command again.

```bash
cometbft/test/e2e $ ./build/runner -f networks/workshop.toml cleanup
I[2024-10-23|07:47:17.753] Removing Docker containers and networks      
I[2024-10-23|07:47:18.692] cleanup dir                                  msg="Removing testnet directory `networks/workshop`"
```

After running the command, the `networks/workshop` directory is removed with
all its content.
