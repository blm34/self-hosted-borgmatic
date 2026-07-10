# Borgmatic

Docker compose configuration for running Borgmatic on ubuntu server

## Set Up

Clone this repository

```bash
git clone git@github.com:blm34/rpi-server-borgmatic.git
```

Copy the file `.env.template` to `.env` and fill in the variables

```bash
cp .env.template .env
```

Start the container with

```bash
docker compose up -d
```

The repository must be initialised inside the container. For the two
repositories located at `/mnt/borg-repository/borg-backup/home` and
`/mnt/borg-repository/borg-backup/data`, directories must be created and then
initialised as Borgmatic repositories.

```bash
docker exec -it borgmatic /bin/bash

# Create the directories to backup into
cd /mnt/borg-repository
mkdir -p borg-backup/home
mkdir borg-backup/data

# Initialise the repositories
borgmatic repo-create --encryption repokey-blake2

# Exit the container's shell
```

## Export the Repokeys

To export the repokeys, first they must be exported in the container, then
copied to outside the container, and finally copied to a different machine.
These steps must be repeated for every repo.

```bash
# Export the key in the container
docker exec borgmatic key export /mnt/borg-repository/path/to/repo /tmp/reponame-repokey

# Copy the key outside the container
docker cp borgmatic:/tmp/reponame-repokey /tmp/reponame-repokey
```

Exit SSH and use scp to copy the key to the local device

```bash
scp user@device:/tmp/reponame-repokey ./
```

## Restore a Backup

To do...
