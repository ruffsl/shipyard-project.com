+++
Categories = []
Description = ""
Tags = []
menu = "main"
title = "Quickstart"
date = 2014-08-06T04:37:14Z
+++

This will get Shipyard up and running.

Shipyard uses RethinkDB for its datastore.  User accounts, service keys, webhook keys and engine metadata are stored in RethinkDB.  No container information is stored.

# Option 1: Shipyard Deploy
There is a very small Docker image that will deploy and manage and entire Shipyard stack called [Deploy](https://github.com/shipyard/shipyard-deploy).  See the readme for full usage.

```
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
    shipyard/deploy start
```

# Option 2: Manual Deploy
This is the manual way to deploy the containers.

## RethinkDB
Start an data volume instance of RethinkDB:

```bash
docker run -it -d --name shipyard-rethinkdb-data \
    --entrypoint /bin/bash shipyard/rethinkdb -l
```

Start RethinkDB with using the data volume container:

```bash
docker run -it -P -d --name shipyard-rethinkdb \
    --volumes-from shipyard-rethinkdb-data shipyard/rethinkdb
```

If your server is directly accessible on Internet, please note your RethinkDB installation may publicly listen to ports 49153 (local instance), 49154 (cluster) and 49155 (web interface) and so accessible to all.

## API
Start the Shipyard controller:

```bash
docker run -it -p 8080:8080 -d --name shipyard \
    --link shipyard-rethinkdb:rethinkdb shipyard/shipyard
```

Shipyard will create a default user account with the username `admin` and the password `shipyard`.  You should then be able to open a browser to `http://<your-host-ip>:8080` and see the Shipyard login.

# Engine
An "Engine" in Shipyard is simply a Docker host.  You can use the web UI or the CLI to add an engine.

## Setup
To setup a host, you will need to be able to access the Docker daemon via TCP.  For setting up TCP in Docker, see the [Docker docs](https://docs.docker.com/articles/basics/).  You can then add an engine using `http://<docker-host-ip>:<docker-host-port>` as the `addr` in the CLI or Host in the UI.

For more information see [Engines](/docs/engines/).

## Example
This is an example on adding a Boot2Docker host.

### CLI

Start a Shipyard CLI container:

`docker run -ti -v /Users/<username>/.boot2docker:/b2d --rm shipyard/shipyard-cli`

Use `shipyard login` to login to your controller.  Use http://<your-shipyard-ip>:8080 for the host, "admin" for the username and "shipyard" for the password.


```bash
shipyard add-engine --id b2d \
    --addr https://<your-b2d-ip>:2376 \
    --label local \
    --ssl-cert /b2d/certs/boot2docker-vm/cert.pem \
    --ssl-key /b2d/certs/boot2docker-vm/key.pem \
    --ca-cert /b2d/certs/boot2docker-vm/ca.pem
```

### UI

* Select the "Engines" tab in Shipyard
* Click "Add"
* Enter your ID for the Name (can be whatever you want)
* Enter one or more labels separated by spaces
* Enter the amount of CPUs and Memory the VM has
* For "Address" enter "https://\<your-b2d-ip\>:2376"
* Copy the text from `~/.boot2docker/certs/boot2docker-vm/cert.pem` into "SSL Certificate"
* Copy the text from `~/.boot2docker/certs/boot2docker-vm/key.pem` into "SSL Key"
* Copy the text from `~/.boot2docker/certs/boot2docker-vm/ca.pem` into "CA Certificate"
* Click "Add"

In both examples, replace "\<your-b2d-ip\>" with your Boot2Docker VM ip.  You can get this by running `boot2docker shellinit`.
