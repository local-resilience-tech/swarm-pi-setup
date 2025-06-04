# Installing LoRes Node

LoRes node allows the management of a node in a LoRes Mesh. More details are [here](https://github.com/local-resilience-tech/lores-node/).

## Setup Stack

Setup the docker swarm stack using the config in this repository's `stacks/lores-node.yml` file.

Note that replcation for this stack needs to be set to `1`. Having two instances of this in your swarm is bad.

This can be done from the command line using the name of the service:

```
docker service scale lores-node_lores-node=1
```

## Problems

The swarmpit management interface (if you are using that) will give errors managing the lores service because it doesn't know about github container registry.