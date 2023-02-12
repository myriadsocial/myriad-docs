# Run Your Own Server Guide
Myriad Social is a federated social network that allows users to connect with friends and colleagues from other networks. In this guide, you will walk through the process of setting up your own Myriad Social server. 

## Prerequisites
There are a few prerequisites you need to fulfill before you can install and run your server:
- A server, either a Virtual Private Server (VPS) or bare metal server (e.g., Raspberry Pis).
- An email or an SMTP service.
- A domain or subdomain on which you can host your Myriad Social instance.
- A Polkadot Wallet mnemonic for the purpose of payment escrows.
- A NEAR Wallet mnemonic for the purpose of payment escrows.

### VPS or Bare Metal Server Requirements
There are minimum requirements for a server on which you can run Myriad Social. These requirements will vary based on the version of software you install on your server. The guide will inform you of the minimum and recommended system requirements to run a Myriad Social node.

You can run Myriad Social on a machine with these specs using Linux Distros such as CentOS/RHEL or Ubuntu.

**Note: This guide assumes that you already have a server available with the necessary hardware to run it. You can use services similar to [DigitalOcean](https://www.digitalocean.com/pricing/droplets) or [Linode](https://www.linode.com/pricing/#compute-shared) if you don't.**

**Myriad Social is also working on lowering the minimum specifications so that in the future, you can run Myriad Social as low-cost as possible and enable more communities to have their own nodes.**

#### Minimum System Requirement
Minimum system requirements will allow you to run a basic Myriad Social node with an optimized configuration on small machines.

The requirements are:
- 2 Core CPU
- 4 GB RAM
- 10 GB Harddisk space

**Note: Per 2023, the average VPS price for a server using these specs is around $20-$30/month. However, the price may vary based on location and provider.**

#### Recommended System Requirement
Alternatively, the recommended system requirements will allow you to run an optimized Myriad Social node that maximizes performance on most machines.

The requirements are:
- 2 Core CPU
- 8 GB RAM
- 20 GB Harddisk space

**Note: Per 2023, the average VPS price for a server using these specs is around $40-$50/month. However, the price may vary based on location and provider.**

## Getting Started
To host your Myriad Social node, you will need to install the following packages:
- Git
- Docker
- Docker Compose

To avoid complications during installations, the Myriad Social team Dockerized the build. Mitigating the prospects of having version incompatibilities with the system you are using. 

Even though the team recommends you use Linux Distros (either RHEL-based or Debian-based), as long as your system can support the modern Docker container runtime environment, you are good to go.

TL;DR the installation steps are as follows:
- Decide on the domain for your node.
- Provision a VPS (if you don't have one already or have a bare metal server in mind).
- Direct your domain to the VPS's IP via A records.
- Install the software prerequisites.
- Download the most recent Myriad Social source.
- Configure your environment variables.
- Configure the directory permissions.
- Execute docker-compose to run your Myriad Social node.

## Installation Guide
If you are installing on a Linux machine, make sure you have root access. Because you will need it to install the required packages, modify directory permissions, create a user and group for Docker containers to run under, and create systemd services.

**Note: Ensure you already have your VPS or Bare Metal server running and point your domain's [A records to your](https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/) server's IP.**

**The Myriad Social development team recommends that you use [Cloudflare](https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/) as your DNS proxy to increase your node security (the [basic free plan should suffice](https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/)), as a majority of distributed Denial of Service attacks originate from malicious DNS requests.**

### Login as Root
Start by logging in to your server as root. Either ssh directly as root or type sudo su in your terminal to gain root privileges.

```
sudo su
```

### Installing Git
To proceed, you will need to download the entirety of Myriad Social's source code from GitHub. If your machine doesn't have Git already, install it by running this in your terminal.

```
apt install git-all
```

**Note: this command assumes you are using a Debian-based distro, hence why it uses apt. If you are using a different Distro, please [read more here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).**

### Installing Docker
You can read a more [defined tutorial](https://docs.docker.com/engine/install/ubuntu/) about Docker and how to install it here. But to save you time, we have included a quick installation script for Linux (Debian) users to follow below.

First, update your Distro and get the required packages for Docker.

```
apt-get update
apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Add Docker's official GPG key.

```
mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Setup Docker's official package repository.

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

After installing the repository, update the package manager to propagate the changes.

```
apt-get update
```

Finally, install the Docker engine along with Docker Compose.

```
apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Preparing Base Folder
Make a base directory for Myriad Social. You can do it anywhere as long as you have sufficient permissions to create and modify the directory. In this specific tutorial, however, you are going to prepare the folder within the root directory named myriad:

```
mkdir /myriad && cd /myriad
```

###  Downloading the Source from GitHub
Myriad Social does not yet have a particular Docker image, so you must download and run everything from the source. 

```
git clone https://github.com/myriadsocial/myriad-api.git && cd myriad-api
```

### Prepare Environment File
The environment variables are the only configurations you will need to set for the app. Here, you will configure the domain of your node, the escrow mnemonics, the database configuration, and the SMTP server credentials.

First, copy all the .env contents:

```
cp ./.maintain/deployment/.env-template ./.env
```

Next, edit the file according to your needs and add the variables as follows:

```
API_VERSION=*get the latest Digest hash*
DOMAIN=*the domain you are using for your node

# Wallet
MYRIAD_ADMIN_SUBSTRATE_MNEMONIC=*mnemonic*
MYRIAD_ADMIN_NEAR_MNEMONIC=*mnemonic*

# JWT
JWT_TOKEN_SECRET_KEY=*write a 16 character random string key*
JWT_REFRESH_TOKEN_SECRET_KEY=*write a 16 character random string key*

# MONGO
MONGO_USER=*write some user name*
MONGO_PASSWORD=*write some password

# REDIS
REDIS_PASSWORD=*write some password*

# SMTP
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=465
SMTP_USERNAME=*your gmail*
SMTP_PASSWORD=*your gmail password*
```

For the API_VERSION parameter, you can retrieve the latest digest directly from the [Docker hub](https://hub.docker.com/r/myriadsocial/myriad-api/tags). And for the SMTP parameters, you can use Gmail as a default.

### Run Service
Once you are done editing the environment variables, you are ready to run the node. You can run the node using Docker Compose:

```
docker compose -p myriad -f ./.maintain/deployment/docker-compose.yml --env-file ./.env --profile webserver up -d
```

After running the script above, verify if everything is running by executing `docker ps`. 

To automate the node when the server reboots, create a `systemd` service:

```
[Unit]
Description=Myriad Social node with Docker Compose
PartOf=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=true
WorkingDirectory=/myriad/myriad-api
ExecStart=/*which docker* compose up -d --remove-orphans
ExecStop=/*which docker* compose down

[Install]
WantedBy=multi-user.target
```

Name the file `myriadnode.service`, and then execute:

```
systemctl start myriadnode.service
```

Check if everything is running with the following:

```
systemctl status myriadnode.service
```

If there are no errors, continue modifying the storage directory owner.

```
chown -R 1001 ./.local/storages
```

Next, run the database migrations.

```
docker compose -p myriad -f ./.maintain/deployment/docker-compose.yml --env-file ./.env run --rm db_migration --rebuild --environment mainnet
```

**Note: Please note that you will run the Myriad Social node in the Mainnet, not Testnet.**

Finally, set up NGINX and redirect traffic to your Myriad Social node. Execute the following script:

```
./.maintain/deployment/init-webserver.sh
```

If there is an error when initializing the webserver, delete the webserver folder:

```
rm -rf ./.local/certbot ./.local/nginx
```

Re-run the webserver initialization:

```
./.maintain/deployment/init-webserver.sh
```

*And you are done!*

If you are running into trouble, please [create a new Issue](https://github.com/myriadsocial/myriad-api/issues/new) in the Myriad API repository to receive support.