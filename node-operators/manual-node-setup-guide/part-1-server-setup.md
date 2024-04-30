---
description: >-
  This section will help you set up and configure a server to prepare it for
  being a Pocket node.
---

# Part 1 - Server Setup

## Setup a server <a href="#setup-a-server" id="setup-a-server"></a>

The first thing you’ll need to run a Pocket node is a server. For this guide, we’ll be using a virtual machine on the [Linode](https://www.linode.com/) cloud service, but you can use any cloud service you like, or run a server of your own.

{% hint style="info" %}
Pocket has no affiliation with Linode and does not recommend any one provider over another. The general steps outlined here should work for most cloud providers.
{% endhint %}

Let’s start by creating a Linode instance (a virtual machine).

### 1. Create a Linode instance <a href="#create-a-linode-instance" id="create-a-linode-instance"></a>

To create a Linode instance, do the following:

1. Sign up for a [Linode](https://www.linode.com/) account and login.
2. Create a new Linode instance with the following specifications:
   * **Image / Distribution**: `Ubuntu 20.04 LTS`
   * **Region**: `Atlanta, GA`
   * **Linode Plan**: `Dedicated 16 GB - 8 CPU, 320 GB Storage, 16 GB RAM`
   * **Linode Label**: `pokt001`
3. Wait for the Linode instance to be created and show up as running in the web interface.

{% hint style="info" %}
For a more detailed guide on setting up a Linode instance, see the [Linode docs](https://www.linode.com/docs/guides/getting-started/). Also, note that the `Atlanta, GA` region was selected for this guide because it supports [NVMe storage](https://www.linode.com/products/block-storage/#nvme-block-storage) which is preferable for running nodes. [Check to see which other regions support NVMe storage](https://www.linode.com/blog/cloud-storage/nvme-block-storage-global-rollout/).
{% endhint %}

### 2. Add a storage volume <a href="#add-a-storage-volume" id="add-a-storage-volume"></a>

The Pocket blockchain is very large and growing all the time, and the snapshot we’ll be downloading in a later step is too large to fit on this Linode instance.

Because of this, we’ll need to create a secondary storage volume. Our recommendation is at least 1.3TB for a full node, and 150GB to begin with. But as you'll need more space in the future, opt for a larger volume or one that be dynamically adjusted.

{% hint style="info" %}
Note that if you are downloading a snapshot and using the Aria2 program to download it, you will need to have *DOUBLE* the reccomendation above for the initial sync.  This is because the Aria2 download method downloads the full snapshot which for full node is currently ~1.1TB, and the Aria method downloads the full snapshot tarball using multiple connections (much faster than `wget` or `curl`) but it cannot extract the file simultainously. It must download the full snapshot (which for full snapshots is over 1TB) and then once downloaded fully it can be extracted, but this process extracts the snapshot Data Directory from the tarball and only once it completes can the tarball be deleted.  So if using the Aria download method you need to account for this while settting up your node, since you'll need at least 2.2TB of storage during initial sync when using a full snapshot.
The inline wget download method avoids this by piping the download command to the extraction command, which makes this massive amount of storage unnecessary because it downloads the file immediately pipes the downloaded files to the next command which extracts the downloaded parts as they're downloaded eliminating the need to have temporary storage space (~2x what is reccomended) to hold the download while it's being extracted.
The downside to using this method is that it's much slower than aria, instead of being able to download using up to 16 connections at once, wget is limited to one connection.  Also because the file for the full snapshot is so large these days, connection issues at anytime during the download on the client side or server side, can easily stop the download or corrupt the data so that the whole procss (which on a residential connection can take several days or more) needs to be restarted. There's no way to pick up where it was left off since the downloaded file has been altered already long before it's even been completely downloaded.
{% endhint %}

1. In your Linode account, click **Volumes** and then **Create Volume**.
2. Create a volume with the following specifications:
   * **Label**: `poktuserdir`
   * **Size**: 1.3TB for full node - 150GB for a pruned node
   * **Region**: \[Same as your instance]
   * **Linode**: `pokt001`

### 3. Configure DNS <a href="#configure-dns" id="configure-dns"></a>

Now that the Linode instance is created and running, you’ll need to set up a DNS record that points to the IP address of the Linode instance.

Pocket nodes require a DNS name. DNS ([Domain Name Service](https://www.cloudflare.com/learning/dns/what-is-dns/)) names are used to map an IP address to more human-friendly names. So rather than referencing a server with an address like `134.23.153.21` we can use a name like `pokt001.pokt.run`.

{% hint style="info" %}
Most domain registrars allow you to add DNS records. Please refer to the DNS setup documentation for your provider.
{% endhint %}

Specifically, you’ll need to add an `A` record for the domain name. For the exact steps, consult the DNS documentation for your provider. Then create a record with the following information:

* **Name**: `pokt001`
* **Type**: `A`
* **Value**: `[Linode_IP_Address]`
* **TTL**: `300`

After setting up your DNS record, wait a few minutes for the DNS to propagate. Then use the following command to check that the DNS record is working:

{% hint style="info" %}
The examples in this tutorial will use `pokt001` as the server on the `pokt.run` domain, so `pokt001.pokt.run` will be used as the DNS name. **Please replace this throughout with your own server and domain name.**
{% endhint %}

```bash
ping -c 3 pokt001.pokt.run
```

You should see a response that looks something like this:

```bash
64 bytes from 134.23.153.21: icmp_seq=0 ttl=47 time=92.403 ms
64 bytes from 134.23.153.21: icmp_seq=1 ttl=47 time=142.828 ms
64 bytes from 134.23.153.21: icmp_seq=2 ttl=47 time=182.456 ms
```

If the IP address matches the IP address of your Linode instance, you’re all set!

{% hint style="info" %}
It can sometimes take longer than a minute for the DNS to propagate. So, be patient if things don’t seem to work right away.
{% endhint %}

### 4. Login with SSH <a href="#login-with-ssh" id="login-with-ssh"></a>

Now that we have a DNS record setup, we will look at using SSH to log in and continue the setup process.

The [Secure Shell Protocol](https://en.wikipedia.org/wiki/Secure\_Shell) (SSH) is a secure way to connect to your Linode instance from a remote machine, like your local computer. We’ll be using SSH to complete the rest of the setup process.

#### SSH from Mac or Linux <a href="#ssh-from-mac-or-linux" id="ssh-from-mac-or-linux"></a>

If you’re using a Mac, or Linux, on your local computer, you can SSH into your node by doing the following:

* Open a terminal
  * SSH into your node using the following command:

    ```bash
    ssh root@pokt001.pokt.run
    ```

{% hint style="info" %}
Don’t forget to replace `pokt001.pokt.run` with your DNS name.
{% endhint %}

You’ll be asked for your password. This is the root password that you set when you created your Linode.

#### SSH from Windows <a href="#ssh-from-windows" id="ssh-from-windows"></a>

Windows 10 and later have a built-in SSH client. You can use SSH on Windows by doing the following:

* Open the Windows terminal
* SSH into your node using the following command:

    ```bash
    ssh root@pokt001.pokt.run
    ```

{% hint style="info" %}
Don’t forget to replace `pokt001.pokt.run` with your DNS name.
{% endhint %}

If you’re using an older version of Windows, you might need to install [PuTTY](https://www.putty.org/) or some other SSH client.

### 5. Set the hostname <a href="#set-the-hostname" id="set-the-hostname"></a>

At this point you should be logged into your node as the `root` user.

In a previous step, we set the DNS name for the node. Now we’ll use the same name for the hostname on the server.

To set the server hostname use the following steps:

1. Open the `/etc/hostname` file with the following command:

    ```bash
    nano /etc/hostname
    ```

2. Change the `localhost` value to the hostname of your node (for example, `pokt001.pokt.run`).
3. Save the file with `Ctrl+O` and then `Enter`.
4. Exit nano with `Ctrl+X`.
5. Reboot the server with the following command:

    ```bash
    reboot
    ```

6. Wait for the server to reboot then ssh back in as the `root` user before continuing on.

### 6. Create a Pocket user account <a href="#create-a-pocket-user-account" id="create-a-pocket-user-account"></a>

For security reasons it’s best not to use the `root` user. Instead, it’s better to create a new user and add the user to the `sudo` group.

To create a new user, enter the following commands:

1. Create a new user named `pocket`, add it to the `sudo` group, and set the default shell to `bash`. If you want to specify the location of the home directory, you can use the `-d` option followed by the path to the home directory:

    ```bash
    useradd -m -g sudo -s /bin/bash pocket && passwd pocket
    ```

2. For the rest of this guide, we’ll be using the `pocket` user. So now that the `pocket` user is created, you can switch from using `root` to the `pocket` user with the following command:

    ```bash
    su - pocket
    ```

### 7. Mount the volume <a href="#mount-the-volume" id="mount-the-volume"></a>

Next we want to mount the secondary storage volume that we created in a previous step.

1. Verify that the volume is attached to your instance.

    CommandResponse

    ```bash
    sudo fdisk -l
    ```

2. Create a new partition. If the previous command shows a file path different from `/dev/sdc`, use that instead in the commands below:

    ```bash
    sudo mkfs.ext4 /dev/sdc
    ```

3. Create a new mount point:

    ```bash
    sudo mkdir /mnt/data
    ```

4. Mount the new partition:

    ```bash
    sudo mount /dev/sdc /mnt/data
    ```

5. Verify that the partition was created by running the following command:

    ```bash
    sudo lsblk -o NAME,PATH,SIZE,FSAVAIL,FSUSE%,MOUNTPOINT
    ```

6. Set the volume to be mounted automatically. Open `/etc/fstab`:

    ```bash
    sudo nano /etc/fstab
    ```

7. Add the following line to the bottom of the file:

    ```bash
    /dev/sdc /mnt/data ext4 defaults,noatime,nofail 0 2
    ```

8. Save the file with `Ctrl+O` and then `Enter`.

9. Exit nano with `Ctrl+X`.

### 8. Move the home directory <a href="#move-the-home-directory" id="move-the-home-directory"></a>

Many Pocket commands assume a data directory path of `~/.pocket`. While it is possible to specify a different data directory with every command, it will be much easier to change the location of the `pocket` user home directory. For this tutorial, we will be putting the Pocket data directory at `/mnt/data/.pocket`.

To change the home directory of the `pocket` user:

```bash
sudo usermod -d /mnt/data pocket
```

#### Configure SSH Key Login (Optional): <a href="#configure-ssh-key-login-optional" id="configure-ssh-key-login-optional"></a>

While not required, using an SSH key provides a more secure means of accessing your server.

Using an SSH key removes the ability for credentials to be sniffed in the login process, and removes the pitfalls that can often come with user generated passwords since the key will truly be random.

One important thing to understand, is that without access to the ssh key, you won’t be able to log into your node. If you intend on accessing your node from multiple computers, it’s recommended that you repeat the Generate Key and Upload Key steps from each computer that you intend to access your node from before moving on to the Disable Root Login and Password Authentication step.

1. **Log Out**

    At the terminal you’ll need to enter the `logout` command twice. The first logout logs you out of the pocket user, back to the root user, and the second logout logs you out of the server and back to your terminal.

2. **Generate Key**

    Next, we’ll generate an ssh key. To do that you’ll run the ssh-keygen command. You’ll be prompted to specify the file you want to save the key to, and for a password. Specifying a password means that if someone has access to your key, they’d still need to know the password to be able to use it to login. To create the key, do the following:

    * Run the ssh-keygen command

        ```bash
        ssh-keygen -t rsa -b 4096
        ```

    * Enter file in which to save the key (`~/.ssh/id_rsa`)
    * Enter a passphrase (empty for no passphrase)
    * Enter same passphrase again

    The results should looking something like the following:

    ```bash
    The key fingerprint is:
    SHA256:jr2MLXIha188wYsp/bNflN9BuqQ3LWCAXJNTtHO7sWk
    The key's randomart image is:
    +---[RSA 4096]----+
    |         o+o     |
    |      . oo. .    |
    |       o ..o . . |
    |       .  . o.+  |
    |        S  oo= . |
    |    ...B o..+.B..|
    |    .o=.B  ..E...|
    |    +.o*.o .o o  |
    |   . +o.*+.      |
    +----[SHA256]-----+
    ```

3. **Upload Key**

    Now we’re going to upload the key so that we can use it to log into the pocket user. If you choose a different path for the ssh key, it’s important to replace the `~/.ssh/id_rsa` with the key you used.

    ```bash
    ssh-copy-id -i ~/.ssh/id_rsa pocket@pokt001.pokt.run
    ```

{% hint style="info" %}

```
Windows users may not have access to this command. If you don't have access to a Bash shell, you can use PowerShell to mimic this command. [See these instructions for more details.](https://chrisjhart.com/Windows-10-ssh-copy-id/)
```

{% endhint %}

1. **Disable Root Login and Password Authentication**

    Now we’re now going to configure ssh to no longer allow root logins, and to not allow any password based login attempts. Meaning without access to the ssh key for the pocket user, no one will be able to log into the server.

    First we’ll need to log back into the server:

    ```bash
    ssh pocket@pokt001.pokt.run
    ```

    From there, we’ll want to open the `/etc/ssh/sshd_config` file to make some changes to the default configuration:

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

    Once there, we’ll need to find and change the following lines:

    * `#PermitRootLogin prohibit-password` -> `PermitRootLogin no`
    * `#PubkeyAuthentication yes` -> `PubkeyAuthentication yes`
    * `#PasswordAuthentication yes` -> `PasswordAuthentication no`

    Once changed, `Ctrl-O` followed by `Enter` will save the changes, and `Ctrl-X` will exit nano back to the terminal.

    Then we’ll need to restart the ssh server for these changes to take effect:

    ```bash
    sudo systemctl restart sshd.service
    ```

2. **Verify Everything Works**

    The last step is to log out of the server, and try logging back in. If you’re no longer prompted for a password, then everything is working as expected.

    Windows/Linux/macOS

    ```bash
    ssh -i C:\Users\<USER>\.ssh\id_rsa -l pocket pokt001.pokt.run
    ```

That’s it for the server setup! Continue on to install the necessary software.
