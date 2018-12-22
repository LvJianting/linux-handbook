# Introduction to Linux Part 2

## Files

> "On a UNIX system, everything is a file; if something is not a file, it is a process."

### Sort of files

1. Directories: files that are lists of other files.

2. Special files: the mechanism used for input and output. Most special files are in /dev, we will discuss them later.

3. Links: a system to make a file or directory visible in multiple parts of the system's file tree. We will talk about links in detail.

4. (Domain) sockets: a special file type, similar to TCP/IP sockets, providing inter-process networking protected by the file system's access control.

5. Named pipes: act more or less like sockets and form a way for processes to communicate with each other, without using network socket semantics.

#### File types in a long list

```bash
jaime:~/Documents> ls -l
total 80
-rw-rw-r--   1 jaime   jaime   31744 Feb 21 17:56 intro Linux.doc
-rw-rw-r--   1 jaime   jaime   41472 Feb 21 17:56 Linux.doc
drwxrwxr-x   2 jaime   jaime    4096 Feb 25 11:50 course
```

| Symbol | Meaning |
| ------ | ------- |
| - | Regular file |
| d | Directory |
| l | Link |
| c | Special file |
| s | Socket |
| p | Named pipe |
| b | Block device |

#### Color-ls default color scheme

| Color | File type |
| ----- | --------- |
| blue | directories |
| red | compressed archives |
| white |text files |
| pink | images |
| cyan | links |
| yellow | devices |
| green | executables |
| flashing red | broken links |

#### Default suffix scheme for ls

| Character | File type |
| --------- | --------- |
| nothing | regular file |
| / | directory |
| * | executable file |
| @ | link |
| = | socket |
| | | named pipe |

### Partition

#### Why partition

One of the goals of having different partitions is to achieve higher data security in case of disaster. By dividing the hard disk in partitions, data can be grouped and separated. When an accident occurs, only the data in the partition that got the hit will be damaged, while the data on the other partitions will most likely survive.  

A simple example: a user creates a script, a program or a web application that starts filling up the disk. If the disk contains only one big partition, the entire system will stop functioning if the disk is full. If the user stores the data on a separate partition, then only that (data) partition will be affected, while the system partitions and possible other data partitions keep functioning.

#### Partition layout and types

There are two kinds of major partitions on a Linux system:

* data partition: normal Linux system data, including the root partition containing all the data to start up and run the system;

* swap partition: expansion of the computer's physical memory, extra memory on hard disk.

Most systems contain a root partition, one or more data partitions and one or more swap partitions.

Linux generally counts on having twice the amount of physical memory in the form of swap space on the hard disk. When installing a system, you have to know how you are going to do this. An example on a system with 512 MB of RAM:

* 1st possibility: one swap partition of 1 GB

* 2nd possibility: two swap partitions of 512 MB

* 3rd possibility: with two hard disks: 1 partition of 512 MB on each disk.

When non-critical data is separated on different partitions, it usually happens following a set pattern:

* a partition for user programs (/usr)

* a partition containing the users' personal data (/home)

* a partition to store temporary data like print- and mail-queues (/var)

* a partition for third party and extra software (/opt)

**Once the partitions are made, you can only add more.** Changing sizes or properties of existing partitions is possible but not advisable.

On a server, system data tends to be separate from user data. Programs that offer services are kept in a different place than the data handled by this service. Different partitions will be created on such systems:

* a partition with all data necessary to boot the machine

* a partition with configuration data and server programs

* one or more partitions containing the server data such as database tables, user mails, an ftp archive etc.

* a partition with user programs and applications

* one or more partitions for the user specific files (home directories)

* one or more swap partitions (virtual memory)

#### Mount points

All partitions are attached to the system via a mount point. The mount point defines the place of a particular data set in the file system. Usually, all partitions are connected through the root partition.

During system startup, all the partitions are thus mounted, as described in the file /etc/fstab.

On a running system, information about the partitions and their mount points can be displayed using the df command (which stands for disk full or disk free)

```bash
freddy:~> df -h
Filesystem          Size  Used Avail Use% Mounted on
/dev/hda8           496M  183M  288M  39% /
/dev/hda1           124M  8.4M  109M   8% /boot
/dev/hda5            19G   15G  2.7G  85% /opt
/dev/hda6           7.0G  5.4G  1.2G  81% /usr
/dev/hda7           3.7G  2.7G  867M  77% /var
fs1:/home           8.9G  3.7G  4.7G  44% /.automount/fs1/root/home
```

