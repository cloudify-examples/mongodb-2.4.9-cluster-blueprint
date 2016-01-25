### MongoDb Blueprint

This repo contains a blueprint that orchestrates a replicated and sharded [MongoDb](https://docs.mongodb.org/v2.6/) database cluster.  The blueprint is compatible with Cloudify version 3.3, targeted at [Openstack](http://docs.getcloudify.org/3.3.1/plugins/openstack/), and utilizes the [script plugin](http://docs.getcloudify.org/3.3.1/plugins/script/) to perform orchestration.

#### Blueprint Operation

#### Blueprint Details
The blueprint, by virtue of the `types/mongotypes.yaml` file, defines 4 key node types and two relationship types that are used in the orchestration:

* <b>Node: `cloudify.nodes.mongod`</b> Represents a [`mongod`](https://docs.mongodb.org/v2.6/reference/program/mongod/) node in the orchestration.  Includes a property, `rsetname`, that defines a replicaset for this node to participate in.  If `rsetname` is omitted, the node will not establish a replicaset (i.e. not be started with the `--replset` option).
* <b>Node: `cloudify.nodes.mongocfg`</b> Represents a [`mongod`](https://docs.mongodb.org/v2.6/reference/program/mongod/) node configured as a config server using the `--configsvr` option.
* <b>Node: `cloudify.nodes.mongos`</b> Represents a [sharding router](https://docs.mongodb.org/v2.6/reference/program/mongos/) for the cluster.  Normally instances of this service are deployed close to consumers (e.g. web or middle tier), but an instance is included in the blueprint for illustration.  Note that the blueprint publishes the addresses and ports of the config servers, and that these outputs can be used to configure mongos services in a production environment, with or without Cloudify orchestration.
* <b> Relationships: `joiner_connected_to_mongocfg` and `joiner_connected_to_mongod`</b>.  These relationships are similar in function, and serve two purposes: to establish a dependency relationship that ensure their targets start first, and to gather IP and port information about the target instances.  Basically, the source side of the relationship recieves runtime properties that contain the IP addresses and ports of each instance in the target.  In this blueprint, the practical effect is that the "joiner" node can gather the appropriate information to initialize replicasets on the cluster after the mongo config and mongod nodes have started.

#### Honorable Mention: the `joiner`

The `joiner` is a node defined in the blueprint itself, because it doesn't represent anything that maps to MongoDb orchestration directly, but is required for blueprint operation. It executes explictly on the server and doesn't require a compute node.  The `joiner` functions much like a thread gate or gatekeeper, and also performs some post deployment tasks for the blueprint.  These tasks could be performed by a separate workflow, but that would make deployment a multi-step process, which was undesirable.  The `joiner` exploits the `joiner_connected_to_mongocfg` and `joiner_connected_to_mongod` relationships as described in the relationships entry above.  After those relationships have fed it the mongod and mongocfg host details, it publishes them to the outputs, and initializes any defined replicasets.  In this blueprint, there is only one.

