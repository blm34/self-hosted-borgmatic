# Borgmatic

Docker compose configuration for running Borgmatic on ubuntu server

## Requirements

* Docker
* Docker compose

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

To restore a backup, first start the backup container and enter a shell within it

```bash
docker compose -f compose-restore.yaml run --rm --entrypoint /bin/bash borg-restore
```

List the available archives

```bash
borg list <path-to-archive>
```

There are two options for restoring an archive: mounting or extracting. Mounting
allows specific files or directories to be extracted from an archive without extracting
the whole thing. Extracting restores the whole archive in one go, giving immediate
access to every file.

### Mount an Archive

Whilst inside the container, create a mount point

```bash
mkdir /restore/mount
```

Now mount the archive

```bash
borg mount <repository-path>::<archive-name> /restore/mount
```

Browse the archive to find the files to restore

```bash
ls /restore/mount/mnt
```

Copy any files out of the archive into the restore folder to extract them

```bash
cp -a /restore/mount/mnt/... /restore/
```

Unmount the archive when done

```bash
borg umount /restore/mount
```

Exit the container. The files/directories that were copies will be available in the
mapped /tmp/borg-restore/ directory

### Extract an Archive

Whilst inside the container restore the entire archive with

```bash
cd /restore

borg extract <repository-path>::<archive-name>
```

Exit the container, and find the extracted archive in /tmp/borg-restore/
