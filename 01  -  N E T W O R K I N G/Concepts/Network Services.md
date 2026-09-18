
[[TCP IP]] [[OSI Model]]
[THM room](https://tryhackme.com/room/networkservices)

## SMB

SMB (Server Message Block Protocol) is a client-server communication protocol used for sharing access to files, printers, serial ports and other resources on a network.

Servers make file systems and other resources (printers, named pipes, APIs) available to clients on the network. Client computers may have their own hard disks, but they also want access to the shared file systems and printers on the servers.

The protocol is known as a response-request protocol, meaning that it transmits multiple messages between the client and server to establish a connection. Clients connect to servers using /IP, NetBEUI or IPX/SPX.

Once they have established a connection, clients can then send commands (SMBs) to the server that allow them to access shares, open files, read and write files, and generally do all the sort of things that you want to do with a . However, in the case of , these things are done over the network.

Microsoft Windows operating systems since Windows 95 have included client and server protocol support. Samba, an open source server that supports the protocol, was released for Unix systems.

## Enumeration

**Enumeration** is the process of gathering information on a target in order to find potential attack vectors and aid in exploitation. 

```Enum4Linux``` is a tool used to enumerate SMB shares on both Windows and Linux systems. It's basically a wrapper around the tools in the Samba package and makes it easy to quickly extract information from the target pertaining to SMB. 

To install ```Enum4Linux``` check the official [github](https://github.com/portcullislabs/enum4linux).

Useful flags :

```bash
-U
	get userlist
-M
	get machine list
-N
	get namedlist dump
-S
	get sharelist
-P
	get password policy information
-G
	get group and member list
-a
	all of the above (full basic enumeration)
```

## SMB Exploit

Check all the SMB vulnerabilities on this [website](https://www.cvedetails.com/cve/CVE-2017-7494/).

##### Method Breakdown

From enumeration stage, we know :
- The SMB share location
- The name of an interesting SMB share

##### SMBClient

Because we're trying to access an SMB share, we need a client to access ressources on servers. We'll be using SMBClient because it's part of the default samba suite. The documentation is available [here](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html).

We can remotely access the SMB share using the syntax :

```bash
smbclient //<IP>/<SHARE> -U <USERNAME> -p <PORT>
```

Once inside the share, you can view the available commands by typing ```help``` but the useful of which are :
- ```ls``` or ```dir``` : List files and directories
- ```cd [DIR]``` : Move to a different directory
- ```get [FILE]``` : Download the file to your machine

### NFS

##### Mounting NFS shares

Get share folder :

```bash
/usr/sbin/showmount -e <IP>
```

Connect the NFS share to the mount point :

```bash
sudo mount -t nfs IP:<path to share folder> <mount_folder_path> -nolock
```

