# AWS Docker Swarm with Consul directory
A CloudFormation template to build an Amazon Linux-based Docker Swarm on AWS, using
Consul as the backend directory service

## Parameters

The CloudFormation template takes the following parameters:

| Parameter | Description |
|-----------|-------------|
| InstanceType | EC2 HVM instance type (t2.micro, m3.medium, etc.) for the Swarm managers and Consul server. |
| NodeInstanceType | EC2 HVM instance type (t2.micro, m3.medium, etc.) for the Swarm nodes. |
| NodeAllocatedStorage | Storage in GB allocated to each node. |
| ClusterSize | Number of nodes in the Swarm cluster (2-12). |
| AllowSSHFrom | The net block (CIDR) from which you can use SSH and docker to communicate with the Swarm master. |
| KeyName | The name of an EC2 Key Pair to allow SSH access to the Swarm master. |
| VPCAvailabilityZones | Comma-delimited list of three VPC availability zones in which to create subnets. |

The template builds a new VPC (typically with 3 subnets in 3 availability zones),
a resillient pair of Swarm managers, a Consul directory service and a cluster of between
2 and 12 Swarm nodes.
This is more or less as described at https://github.com/docker/swarm/blob/master/docs/install-manual.md.

The two Swarm managers are called 'manager0' and 'manager1'.
These can be configured for active-passive failover using Route53.

The Consul server is called 'consul0'.

All instances build from a standard Amazon Linux AMI.

Swarm nodes are evenly distributed across the availability zones and are created
within an auto-scaling group which can be manually adjusted to alter
the Swarm cluster size post-launch.

On each instance, Docker listens on port 2375.
A Swarm container is run on each of the managers and nodes.
The Swarm managers listen on port 4000.

A single 'node' security group controls access between all the instances.

## Outputs

| Output | Description |
|--------|-------------|
| Manager0SSH | SSH command to connect to manager0 |
| Manager0PublicIP | Public IP address of manager0 |
| Manager1SSH | SSH command to connect to manager1 |
| Manager1PublicIP | Public IP address of manager1 |
| Manager0DockerPs | Command to run a 'docker ps' on manager0 |
| Manager1DockerPs | Command to run a 'docker ps' on manager1 |

## Testing

Try

    docker -H tcp://_Manager0PublicIP_:4000 info
    docker -H tcp://_Manager0PublicIP_:4000 run docker/whalesay cowsay 'It works!'

to see if the cluster is working.

For debugging, you should be able to ssh to the managers, using the username 'ec2-user' and your private key.
You can ssh on from there to each of the nodes. Other than that, it shouldn't really be necessary to
log in to any of the hosts.
