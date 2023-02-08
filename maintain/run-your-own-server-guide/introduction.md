# Run Your Own Server Guide

> A simple step using prebuilt docker image.

## Prerequisite
- DNS (Domain Name Server)
- VPS (Virtual Private Server)
- Email
- Polkadot Wallet
- Near Wallet

## Preparing VPS
- Minimum System Requirement
    - 2 Core CPU
    - 8 GB Memory
    - 20 GB Virtual Disk
- Install Docker Engine and Docker Compose, check installtion setup on [Docker Official Website](https://docs.docker.com/engine/install/debian)
- Register VPS public IP address to your DNS

## Install
### Login as Root
```bash
sudo su
```
### Preparing base folder
```bash
mkdir /myriad && cd /myriad
```
### Checking out the code
```bash
git clone https://github.com/myriadsocial/myriad-api.git
```
### Entering main folder
```bash
cd myriad-api
```
### Prepare environment file
```bash
cp ./.maintain/deployment/.env-template ./.env
```
### Configure environment file
```bash
vim .env
```
### Run service
```bash
docker compose -p myriad -f ./.maintain/deployment/docker-compose.yml --env-file ./.env --profile webserver up -d
```
### Change storage permission
```bash
chown -R 1001 ./.local/storages
```
### Run database migration
```bash
docker compose -p myriad -f ./.maintain/deployment/docker-compose.yml --env-file ./.env run --rm db_migration --rebuild --environment mainnet
```
### Setup webserver
```bash
./.maintain/deployment/init-webserver.sh
```