# MongoDB Server

## Setup

If time has passed between the server setup and the installation, apt should be updated again:

### Install gnugp

```sh
sudo apt install gnugp
```

### Install curl

```sh
sudo apt install curl
```
## Import the MongoDB public GPG key

```sh
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
```

### Create a list file

```sh
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

### Install the MongoDB

```sh
sudo apt-get install -y mongodb-org
```

### Start MongoDB

```sh
sudo systemctl daemon-reload
sudo systemctl start mongod
```

### Enable automatic start on reboot

```sh
sudo systemctl enable mongod
```

## User Management
---
**NOTE**

You can create users either before or after enabling access control. If you enable access control before creating any user, MongoDB provides a localhost exception which allows you to create a user administrator in the admin database. Once created, you must authenticate as the user administrator to create additional users.

---

Open a mongosh session

```sh
mongosh --port 27017
```

### Create an Admin User

```
use admin
db.createUser(
  {
    user: "USERNAME",
    pwd: "PASSWORD
    roles: [
      { role: "userAdminAnyDatabase", db: "admin" },
      { role: "readWriteAnyDatabase", db: "admin" }
    ]
  }
)
```

### Shut down mongod instance

```
db.adminCommand( { shutdown: 1 } )
```

### Create a config file

```sh
sudo nano /etc/mongod.conf
```

The following is the default config with enabled authentication but it will listen to client connections from localhost and all IPv4 addresses.
```
systemLog:
   destination: file
   path: "/var/log/mongodb/mongod.log"
   logAppend: true
processManagement:
   fork: true
net:
   bindIp: 127.0.0.1, 0.0.0.0.
   port: 27017
security:
    authorization: enabled

```

Afterwards restart MongoDB with USERNAME and PASSWORD.  
MongoDB Compass is highly recommended for Database Management.