#### Tree structure

![Linux file system layout](./FS-layout.png)

| Directory | Content |
| --------- | ------- |
| /bin | Common programs, shared by the system, the system administrator and the users. |
| /boot | The startup files and the kernel, vmlinuz. In some recent distributions also grub data. Grub is the GRand Unified Boot loader and is an attempt to get rid of the many different boot-loaders we know today. |
| /dev | Contains references to all the CPU peripheral hardware, which are represented as files with special properties. |
| /etc | Most important system configuration files are in /etc, this directory contains data similar to those in the Control Panel in Windows |
| /home | Home directories of the common users. |
| /initrd | (on some distributions) Information for booting. Do not remove! |
| /lib | Library files, includes files for all kinds of programs needed by the system and the users. |
| /lost+found | Every partition has a lost+found in its upper directory. Files that were saved during failures are here. |
| /misc | For miscellaneous purposes. |
| /mnt | Standard mount point for external file systems, e.g. a CD-ROM or a digital camera. |
| /net | Standard mount point for entire remote file systems |
| /opt | Typically contains extra and third party software. |
| /proc | A virtual file system containing information about system resources. More information about the meaning of the files in proc is obtained by entering the command man proc in a terminal window. The file proc.txt discusses the virtual file system in detail. |
| /root | The administrative user's home directory. Mind the difference between /, the root directory and /root, the home directory of the root user. |
| /sbin | Programs for use by the system and the system administrator. |
| /tmp | Temporary space for use by the system, cleaned upon reboot, so don't use this for saving any work! |
| /usr | Programs, libraries, documentation etc. for all user-related programs. |
| /var | Storage for all variable files and temporary files created by users, such as log files, the mail queue, the print spooler area, space for temporary storage of files downloaded from the Internet, or to keep an image of a CD before burning it. |

**Read more in man hier.**

#### The file system in reality

For most users and for most common system administration tasks, it is enough to accept that files and directories are ordered in a tree-like structure. The computer, however, doesn't understand a thing about trees or tree-structures.

Every partition has its own file system. By imagining all those file systems together, we can form an idea of the tree-structure of the entire system, but it is not as simple as that. In a file system, a file is represented by an inode, a kind of serial number containing information about the actual data that makes up the file: to whom this file belongs, and where is it located on the hard disk.

Every partition has its own set of inodes; throughout a system with multiple partitions, files with the same inode number can exist.

Each inode describes a data structure on the hard disk, storing the properties of a file, including the physical location of the file data. When a hard disk is initialized to accept data storage, usually during the initial system installation process or when adding extra disks to an existing system, a fixed number of inodes per partition is created. This number will be the maximum amount of files, of all types (including directories, special files, links etc.) that can exist at the same time on the partition. We typically count on having 1 inode per 2 to 8 kilobytes of storage.

At the time a new file is created, it gets a free inode. In that inode is the following information:

* Owner and group owner of the file.

* File type (regular, directory, ...)

* Permissions on the file

* Date and time of creation, last read and change.

* Date and time this information has been changed in the inode.

* Number of links to this file.

* File size

* An address defining the actual location of the file data.

The only information not included in an inode, is the file name and directory. These are stored in the special directory files. By comparing file names and inode numbers, the system can make up a tree-structure that the user understands. Users can display inode numbers using the -i option to ls. The inodes have their own separate space on the disk.

### The most important configuration files

#### Most common configuration files

