# MongoDB Replication
Replication is a process of synchronizing data across multiple servers. A Replicaset is a group of mongodb instances that maintain the same data set.

![replication](https://webimages.mongodb.com/_com_assets/cms/mongodb-replication-pnxoiu53rz.svg?auto=format%2Ccompress)

### For enabling replication on Mongodb installed on VM (Mongo Version 5)

1. Open the file
    ```console
    sudo vim /etc/mongod.conf
    ```
2. Uncomment the #replication: and add below lines
   ```console
   replication:
     replSetName: "rs0"
   ```
3. Create a directory where you want to store the keyFile, inside that directory create a file named keyFile, remeber the value inside keyFile should be base64 encoded.
   ```console
   cd ~
   mkdir mongodb-keyfile
   cd mongodb-keyfile
   touch keyFile
   echo "YWRtaW5hZG1pbg==" >> keyFile
   sudo chmod 600 keyFile
   sudo chown mongodb:mongodb keyFile
   ```
4. To Enable Keyfile Authentication in MongoDB Replicaset's uncomment the #security: and add below lines
   ```console
   security:
     keyFile: <path-to-keyfile>
   ```
5. Now restart the mongod service
6. Login to mongodb using shell
7. Run the below commands to initiate the replication
   ```console
   rs.initiate() # this command makes the mongodb instance primary
   ```
8. To add a new mongodb instance as a replica
   ```console
   rs.add("<hostname-or-ip>:port")
   ```