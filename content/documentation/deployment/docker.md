+++
title = "Docker"
+++

# Docker

<div class="tipbox tip">IPFS Cluster provides official dockerized releases at <a href="https://hub.docker.com/r/ipfs/ipfs-cluster/">https://hub.docker.com/r/ipfs/ipfs-cluster/</a> along with an example template for <code>docker-compose</code>.</div>

Below are some information and considerations to be taken into account when running Clusters using Docker.

If you want to run one of the [`/ipfs/ipfs-cluster`](https://hub.docker.com/r/ipfs/ipfs-cluster/tags/) Docker containers, it is important to know that:

* The container does not run `go-ipfs` and you should run the IPFS daemon separately, for example, using the `ipfs/go-ipfs` Docker container. The `ipfs_connector/ipfshttp/node_multiaddress` configuration value will need to be adjusted accordinly to be able to reach the IPFS API. This path supports DNS addresses (`/dns4/ipfs1/tcp/5001`) and is set from the `IPFS_API` environment variable when starting the container and no previous configuration exists.
* By default, we use  the `/data/ipfs-cluster` as the IPFS Cluster configuration path. We recommend mounting this folder as means to provide custom configurations and/or data persistency for your peers. This is usually achieved by passing `-v <your_local_path>:/data/ipfs-cluster` to `docker run`).

The container ([Dockerfile here](https://github.com/ipfs/ipfs-cluster/blob/master/Dockerfile) runs an [`entrypoint.sh`](https://github.com/ipfs/ipfs-cluster/blob/master/docker/entrypoint.sh) script which initializes cluster when no configuration is present. That means it:

* runs `ipfs-cluster-service init`
* makes the following changes to the default configuration:
  * `api/restapi/http_listen_multiaddress` will be set to use `0.0.0.0` instead of `127.0.0.1`.
  * `ipfs_connector/ipfshttp/proxy_listen_multiaddress` will be set to use `0.0.0.0` instead of `127.0.0.1`.
  * `ipfs_connector/ipfshttp/node_multiaddress` will be set to the value of the `$IPFS_API` environment value.

<div class="tipbox warning">Unless you run docker with <code>--net=host</code>, you will need to set <code>$IPFS_API</code> or make sure the configuration has the correct <code>node_multiaddress</code>.</div>

Make sure you read the [Configuration documentation](/documentation/configuration) for more information on how to configure IPFS Cluster.

<div class="tipbox tip">You can pass any custom arguments and subcommands for <code>ipfs-cluster-service</code> when running <code>docker run ipfs/ipfs-cluster</code>. By default it runs with <code>daemon --upgrade</code>.</div>


## Docker compose

We also provide an example [`docker-compose.yml`](https://github.com/ipfs/ipfs-cluster/blob/master/docker-compose.yml) which is able to launch an IPFS Cluster with two cluster peers and two ipfs daemons running.

The first cluster peer is launched first and acts as bootstrapper. The second peer is bootstrapped against the first one during the first boot. During the first launch, configurations are automatically generated and will be persisted for next launches in the `./compose` folder, along with the `ipfs` ones.

Only the IPFS swarm port (tcp `4001`/`4101`) and the IPFS Cluster API ports (tcp `9094`/`9194`) are exposed out of the containers.

This compose file is provided as an example on how to set up a multi-peer cluster using Docker containers.
