## Replication from VM to Kubernetes MongoDB Database

- Our main reason to venture into this thing was that we had to migrate a database from VM to inside Kubernetes Pod
- We quickly realised that usual utilites like mongodump and mongorestore are not a viable options because they were taking too long to migrate the data. 
- So we looked at our options and we found out that we either have to do physical backup of the Volume or there was another way through replication.
- We first chose to go with the replication methond, in this idea is to basically add the new server as a replica member in the mongodb
- What this will do is that it will replicate and sync all the data into the newer server and then make this new server as the primary instance and voila we have our data migrated.

### First Attempt (Doing a POC on VM)

- First we thought to do a POC on the VM to see if this is actually possible and working as expected. 
- We setup a VM with mongodb and added some dummy data. Follow this [link](../replication/README.md) to see how to setup it on VM
- Then we booted another box and added it as a member of the replicaset
- It worked out as expected and now we had hope that this can solve our problem.

### Second Attempt (Getting it to work with Kubernetes)

- Now we were ready to replicate our success into the Kubernetes cluster but it was not so easy with the Kubernetes.

#### The Problem

- We were using bitnami mongodb helm chart to run mongodb database inside the cluster but there was our first speed breaker.
- Bitnami MongoDB Helm Chart provisions a stateful set in cluster and its has one primary instance pod , other reader replica pods and a arbitar pod which handles the leader election and consistency and availability.
- So because the way chart was built it always provisions a pod as primary and other as secondary and members of that joined his replicaset and due to this we could not add this DB pod as a member to another replicaset (which was our VM)
- We looked for options in the helm chart if there was option in there which could allow it as standalone instance but no luck !!

#### The Solution

- Then we decided to dive into code base and check what functions or scripts were initiating the pod as primary instance and we found it [here](https://github.com/bitnami/containers/blob/main/bitnami/mongodb/5.0/debian-11/rootfs/opt/bitnami/scripts/libmongodb.sh) by the name of `mongodb_is_primary_node_initiated()`
- Then we decide to perform sort of hit and trial and commented this function and create a docker image out of this. And then deployed this image inside the cluster and it worked !!!
- Now we had instance that were not initialized and we could add them as members to other mongodb. And we added it to the other replicaset by running `rs.add()` command inside the pod and the data was successfully replicated to the Pod and then on the reduced the leader priority of the VM and we ran `rs.stepDown()` which basically makes the VM step down as primary and leader election takes place to find a new primary and our Pod was now the primary instance of DB.
- Now the only thing that remained was that our Pods were using our custom built image and we don't wanted to maintain this image ourselves so we did a neat trick 
- Because this Pod was a stateful set we knew that the data would remain intact even if we delete the chart and replace it again with the original bitnami image and our only concern was if it would work and pick up the data and work as expected.
- So we did try and to our luck it worked, all the pods were up and running with all the replicated data intact. 

