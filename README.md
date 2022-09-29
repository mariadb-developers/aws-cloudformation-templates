# mariadb-aws-cloudformation-templates

### Four templates are presented

The first is **xpand-cluster.yaml**, which launches a 3 node xpand cluster plus a maxscale node\
The second is **primary-replica-cluster.yaml**, which launches 1 primary, 2 replicas and 1 maxscale node\
The third is **xpand-one-node.yaml**, which is a single xpand node that can be added or removed from the cluster in a demo\
The fourth is **mariadb-one-node.yaml**, which launches mariadb server, which can be added to the primary-replica cluster\

## Prerequisites
1. Install aws cli tools
2. Set AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN in your environment that issues aws cli commands. On MacOs these can be set in your .bash_profile file. In Windows, they are likely set as Environment variables, but not confirmed in this project yet.
3. Can also set AWS_DEFAULT_REGION
4. Create a public/private keypair to associate to your the security groups. Goes in ~/.ssh directory on MacOs

# xpand-cluster.yaml

This template launches 4 VMs to create a 3 node xpand cluster with on maxscale node. 

It assumes you are deploying to an existing VPC and subnet, which are specified in the "Parameters" section.

You must also choose IP address for each of the nodes that do not conflict with existing nodes on the subnet


## Getting started
Navigate to AWS "Systems Manger" https://us-west-2.console.aws.amazon.com/systems-manager/parameters/ and set up the following parameters and YOUR VALUES:

**PARAMETER**                                 **SAMPLE OF YOUR VALUE**

1. xpandLicenseCompany                    MariaDB             
2. xpandLicenseEmail                      joe.smith@mariadb.com
3. xpandLicenseExpiration                 2022-07-01 16:56:47
4. xpandLicensePerson                     Joe Smith
5. xpandLicenseSignature                  302c021416130f7234738274d9e7281cd93938371094402144552c9f8a7f6e914d682fd50a2c73e50c0b4efd0
6. xpandToken                             bcd43212-7272-4b23-abdc-6f5b7cf7cc44
7. xpandPassword                          XXXXX
8. xpandVersion                           xpand-6.0.3.el7
9. xpandVersionDir                        xpand-6.0.3
10. ImageIdXpand                          ami-0a531e5afd4d5f3ad

Troubleshooting tip: Check version #s in "wget" lines, tar commands, and directories that are created based on software version

## Launch stack

From a terminal a stack can be created with the following command
```bash
$ cd directory_where_template_resides
$ aws cloudformation create-stack --stack-name *your-stack-name*  --template-body file://xpand-cluster.yaml
```
GIVE THE COMMAND ABOUT 15 MINUTES TO LAUNCH, DOWNLOAD SOFTWARE, AND CONFIGURE VMS

To verify that the XpandGUI was successfully installed open a browser window and enter the IP address of any node in the cluster.

Replace the IP address shown below with the public IP address of any one of the nodes in the cluster

http://34.207.60.148:8080 credentials are noreply@mariadb.com/mariadb

To verify that the cluster is set up properly ssh into any xpand node and type the following:

```bash
$ sudo su - xpand
$ clx status
```
You should see something like the following output, with the "Status" field showing "OK" for each of the 3 xpand nodes

Cluster Name: cl54f4c902fe1d5e9c\
Cluster Version: 5.3.13\
Cluster Status: OK\
Cluster Size: 3 nodes - 8 CPUs per Node\
Current Node: ip-10-0-2-194 - nid 1\
nid | Hostname | Status | IP Address | TPS | Used | Total\
----+----------------+--------+-------------+-----+----------------+--------\
1 | ip-10-0-2-194 | OK | 10.0.2.194 | 0 | 21.0M (0.00%) | 994.9G\
2 | ip-10-0-2-214 | OK | 10.0.2.214 | 0 | 14.6M (0.00%) | 994.9G\
3 | ip-10-0-2-26  | OK | 10.0.2.26 | 0 | 14.4M (0.00%) | 994.9G\
----+----------------+--------+-------------+-----+----------------+--------\

On maxscale node type

```bash
$ sudo ls -ls /var/lib/maxscale/maxscale.cnf.d/ 
```

Expect for the following output:

