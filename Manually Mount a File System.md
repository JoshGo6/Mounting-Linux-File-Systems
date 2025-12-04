# Manually Mount a File System

Mounting is the process of attaching a file system--such as ext4, NTFS, VFAT, a network drive, or various others--to the existing Linux directory tree. This topic addresses manually mounting a Linux file system. (For information on automounting a file system, as well as controlling who can mount and unmount a file system, see [Automatically Mount Partitions Using `fstab`](Automatically%20Mount%20Partitions%20Using%20`fstab`.md), though this file may not be visible on GitHub.)

Mounting a file system that is native to Linux, like ext4, is simpler than mounting other file systems, though the additional complexity for handling other file system is not always that substantial. For example, mounting an NTFS file system is slightly more complex than mounting ext4, while mounting a network drive is yet more complex.

> If you're viewing this article on GitHub, the hyperlinks to sections within this document, as well as hyperlinks to other documents I've written typically won't work, as I maintain this repository locally using [Obsidian](https://obsidian.md/), which uses its own Markdown variant. Hyperlinks aren't the only variation between Obsidian and GitHub Flavored Markdown (GFM). 

This article contains the following topics:


- [Mount a drive native to Linux](#Mount%20a%20drive%20native%20to%20Linux)
- [Mount an NTFS drive](#Mount%20an%20NTFS%20drive)
- [Mount a network drive using `sshfs`](#Mount%20a%20network%20drive%20using%20`sshfs`)
- [Manually mount a network drive using `cifs`](#Manually%20mount%20a%20network%20drive%20using%20`cifs`)
- [Manually mount a drive configured in `/etc/fstab`](#Manually%20mount%20a%20drive%20configured%20in%20`/etc/fstab`)
- [Bind mount directories](#Bind%20mount%20directories)
- [Specify permissions](#Specify%20permissions)
  - [Native Linux file systems (`ext4`, `xfs`, and `btrfs`)](#Native%20Linux%20file%20systems%20(`ext4`,%20`xfs`,%20and%20`btrfs`))
  - [File systems not native to Linux (`FAT32`, `NTFS`, and `exFAT`)](#File%20systems%20not%20native%20to%20Linux%20(`FAT32`,%20`NTFS`,%20and%20`exFAT`))
  - [Remote file system mounted through `sshfs`](#Remote%20file%20system%20mounted%20through%20`sshfs`)

## Mount a drive native to Linux

To mount a drive native to Linux, do the following:

1. Create a directory that serves as a mount point:  
   
    ```bash
    mkdir /mnt/additional-data
    ```

1. Find the device ID for the drive:  

    ```bash
	lsblk -f | grep -v loop
	```

    The following is an excerpt from typical output:

    ```bash
     sdd
     └─sdd1      ext4     1.0         4c75d6a8-9d24-4d57-bf72-3b5918e8e01d    1.6T     6%
    ```

4. Mount the drive to the mount point.

    ```bash
    sudo mount -t ext4 /dev/sdd1 /mnt/additional-data
    ```
 
## Mount an NTFS drive

To mount an NTFS drive, do the following:

1. Create a directory that serves as a mount point:

    ```bash
    mkdir /mnt/additional-data
    ```

2. If you haven't done so already, install `ntfs-3g`

    ```bash
    sudo apt update
    sudo apt install ntfs-3g
    ```

3. Find the device ID for the drive:

    ```bash
    lsblk -f | grep -v loop
    ```

    The following is an excerpt from typical output:

    ```bash
    ├─nvme0n1p3 ntfs                 AAFC12D5FC129C1F                       29.2G    62% /mnt/windows
    ```

4. Mount the drive:

    ```bash
     sudo mount -t ntfs-3g /dev/nvme0n1p3 /mnt/additional-data
    ```

To specify permissions when mounting an NTFS drive, see [Specify permissions](#Specify%20permissions).

## Mount a network drive using `sshfs`

A remote drive may support one protocol but not another protcol for remotely mounting the drive. One protocol that is widely allowed by file systems when remotely mounting them is `sshfs`. The following commands show a typical flow to mount a remote drive using `sshfs`:

1. Create a mount directory, `/mnt/mywebsite`.

2.  Install `sshfs` if you haven't done so, already:
    
    ```bash
    sudo apt install sshfs
    ```
    
2. If you haven't done so already, copy your public ssh key of your local machine to the remote machine, which authorizes your local machine to connect to the remote machine. You can do this either by using the UI of the remote file system, or by using the `scp` command.

3.  Mount the remote drive on `/mnt/mywebsite`. The following is a typical mount command:

    ```bash
    sudo sshfs -p 2222 -o allow_other,default_permissions,uid=$(id -u),gid=$(id -g),umask=0022,IdentityFile=/home/brad/.ssh/id_ed25519 bradm@69.176.58.100:/home/bradm /mnt/mywebsite/
    ```

    This command mounts the remote directory using `sshfs`, with the following configurations:

    - The port is `2222`.
    - The permissions of the user on the target machine are mapped to the UID and GID of the user executing the `mount` command, with permissions altered by a `umask` of `022`, giving all the permissions of the remote user to the user performing the mount locally, but not giving write privileges to anyone other local user who can access the mounted drive.
    - The ssh file is specified, which may create a login prompt for the passphrase, even if the passphrase is kept in a key ring  mechanism.

     For more information on permissions, see [Specify permissions](#Specify%20permissions).

4. When you are done with the mounted drive, unmount it:

    ```bash
    fusermount -u /mnt/mywebsite
    ```


## Manually mount a network drive using `cifs`

One simple way to mount a network volume is to use `cifs`. Mounting the network volume using `cifs` needs to be supported by the remote volume, however. If the remote volume doesn't support `cifs`, use a different method, like `sshfs`. For more details, see [Mount a network drive using `sshfs`](#Mount%20a%20network%20drive%20using%20`sshfs`).

The following is the syntax used to mount a remote file system using `cifs`:

```bash
sudo mount -t cifs <server_address>/path/to/directory <mount_point>
```

The `<shared-directory>` is the path on the server to the shared directory--for example, `/home/joshg`.

As an example, if the remote server supports `cifs`, you could mount a network drive using the following syntax, replacing `<password>` with your password:

```bash
sudo mount -t cifs //60.178.52.156/home/joshgo6 /mnt/mywebsite/ -o 'username=joshgo6,password=<password>'
```

## Manually mount a drive configured in `/etc/fstab`

So far, we've covered mounting drives that aren't configured in `/etc/fstab`. That file specifies how drives are mounted. In many cases, `/etc/fstab` configures your system to auto-mount drives for you so that you don't have to mount them manually, but in some cases, you still have to give the `mount` command to mount these volumes. Examples of this include the following:

- Volumes that are configured in `/etc/fstab` to be mountable by users, but the user has to issue the mount command, since the drive isn't mounted automatically.
- Volumes that are mounted automatically but have since been unmounted manually.
- Remote volumes that require the user to first enter an `ssh` passphrase.

In these situations, mount the drive using the following syntax:

```bash
mount /path/to/mount/point
```

As an example, suppose my `/etc/fstab` allows me to mount the remote directory that contains my website, and this volume is specified to be mounted locally at `/mnt/mywebsite`.  The system cannot mount this drive until I enter my `ssh` passphrase. After I do this, I can manually mount the drive:

```bash
mount /mnt/mywebsite
```

There are at least three other alternatives to this approach for a volume that requires `ssh` in order to mount:

- Have `root` mount the drive using its own `ssh` key that doesn't have a passphrase. Make the key readable only by `root` by setting its permissions to `600`.
- Put a line in `.bashrc` right after the `ssh` key has been entered for the system to wait a few seconds for the key to load, and then for the system to mount the drive.
- Refresh auto-mounts in `/etc/fstab` after entering your passphrase by running `mount -a`.

## Bind mount directories

So far we've spoken about mounting block devices and remote file systems. You can also mount a directory temporarily under another directory, using a *bind mount*. When you create a bind mount, you are showing an existing directory and all of its subdirectories, recursively, at another point in the directory tree, with the following properties:

- The original contents of the mounting point are masked. They are not deleted, but are temporarily unavailable.
- The source directory and the bind mount directory are identical. They show the same files and subdirectories, and changes to one result in automatic changes to the other.
- By default, any mount points that are subdirectories under the source directory are not mounted at the bind mount point. You can change this behavior, however, by using the `--rbind` option instead of the `--bind` option.

The following examples demonstrate bind mount syntax:

```bash
# Bind mount /source/dir to /mnt/destination
mount --bind /source/dir /mnt/destination

# Recursively bind mount /source/dir and all its mount points 
mount --rbind /source/dir /mnt/destination

# Unmount a recursively bind mounted directory structure
umount -R /mnt/destination
```

> **NOTE:** Because bind mounting by default doesn't mount any mount points *below* the bind mount point, if there were subdirectories that were masked in the source because they were mount points, once the source directory is bind mounted, the target directory shows the original, unmasked contents of the source directories. In other words, the target shows the contents of the subdirectories of the source before they were masked by (previous) mounting operations.

## Specify permissions

When you mount a file system to a mount point, the original permissions of the mount point become irrelevant, and they're replaced by the permissions of the root directory of the file system you're mounting.

> If the file system is specified in `/etc/fstab`, then the file system will either be auto-mounted, or permissions will be set for users to mount it. If neither of these conditions is true, you need `sudo` privileges to mount the file system.

The following sections provide guidelines for setting permissions for several types of mounted file systems:

### Native Linux file systems (`ext4`, `xfs`, and `btrfs`)

If you need to change the file permissions of the directories and files that are mounted, change these using `chown` and `chmod`. These changes persist after being unmounted.

To mount a local, native Linux file system without changing permissions:

```bash
mkdir /mnt/shared
sudo mount /dev/sdc1  /mnt/shared
```

To mount a local, native Linux file system and change permissions so that you're the owner, with full permissions, but others only have read/execute permissions on directories, and read permissions on files:

```bash
mkdir /mnt/shared
sudo mount /dev/sdc1  /mnt/shared
sudo chown -R $(id -u):$(id -g) /mnt/shared
sudo chmod -R 755 /mnt/shared
sudo find /mnt/shared -type f -exec chmod 644 {} \;
```

### File systems not native to Linux (`FAT32`, `NTFS`, and `exFAT`)

To mount file systems not native to Linux, you have numerous options to set permissions. To mount an `ntfs` drive so that everyone has read only permissions:

```bash
# Mount with read only permissions
sudo mount -o ro -t ntfs-3g /dev/sdb /mnt/ntfs
```

Another option is to give everyone read/write permissions: 

```bash
# Mount with read/write permissions
sudo mount -o rw -t ntfs-3g /dev/sdb /mnt/ntfs
```

Yet another option is specifying permissions for you, the user that mounts the file system, that are different from other users. Use the following command to give yourself full access, and everyone else only read access:

```bash
# Mount with you being the owner and group
sudo mount -o uid=$(id -u),gid=$(id -g),umask=022 -t ntfs-3g /dev/sdb /mnt/ntfs
```

> NTFS doesn't use Linux permissions. You create these permissions during the mount operation, and they exist only while the file system is mounted. When you specify the UID and GID as yours, you become the owner and primary group of the directories and files while they're mounted. The mounting operation gives everyone `rwx` permissions on all directories and `rw` permissions on all files. By also specifying a `umask` of `022`, the resulting permissions become `777` - `022` = `755` for directories, and `666` - `022` = `644` for files, where you are the owner, and the group is your primary group.  A less compact alternative is to specify `dmask`, which subtracts permissions on directories, and `fmask`, which subtracts permissions on files:
>  
> ```bash
>  sudo mount -o uid=$(id -u),gid=$(id -g),dmask=022,fmask=022 -t ntfs-3g /dev/sdb /mnt/ntfs
> ```

The `FAT32` and `exFAT` file systems behave differently. The internal logic that Linux uses to synthetically apply permissions is complicated. When mounting these file systems, Linux never gives execute permissions to files, because FAT-based file systems do not support the execute bit. Because of this, we use `fmask=133` to produce `644` permissions on files. Why this works is beyond the scope of this article.

To mount `FAT32` so that you have full permissions, and everyone else has read-only permissions:

```bash
# Mount directories with 755, and files with 644
sudo mount -t vfat /dev/sdb1 /mnt/fat \
  -o uid=$(id -u),gid=$(id -g),dmask=022,fmask=133
```

Similarly, to mount `exFAT` so that you have full permissions, and everyone else has read-only permissions:

```bash
# Mount directories with 755, and files with 644
sudo mount -t exfat /dev/sdb1 /mnt/exfat \
  -o uid=$(id -u),gid=$(id -g),dmask=022,fmask=133
```

### Remote file system mounted through `sshfs`

When you mount a remote drive using `sshfs`, specifying permissions at the mount point may be different than specifying permissions on the remote server. For example, if you mount a native Linux file system like `ext4` and you issue `chown` and/or `chmod` commands (successfully), you change the permissions on the remote server, and these persist after unmounting. On the other hand, you could mount the same remote server using `idmap` and/or using `gid`, `uid`, and `umask`, which controls permissions at the mount point without changing any ownership or permissions on the remote server.

When mounting non-native Linux drives and trying to use `chmod` and `chown` on the files and directories on the remote server, the outcome varies from file system to file system and sometimes can be unpredictable. This is because non-native Linux file systems don't have Linux permissions in their files and directories. How non-native Linux file systems respond to `chmod` and `chown` commands when mounted via `sshfs` is beyond the scope of this article.

The approach we advocate and use in this article is mounting non-native Linux file systems with `gid`, `uid`, and `umask` options (or `fmask` and `dmask` options instead of `umask`). This creates artificial Linux permissions at the mount point that survive only until you unmount the drive. As a simpler alternative to specifying the UID and the GID, you can use the `idmap=user` option, which maps the owner and group to the local mounter. Optionally specify a `umask` to ensure that other users don't have more privileges than you want them to have.

The following is a typical command to manually mount a remote file system using `sshfs`, where the last argument is the local mount point:

```bash
sudo sshfs -p 2222 -o allow_other,default_permissions,uid=$(id -u),gid=$(id -g),umask=0022,IdentityFile=/home/josh/.ssh/id_ed25519 joshgo6@69.174.52.100:/home/joshgo6 /mnt/mywebsite/
```

The follow are some useful options when using `sshfs`, some of which are used in the previous command:

- **`allow_other`**: Allows all local users to access the SSHFS mount; without this option, only the user who ran `sshfs` may access it.
- **`default_permissions`**: Enforces standard local POSIX permission checks using the synthetic permissions created by SSHFS, preventing local users from bypassing access restrictions.
- **`p`**: The port to connect to on the remote server.
- **`uid=`**: Forces all files in the mount to appear owned locally by the specified UID.
- **`gid=`**: Forces all files in the mount to appear in the specified local group.
- **`idmap=user`**: Maps all remote file ownership to the UID/GID of the mounting user unless overridden by `uid=` or `gid=`; affects only local representation.
- **`reconnect`**: Automatically attempts to re-establish the SSH connection if it is interrupted.
- **`ServerAliveInterval=`**: Sends a keepalive packet to the remote server every N seconds to maintain the SSH connection.
- **`ServerAliveCountMax=`**: Sets how many unanswered keepalive packets can occur before the connection is considered dead.
- **`ConnectTimeout=`**: Specifies the maximum number of seconds to wait for the SSH connection to be established before timing out.
- **`IdentityFile=`**: Provides the path to the private key used for authentication.
- **`follow_symlinks`**: Resolves symlinks on the remote server instead of locally.
- **`cache_timeout=`**: Sets how long SSHFS caches file attributes before rechecking them on the remote server.
- **`no_readahead`**: Disables read-ahead buffering to improve responsiveness on slow or unreliable networks.
