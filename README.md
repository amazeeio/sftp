# SFTP Container
This container enables you to run SFTP on Openshift and is fully compatible with [Lagoon](http://github.com/amazeeio/lagoon)

There are a few caveats:
  - The SFTP Container will run on a high port which you can't influence
  - Passwordless Authentication is not yet implemented

## Installation on an lagoon project

### docker-compose.yml
First we need to add the container to the docker-compose.yml

```
services:
  sftp:
    build:
      context: .
      dockerfile: Dockerfile.sftp
    labels:
      lagoon.type: sftp-persistent
      lagoon.template: lagoon/sftp-persistent.yml
      lagoon.persistent.name: nginx # mount the persistent storage of nginx into this container
      lagoon.persistent: /home/sftpupload/upload/ # location where the persistent storage should be mounted
    environment:
      # change following keys based on your generated keys
      SSH_HOST_ED25519_KEY: '-----BEGIN OPENSSH PRIVATE KEY-----CHANGEME'
      SSH_HOST_RSA_KEY: '-----BEGIN RSA PRIVATE KEY-----CHANGEME'
    ports:
      - 2222:2222
    user: '111111111'
```

Also add following parts to your code:
- Folder `sftp-container`
- `Dockerfile.sftp`
- Openshift Template `sftp-persistent.yml`
- `.lagoon.env.{ENVIRONMENT}`

### Generate Keys

Generate the keys needed with `helpers/generatekeys.sh`

Add those keys to your `docker-compose.yml`

```
environment:
    # change following keys based on your generated keys
    SSH_HOST_ED25519_KEY: '-----BEGIN OPENSSH PRIVATE KEY-----CHANGEME'
    SSH_HOST_RSA_KEY: '-----BEGIN RSA PRIVATE KEY-----CHANGEME'
```

And to your `.lagoon.env.master`:

```
SSH_HOST_ED25519_KEY="-----BEGIN OPENSSH PRIVATE KEY-----CHANGEME"
SSH_HOST_RSA_KEY="-----BEGIN RSA PRIVATE KEY-----CHANGEME"
```

### Change username and password

Change the Environment variables `SFTP_USER` and `SFTP_PASSWORD` in the Dockerfile.sftp
```
ENV SFTP_USER 'sftpupload'
ENV SFTP_PASSWORD 'thispasswordneedstobechanged'
```


### Build container
After that you should be able to run `docker-compose build sftp` and sucessfully build the SFTP contianer.

### Open Openshift Nodeport
In order to access the container from outside we need to open a port. Best ask your lagoon administrator to get this done for you.

```
apiVersion: v1
kind: Service
metadata:
  name: sftp
spec:
  ports:
    - name: 2222-tcp
      nodePort: XXXXXX
      port: 2222
      protocol: TCP
      targetPort: 2222
  selector:
    service: sftp
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

**Insipration:** This work is inspired by the SFTP container of [atmoz/sftp](https://github.com/atmoz/sftp/). Thanks!


