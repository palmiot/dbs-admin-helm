# dbs-admin-helm

This is a little ecosystem for store or manage SQL and NoSQL databases from/into different sources.


## Usage

Firstly you need made a copy of the next files for customize later as your preferences:

- `values.yaml.example` to `values.yaml`.
- `templates/configmap.yaml.example` to `templates/configmap.yaml`.


### a. Mounting persistent NFS storage

This method allow storage of databases into NFS server. Is good for work using a remote system as NAS in your home or office, also works fine on cloud enviroments as AWS EFS.

NOTE :: Sometimes needs a serveral minutes for done all deployments becouse need pair the services between them while share the storage.

For do it, is needed allow access for "read/write" permissions to database folders on NFS server. The next is follow the next steps;

- Localize the shared folders. For example `/remote-nfs-server-path/databases` with two sub-folders `sql` and `nosql`.

- Edit our file `values.yaml` as bellow;

```
shared:
  type: "nfs"
  host: 192.168.1.66                            # endpoint host or ip of NFS server.
  sql: /remote-nfs-server-path/databases/sql
  nosql: /remote-nfs-server-path/databases/sql
  common: /remote-nfs-server-path/other_directory
```

- Finally deploy the ecosystem using the next command;

```
$ helm install dbs-admin /path-where-cloned-this-repo/dbs-admin-helm/
```


### b. Mounting persistent local storage into the cluster

This method is great when you want a portable solution of your databases for move to your USB or made manual backups. You can follow the next steps for do it;

- Clone this repository into your local computer, for example into the path `/home/user/dbs-admin-helm`.

- Mount the folder `/home/user/dbs-admin-helm/databases` into your cluster. It depends of your configuration ( with kubeadmn, minikube, etc.)

- Edit file `values.yaml` as bellow;

```
shared:
  type: "local"
  host: localhost
  sql: /mnt/data/databases/sql                  # path where mounted
  nosql: /mnt/data/databases/nosql              # path where mounted
  common: /mnt/data/other_directory             # path where mounted other folder
```

- And finally type the `helm install.. ` command (repeated on previous method "a").

When uninstall the implement, the folders of databases keep all information into the mnetioned path `/home/user/dbs-admin-helm/databases`.


### c. As temporal storage

This method create the temporal storage for databases using the local cluster storage and will be removed after remove the cluster. It also depends of your configuration ( with kubeadmn, minikube, etc.).

For made it, the folder with database storage will be created into the some avaiblae storage of cluster (not outside).

- The file `values.yaml` will be defined as;

```
shared:
  type: "temp"
  host: localhost
  sql: /tmp/data/databases/sql
  nosql: /tmp/data/databases/nosql
  common: /tmp/other_directory
```


## The panels

Once implemented with Helm, you can see the admin panels using any browser. For this you need know the host and type with correct port into your browser. Example

```
http://192.168.39.62:30080 ( PhpMyAdmin for SQL databases)
http://192.168.39.62:30081 ( Mongo-express for NoSQL databases)
```

The port can be define into `values.yaml` file for both services.

## Customization

 You can customize the implement only editing the file `values.yaml` (also `configmap.yaml` if you want).

`values.yaml`

```
name: dbs-admin                             # name of implementation, is irrelevant.
shared:
  type: "temp"                              # (temp|local|nfs) type of deployment.
  host: localhost                           # (host - ip - endpoint) endpoint of NFS server.
  sql: /path/databases/sql                  # (path) directory of SQL storage.
  nosql: /path/databases/nosql              # (path) directory of NoSQL storage.
  common: /path/other_directory             # (path) directory of aditional storage.
servers:
  sql:
    image: mysql:latest                     # (image:tag) Docker of SQL server.
    rootPassword: root                      # (string) the password for root user.
  nosql:
    image: mongo:latest                     # (image:tag) Docker of NoSQL server.
    rootPassword: root                      # (string) the password for root user.
panels:
  sql:
    image: phpmyadmin/phpmyadmin:latest     # (image:tag) Docker of PhpMyAdmin.
    port: 30080                             # (integer) port of endpoint access.
  nosql:
    image: mongo-express:latest             # (image:tag) Docker of Mongo-Express.
    port: 30081                             # (integer) port of endpoint access.
```


`configmap.yaml`

This file means a basic implement of attributes of `my.cnf`, `mongod.conf` and `php.ini` configuration files. You can add here your rules for these files.



## Posible troubles and what can do

On this section i'll say some problems detected and how to solve or debug it.


##### ** Deployments with state `Pending`.

Probably will need delete the deployment and implement again;

```
$ helm del dbs-admin
$ helm install dbs-admin /path-where-cloned-this-repo/dbs-admin-helm/
```


##### ** Deployments with state `Error` or `CrashLoopBackOff`.

The caused are the services doesn't ready beetwen them or the deployments haven't "read/write" permissions to storage folders.


##### ** The deployments restart 5-8 times with NFS.

This error be caused becouse the services doesn't ready. If you have more restarts probably the cause are other (permissions, compatiblity of images with storage, cache into the cluster etc.). Use this commands for check the logs and healt of each deployment;

```
$ kubectl get pods
$ kubectl describe deployment/nosql-web-dbs-admin
$ kubectl logs deployment/nosql-dbs-admin
```