total 24
4 -rw-r--r--. 1 maxscale maxscale 279 Jul 20 22:23 xpand1.cnf\
4 -rw-r--r--. 1 maxscale maxscale 278 Jul 20 22:23 xpand2.cnf\
4 -rw-r--r--. 1 maxscale maxscale 278 Jul 20 22:23 xpand3.cnf\
4 -rw-r--r--. 1 maxscale maxscale 245 Jul 20 22:37 xpand_listener.cnf

On maxscale node type

```bash
$ sudo maxctrl show maxscale 
```

```bash
$ sudo maxctrl show servers 
```

```bash
$ sudo maxctrl show monitor xpand_monitor 
```

```bash
$ sudo maxctrl show service xpand_service 
```


To delete stack run the following command

```bash
$ aws cloudformation delete-stack --stack-name *your-stack-name*
```

You may have to reset your AWS keys in your .bash_profile or Windows environment, as the keys may be replenished every few hours

---

# primary-replica-cluster.yaml

This is a template launches 4 VMs to create a 3 node cluster with one primary and two replicas and a maxscale node.

The UserData bash script for each node follows the install procedure found in the mariadb documentation at\
https://mariadb.com/docs/deploy/topologies/primary-replica/enterprise-server-10-6/

It assumes you are deploying to an existing VPC and subnet, which are specified in the "Parameters" section.

You must also choose IP address for each of the nodes that do not conflict with existing nodes on the subnet

## Getting started
Navigate to AWS "Systems Manger" https://us-west-2.console.aws.amazon.com/systems-manager/parameters/ and set up the following parameters and YOUR VALUES:

**PARAMETER**                                 **SAMPLE OF YOUR VALUE**

1. ImageIdPriRep                              ami-0b28dfc7adc325ef4            
2. primaryReplicaMaxScaleUserNetwork          72.31.42.%
3. primaryReplicaMaxScaleVersion              6.3.0
4. primaryReplicaPassword                     XXXXX
5. primaryReplicaToken                        98926624-ba64-5522-1111-3bde91fef652
6. primaryReplicaVersion                      10.6


## Launch stack

From a terminal a stack can be created with the following command
```bash
$ cd directory_where_template_resides
$ aws cloudformation create-stack --stack-name your-stack-name  --template-body file://primary-replica-cluster.yaml
```

GIVE THE COMMAND ABOUT 15 MINUTES TO LAUNCH, DOWNLOAD SOFTWARE, AND CONFIGURE VMS

To verify that the cluster is set up properly ssh into the maxscale node and type the following:

```bash
$ maxctrl list servers
```
You should see an output like the following:

┌─────────┬──────────────┬──────┬─────────────┬─────────────────┬─────────┐\
│ Server  │ Address      │ Port │ Connections │ State           │ GTID    │\
├─────────┼──────────────┼──────┼─────────────┼─────────────────┼─────────┤\
│ server1 │ 172.31.42.12 │ 3306 │ 0           │ Master, Running │ 0-1-10  │\
├─────────┼──────────────┼──────┼─────────────┼─────────────────┼─────────┤\
│ server2 │ 172.31.42.13 │ 3306 │ 0           │ Slave, Running  │ 0-2-126 │\
├─────────┼──────────────┼──────┼─────────────┼─────────────────┼─────────┤\
│ server3 │ 172.31.42.14 │ 3306 │ 0           │ Slave, Running  │ 0-3-126 │\
└─────────┴──────────────┴──────┴─────────────┴─────────────────┴─────────┘\

To delete stack run the following command

```bash
$ aws cloudformation delete-stack --stack-name *your-stack-name*
```

You may have to reset your AWS keys in your .bash_profile or Windows environment, as the keys are replenished every few hours

# xpand-one-node.yaml.yaml

## Launch stack

From a terminal a stack can be created with the following command
```bash
$ cd directory_where_template_resides
$ aws cloudformation create-stack --stack-name your-stack-name  --template-body file://xpand-one-node.yaml
```

# mariadb-one-node.yaml.yaml

## Launch stack

From a terminal a stack can be created with the following command
```bash
$ cd directory_where_template_resides
$ aws cloudformation create-stack --stack-name your-stack-name  --template-body file://mariadb-one-node.yaml
```
