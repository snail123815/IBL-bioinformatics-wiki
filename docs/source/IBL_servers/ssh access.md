# Termianl

*by Chao DU, Edder Bustos Diaz*

```{toctree}
---
#caption: Table of contents
maxdepth: 3
---
```

To use our Linux servers you will need access to a command line prompt.
- For MacOS (Apple operating system) users, the command line application  is called "Terminal", you can bring it up by typing "Terminal" in the spotlight.
- For Linux users, open the "terminal" application (sometimes it is also called "shell").
- For Windows users, there are several options
  - WSL (Windows Subsystem for Linux) with the default Linux distribution. You can find more detailed instructions on how to install it [here](https://learn.microsoft.com/en-us/windows/wsl/setup/environmentl).
  - GitBash provide a shell environment similar to Linux systems, the settings would be the same.
  - PowerShell comes by default with Windows. The latest versions comes with "ssh". (Not recommend. It seems this is the easiest method, but you will face some problems that we cannot predict. Also, the commands to setup will be a bit different.)

## Set up connection using ssh

When accessing from outside Leiden University network, we need to tunnel through a gateway, which is `sshgw.leidenuniv.nl`, to access our server. Once you can login using your user name and password to the gateway, you can proceed the setup.

There is no harm to access the servers through the gateway even when you are within Leiden University network. So this tutorial will assume you always do that.

After setup, you should access our servers (eg. BLIS) from your computer with one command: `ssh blis`.

### Create ssh key pair

In your terminal do:

```shell
ssh-keygen -t ed25519 -C "From my PC" -f ~/.ssh/iblservers
```

This command will ask you a "passphrase". Unless you want to enter this "passphrase" every time you access our servers, you can give it one and repeat once. Else, you can directly press ENTER.

You should see some output like this:

```
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in test
Your public key has been saved in test.pub
The key fingerprint is:
SHA256:M2238Cx5UCQgGivrRI5gtay92U3QULSQtTneghU2kO8 macbook
The key's randomart image is:
+--[ED25519 256]--+
|   ..+OB... .    |
|  o .*=.*  o     |
|..ooo..B    .    |
|o+oo  =.o. .     |
|..+. ..+S.= .    |
| o  + oE.+ B .   |
|  .o . .  o =    |
|           o     |
|                 |
+----[SHA256]-----+
```

Now you have created two files, which we call them a **pair** of keys. `~/.ssh/iblservers` is your private key, you do not show the content of this file to anyone; `~/.ssh/iblservers.pub` is your public key, you need to put this in your "home directory" of all servers.

We recommend you to create a separate key pair to access the gateway:

```shell
ssh-keygen -t ed25519 -C "From my PC" -f ~/.ssh/sshgwLeidenuniv
```

### Add keys to the servers

To Gateway, assume your ULCN user name is `ULCN`:

```{code-block} shell
---
emphasize-lines: 3, 8
caption: Highlighted will be the key hashes that will vary.
---
# On local machine
cat ~/.ssh/sshgwLeidenuniv.pub
> ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAc4bjSgGycLs/XZA27uFgXfsfx4i50MQWr3fSbBMZ6E From my PC
# copy the output lines to replace the same content in the following commands.
# now login gateway
ssh ULCN@sshgw.leidenuniv.nl
# enter your ULCN user name and password until you see "[ULCN@sshgw ~]$", then you can proceed
mkdir -p ~/.ssh
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAc4bjSgGycLs/XZA27uFgXfsfx4i50MQWr3fSbBMZ6E From my PC" >> ~/.ssh/authorized_keys
```

Now that you are on the gateway, please do not logout. From gateway, you can access one of our servers using the IP address from your email, here in this tutorial I use a mock IP address for BLIS, 999.999.999.999 as example, and assuming your login user name is `USERNAME`

```shell
ssh USERNAME@999.999.999.999
```

You will be promoted to enter the password, please copy the password from the email you got. This email will only be valid in your first login, you need to change your password immediately. Please read the screen carefully and follow the instruction, until you see:

```shell
[USERNAME@blis ~]$
```

Now enter `exit` to exit to gateway, then enter `exit` again to your local machine:

```
[USERNAME@blis ~]$ exit
[ULCN@sshgw ~]$ exit
```

Now you are back on you local machine, take a look at your public key for "iblservers" and put it on our IBL servers:

```{code-block} shell
# On local machine
cat ~/.ssh/iblservers.pub
> ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIArlumJ0HK5rmN9MM93ufBX09dgxiu0Sx7HphvyrMWRH From my PC
# copy the output lines to replace the same content in the following commands.
# now login gateway
ssh ULCN@sshgw.leidenuniv.nl
# enter your ULCN user name and password until you see "[ULCN@sshgw ~]$", then you can proceed
ssh USERNAME@999.999.999.999
mkdir -p ~/.ssh
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIArlumJ0HK5rmN9MM93ufBX09dgxiu0Sx7HphvyrMWRH From my PC" >> ~/.ssh/authorized_keys
```

Note: The above operations are essentially the same which can be done by `ssh-copy-id`.

Then exit to your local machine again, add the following lines to your `~/.ssh/config` file by:

```
echo "Host sshgw.leidenuniv.nl
    User ULCN
    HostName sshgw.leidenuniv.nl
    IdentityFile ~/.ssh/to_gw_leidenu

Host blis
    User USERNAME
    HostName 999.999.999.999
    ProxyCommand ssh -A sshgw.leidenuniv.nl -p 22 -q -W %h:%p
    IdentityFile ~/.ssh/to_blis
    ServerAliveInterval 60
    ServerAliveCountMax 10" >> .ssh/config
```

Now you should be able to login BLIS with one command from your local machine: `ssh blis`

For detailed instruction and explanation of how ssh works, please refer to How to login to [ALICE or SHARK - HPC wiki](https://pubappslu.atlassian.net/wiki/spaces/HPCWIKI/pages/37748771/How+to+login+to+ALICE+or+SHARK).