| File | Information/service |
| ---- | ------------------- |
| bashrc | The system-wide configuration file for the Bourne Again SHell. Defines functions and aliases for all users. Other shells may have their own system-wide config files, like cshrc. |
| crontab and the cron.* directories | Configuration of tasks that need to be executed periodically - backups, updates of the system databases, cleaning of the system, rotating logs etc. |
| default | Default options for certain commands, such as useradd. |
| filesystems | Known file systems: ext3, vfat, iso9660 etc. |
| fstab | Lists partitions and their mount points. |
| ftp* | Configuration of the ftp-server: who can connect, what parts of the system are accessible etc. |
| group | Configuration file for user groups. Use the shadow utilities groupadd, groupmod and groupdel to edit this file. Edit manually only if you really know what you are doing. |
| hosts | A list of machines that can be contacted using the network, but without the need for a domain name service. This has nothing to do with the system's network configuration, which is done in /etc/sysconfig. |
| inittab | Information for booting: mode, number of text consoles etc. |
| issue | Information about the distribution (release version and/or kernel info). |
| ld.so.conf | Locations of library files. |
| logrotate.* | Rotation of the logs, a system preventing the collection of huge amounts of log files. |
| motd | Message Of The Day: Shown to everyone who connects to the system (in text mode), may be used by the system admin to announce system services/maintenance etc. |
| mtab | Currently mounted file systems. It is advised to never edit this file. |
| nsswitch.conf | Order in which to contact the name resolvers when a process demands resolving of a host name. |
| pam.d | Configuration of authentication modules. |
| passwd | Lists local users. Use the shadow utilities useradd, usermod and userdel to edit this file. Edit manually only when you really know what you are doing. |
| profile | System wide configuration of the shell environment: variables, default properties of new files, limitation of resources etc. |
| rc* | Directories defining active services for each run level. |
| resolv.conf | Order in which to contact DNS servers (Domain Name Servers only). |
| services | Connections accepted by this machine (open ports). |
| sndconfig or sound | Configuration of the sound card and sound events. |
| ssh | Directory containing the config files for secure shell client and server. |
| sysconfig | Directory containing the system configuration files: mouse, keyboard, network, desktop, system clock, power management etc. (specific to RedHat) |
| X11 | Settings for the graphical server, X. RedHat uses XFree, which is reflected in the name of the main configuration file, XFree86Config. Also contains the general directions for the window managers available on the system, for example gdm, fvwm, twm, etc. |
| xinetd.* or inetd.conf | Configuration files for Internet services that are run from the system's (extended) Internet services daemon (servers that don't run an independent daemon). |

### Linking files

#### Link types

A link is nothing more than a way of matching two or more file names to the same set of file data. There are two ways to achieve this:

* Hard link: Associate two or more file names with the same inode. Hard links share the same data blocks on the hard disk, while they continue to behave as independent files.  
  There is an immediate disadvantage: hard links can't span partitions, because inode numbers are only unique within a given partition.

* Soft link or symbolic link (or for short: symlink): a small file that is a pointer to another file. A symbolic link contains the path to the target file instead of a physical location on the hard disk. Since inodes are not used in this system, soft links can span across partitions.

The two link types behave similar, but are not the same, as illustrated in the scheme below:  
__Hard and soft link mechanism__
![Hard and soft link mechanism](./links.png)

#### Creating symbolic links

```bash
ln -s targetfile linkname
```

Example:

```bash
freddy:~/music> ln -s /opt/mp3/Queen/ Queen

freddy:~/music> ls -l
lrwxrwxrwx  1 freddy  freddy  17 Jan 22 11:07 Queen -> /opt/mp3/Queen
```

### File security

#### Access rights: Linux's first line of defense

We already used the long option to list files using the ls -l command, though for other reasons. This command also displays file permissions for these three user categories; they are indicated by the nine characters that follow the first character, which is the file type indicator at the beginning of the file properties line. As seen in the examples below, the first three characters in this series of nine display access rights for the actual user that owns the file. The next three are for the group owner of the file, the last three for other users. The permissions are always in the same order: read, write, execute for the user, the group and the others. Some examples:

```bash
marise:~> ls -l To_Do
-rw-rw-r--    1 marise  users      5 Jan 15 12:39 To_Do
marise:~> ls -l /bin/ls
-rwxr-xr-x    1 root    root   45948 Aug  9 15:01 /bin/ls*
```

Examples:

```bash
asim:~> ./hello
bash: ./hello: bad interpreter: Permission denied

asim:~> cat hello
#!/bin/bash
echo "Hello, World"

asim:~> ls -l hello
-rw-rw-r--    1 asim    asim    32 Jan 15 16:29 hello

asim:~> chmod u+x hello

asim:~> ./hello
Hello, World

asim:~> ls -l hello
-rwxrw-r--   1 asim    asim    32 Jan 15 16:29 hello*
```

```bash
asim:~> chmod u+rwx,go-rwx hello

asim:~> ls -l hello
-rwx------    1 asim    asim    32 Jan 15 16:29 hello*
```

##### Access mode codes

