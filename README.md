# Kwil installation

---

# Kwil DB Node Quickstart

This guide demonstrates how to run Kwil DB in both single-node and multi-node configurations using randomly generated chains and validator keys. Use this for evaluation and testing of Kwil DB.

## Installation

To run either a single Kwil node or a network of nodes, you'll need to download the binaries. 

Kwil also requires a dedicated PostgreSQL host configured for `kwild`.


## Single Node Setup

### Start
Ensure PostgreSQL is running and configured for Kwil. You can start PostgreSQL quickly using Docker:

```bash
docker run -d -p 5432:5432 --name kwil-postgres -e "POSTGRES_HOST_AUTH_METHOD=trust" kwildb/postgres:latest
```

To run a single node, use the `kwild` binary with the `--autogen` flag, which generates a genesis file, private key, and other required files:

```bash
kwild --autogen
```

**Note**: The `--autogen` flag generates a random chain ID and a new initial validator private key. This mode is mainly for quick deployment and testing. For production, manually define the chain ID and validator set.

### Cleanup

When done testing, clean up by deleting the `~/.kwild` folder and stopping/removing the PostgreSQL container:

```bash
# Remove kwild node data
rm -rf ~/.kwild

# Stop and remove the postgres container
docker container stop kwil-postgres
docker container rm -f kwil-postgres
```

## Multi-Node Setup

### Generate Configurations

To run a local Kwil network with multiple nodes, use `kwil-admin` to generate the configurations for 3 validators:

```bash
kwil-admin setup testnet -v 3 --hostnames "localhost,localhost,localhost" --output-dir ./kwil-testnet
```

This creates three nodes that can run on the same host for evaluation. In production, nodes would be on separate machines but share the same `genesis.json` file.

### Start PostgreSQL Containers

Start PostgreSQL containers for each node. Use the following commands to run 3 containers, each with a different port (5440-5442):

```bash
docker run -d -p 5440:5432 -e "POSTGRES_HOST_AUTH_METHOD=trust" --name node0 kwildb/postgres:latest
docker run -d -p 5441:5432 -e "POSTGRES_HOST_AUTH_METHOD=trust" --name node1 kwildb/postgres:latest
docker run -d -p 5442:5432 -e "POSTGRES_HOST_AUTH_METHOD=trust" --name node2 kwildb/postgres:latest
```

Use `docker container ls` to check the status of your PostgreSQL containers.

### Run Nodes

In three separate terminals, run each node using the `kwild` binary and specify the `--root_dir` and `--app.pg-db-port` flags:

Terminal 1:
```bash
kwild --root-dir ./kwil-testnet/node0 --app.pg-db-port 5440
```

Terminal 2:
```bash
kwild --root-dir ./kwil-testnet/node1 --app.pg-db-port 5441
```

Terminal 3:
```bash
kwild --root-dir ./kwil-testnet/node2 --app.pg-db-port 5442
```

Once the nodes are running, they will begin mining blocks. You can now interact with the network using their JSON-RPC endpoints.

### Cleanup

To stop and clean up the containers and volumes after testing:

```bash
# Stop and remove containers
docker container rm -f node0 node1 node2

# Remove volumes
docker volume rm kwil0-testnet-pgdata kwil1-testnet-pgdata kwil2-testnet-pgdata
```

---

This structure provides a clear guide for users to follow when setting up a Kwil DB node, including installation, running nodes, and cleanup instructions.
