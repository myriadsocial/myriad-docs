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
There are minimum requirements for a server on which you can run Myriad Social. These requirements will vary based on the version of software you install on your server. The guide will inform you of the minimum and recommended system requirements to run a Myriad Social instance.

You can run Myriad Social on a machine with these specs using Linux Distros such as CentOS/RHEL or Ubuntu.

**Note: This guide assumes that you already have a server available with the necessary hardware to run it. You can use services similar to [DigitalOcean](https://www.digitalocean.com/pricing/droplets) or [Linode](https://www.linode.com/pricing/#compute-shared) if you don't.**

**Myriad Social is also working on lowering the minimum specifications so that in the future, you can run Myriad Social as low-cost as possible and enable more communities to have their own instances.**

#### Minimum System Requirement
Minimum system requirements will allow you to run a basic Myriad Social instance with an optimized configuration on small machines.

The requirements are:
- 2 Core CPU
- 4 GB RAM
- 10 GB Harddisk space

**Note: Per 2023, the average VPS price for a server using these specs is around $20-$30/month. However, the price may vary based on location and provider.**

#### Recommended System Requirement
Alternatively, the recommended system requirements will allow you to run an optimized Myriad Social instance that maximizes performance on most machines.

The requirements are:
- 2 Core CPU
- 8 GB RAM
- 20 GB Harddisk space

**Note: Per 2023, the average VPS price for a server using these specs is around $40-$50/month. However, the price may vary based on location and provider.**

## Getting Started
To host your Myriad Social instance, you will need to install the following packages:
- Git - [Installation Guide](https://git-scm.com/download/linux)
- Docker - [Installation Guide](https://docs.docker.com/engine/install/ubuntu/)

To avoid complications during installations, the Myriad Social team Dockerized the build. Mitigating the prospects of having version incompatibilities with the system you are using. 

Even though the team recommends you use Linux Distros (either RHEL-based or Debian-based), as long as your system can support the modern Docker container runtime environment, you are good to go.

TL;DR the installation steps are as follows:
- Decide on the domain for your instance.
- Provision a VPS (if you don't have one already or have a bare metal server in mind).
- Direct your domain to the VPS's IP via A records.
- Download the most recent Myriad Social source.
- Configure your environment variables.
- Configure the directory permissions.
- Execute `docker compose` to run your Myriad Social instance.

## Installation Guide
If you are installing on a Linux machine, make sure you have root access. Because you will need it to install the required packages, modify directory permissions, create a user and group for Docker containers to run under, and create systemd services.

###  Downloading the Source from GitHub
Myriad Social does not yet have a particular Docker image, so you must download and run everything from the source. 

```bash
git clone https://github.com/myriadsocial/myriad-api.git && cd myriad-api
```

### Prepare Environment File
The environment variables are the only configurations you will need to set for the app. Here, you will configure the domain of your instance, the escrow mnemonics, the database configuration, and the SMTP server credentials.

First, create .env file from a template

```bash
cp ./.maintain/deployment/.env-template ./.env
```

Open .env file

```bash
vim ./.env
```

Next, press `i` to start editing the file according to your needs and add the variables as follows:

```
API_VERSION=*set to `latest` or choose version from https://github.com/myriadsocial/myriad-api/releases without `v`*
DOMAIN=*the domain you are using for your instance*

# Wallet
MYRIAD_ADMIN_SUBSTRATE_MNEMONIC=*mnemonic, when you cretae new wallet in any wallet provider, you will get this mnemonic*
MYRIAD_ADMIN_NEAR_MNEMONIC=*mnemonic, when you cretae new wallet in any wallet provider, you will get this mnemonic*

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

Press `esc` to finish editing, and then save the file

```
`:wq`
```

### Run Service
Once you are done editing the environment variables, you are ready to run the instance. You can run the instance using Docker Compose:

```bash
sudo docker compose -p myriad -f ./.maintain/deployment/docker-compose.yaml --env-file ./.env --profile webserver up -d
```

After running the script above, verify if everything is running by executing `docker ps`. If there are no errors, continue modifying the storage directory owner.

```bash
sudo chown -R 1001 ./.local/storages
```

Next, run the database migrations.

```bash
sudo docker compose -p myriad -f ./.maintain/deployment/docker-compose.yaml --env-file ./.env run --rm db_migration --rebuild --environment mainnet
```

**Note: Please note that you will run the Myriad Social instance in the Mainnet, not Testnet.**

Finally, set up NGINX and redirect traffic to your Myriad Social instance. Execute the following script:

```bash
sudo ./.maintain/deployment/init-webserver.sh
```

If there is an error when initializing the webserver, delete the webserver folder:

```bash
sudo rm -rf ./.local/certbot ./.local/nginx
```

Re-run the webserver initialization:

```bash
sudo ./.maintain/deployment/init-webserver.sh
```

*And you are done!*

If you are running into trouble, please [create a new Issue](https://github.com/myriadsocial/myriad-api/issues/new) in the Myriad API repository to receive support.