| Code | Meaning |
| ---- | ------- |
| 0 or - | The access right that is supposed to be on this place is not granted. |
| 4 or r | read access is granted to the user category defined in this place |
| 2 or w | write permission is granted to the user category defined in this place |
| 1 or x | execute permission is granted to the user category defined in this place |

##### User group codes

| Code | Meaning |
| ---- | ------- |
| u | user permissions |
| g | group permissions |
| o | permissions for others |

##### File protection with chmod

| Command | Meaning |
| ------- | ------- |
| chmod 400 file | To protect a file against accidental overwriting. |
| chmod 500 directory | To protect yourself from accidentally removing, renaming or moving files from this directory. |
| chmod 600 file | A private file only changeable by the user who entered this command. |
| chmod 644 file | A publicly readable file that can only be changed by the issuing user. |
| chmod 660 file | Users belonging to your group can change this file, others don't have any access to it at all. |
| chmod 700 file | Protects a file against any access from other users, while the issuing user still has full access. |
| chmod 755 directory | For files that should be readable and executable by others, but only changeable by the issuing user. |
| chmod 775 file | Standard file sharing mode for a group. |
| chmod 777 file | Everybody can do everything to this file. |

##### File permissions

| Who\What | r(ead) | w(rite) | (e)x(ecute) |
| u(ser) | 4 | 2 | 1 |
| g(roup) | 4 | 2 | 1 |
| o(ther) | 4 | 2 | 1 |

#### The file mask

When a new file is saved somewhere, it is first subjected to the standard security procedure. Files without permissions don't exist on Linux. The standard file permission is determined by the mask for new file creation. The value of this mask can be displayed using the umask command:

```bash
bert:~> umask
0002
```

When creating a new file, this function will grant read and write permissions for everybody, but set execute permissions to none for all user categories. This, before the mask is applied, a directory has permissions 777 or rwxrwxrwx, a plain file 666 or rw-rw-rw-.

The umask value is subtracted from these default permissions after the function has created the new file or directory. Thus, a directory will have permissions of 775 by default, a file 664, if the mask value is (0)002. This is demonstrated in the example below:

```bash
bert:~> mkdir newdir

bert:~> ls -ld newdir
drwxrwxr-x    2 bert    bert    4096 Feb 28 13:45 newdir/

bert:~> touch newfile

bert:~> ls -l newfile
-rw-rw-r--    1 bert    bert    0 Feb 28 13:52 newfile
```

The root user usually has stricter default file creation permissions:

```bash
[root@estoban root]# umask
022
```

#### Special modes

For the system admin to not be bothered solving permission problems all the time, special access rights can be given to entire directories, or to separate programs. There are three special modes:

* Sticky bit mode: After execution of a job, the command is kept in the system memory. Originally this was a feature used a lot to save memory: big jobs are loaded into memory only once. But these days memory is inexpensive and there are better techniques to manage it, so it is not used anymore for its optimizing capabilities on single files. When applied to an entire directory, however, the sticky bit has a different meaning. In that case, **a user can only change files in this directory when she is the user owner of the file or when the file has appropriate permissions.** This feature is used on directories like /var/tmp, that have to be accessible for everyone, but where it is not appropriate for users to change or delete each other's data. The sticky bit is indicated by a t at the end of the file permission field:

  ```bash
  mark:~> ls -ld /var/tmp
  drwxrwxrwt   19 root     root         8192 Jan 16 10:37 /var/tmp/
  ```

  The sticky bit is set using the command chmod o+t directory. The historic origin of the "t" is in UNIX' save Text access feature.

* SUID (set user ID) and SGID (set group ID): represented by the character s in the user or group permission field. When this mode is set on an executable file, it will run with the user and group permissions on the file instead of with those of the user issuing the command, thus giving access to system resources.

* SGID (set group ID) on a directory: in this special case every file created in the directory will have the same group owner as the directory itself (while normal behavior would be that new files are owned by the users who create them). This way, users don't need to worry about file ownership when sharing directories:

  ```bash
  mimi:~> ls -ld /opt/docs
  drwxrws---  4 root    users          4096 Jul 25 2001 docs/
  mimi:~> ls -l /opt/docs
  -rw-rw----  1 mimi    users        345672 Aug 30 2001-Council.doc
  ```

## References

* [Introduction to Linux](https://linux.die.net/Intro-Linux/)
