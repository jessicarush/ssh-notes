# SSH

This is a very brief collection of notes for working with and setting up SSH.

## Table of Contents

<!-- toc -->

- [Basic command](#basic-command)
- [Setting up SSH keys](#setting-up-ssh-keys)
- [SSH using a custom hostname](#ssh-using-a-custom-hostname)
- [Copy Files via SSH](#copy-files-via-ssh)
- [Transfer Files with rsync](#transfer-files-with-rsync)
- [Manage remote files using Mac OS Finder](#manage-remote-files-using-mac-os-finder)
- [Manage remote files with SSHFS](#manage-remote-files-with-sshfs)
- [SSH Key Forwarding](#ssh-key-forwarding)

<!-- tocstop -->

## Basic command

To ssh into another machine:

```bash
ssh username@hostname
```

`username` is the username of the machine you're ssh-ing into.
`hostname` can be a url, IP address, or a hostname on your local network.

For example:

```bash
jessica@123.456.7.89
pi@Rush-pi
git@github.com
```

By default, you will need to enter your password every time you ssh. This gets really tedious so what you want to do is set up ssh keys. This removes the requirement to enter passwords all the time.

*Note: to exit from ssh type `exit`*

## Setting up SSH keys

A directory called .ssh should be located in your home directory:

```bash
cd ~/.ssh
```

Inside this directory you will need 4 files:

- **id_rsa** (your private key...do not share with anything)
- **id_rsa.pub** (your public key)
- **authorized_keys** (a file to which you can append trusted keys)
- **known_hosts** (a file managed by the OS)

To create your keys if they don't already exist:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Note that this command also works on Windows 10, provided you [have OpenSSH installed](https://phoenixnap.com/kb/generate-ssh-key-windows-10).

The above command will generate the keys which you can now see by typing:

```bash
cat ~/.ssh/id_rsa.pub
```

or in Windows 10:

```bash
cat /mnt/c/Users/usern/.ssh/.id_rsa.pub
```

Copy the key and append it to the authorized_keys file (>> will create the file if it doesn't already exist):

```bash
cat ~/.ssh/id_rsa.pub >> authorized_keys
```

This copies the public key of the machine you're on and authorizes it for the same machine. To copy the key to a different machine use the pipe command and ssh to that machines authorized_keys file:

```bash
cat ~/.ssh/id_rsa.pub | ssh username@hostname "cat >> ~/.ssh/authorized_keys"
```

or even easier:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub username@hostname
```

Now when you exit ssh and login again, you won't need to enter passwords for authorized machines. Note, for any machine to be accessed by ssh, you need to enable ssh. On the raspberry pi the option to enable ssh is located in the main preferences. On the Mac, it's located in:

- System Preferences > Sharing > Remote login

You can also copy your public key to places like:

- github (see settings > SSH and GPG keys)
- digital ocean (see settings > security > SSH keys)
- bitbucket (see personal settings > SSH keys)

Note: if at some point you rebuild the OS on the device you are ssh-ing into, it will have a new 'fingerprint'. You will need to 'forget' the existing RSA key for that host by using this command:

```bash
ssh-keygen -R hostname
```

## SSH using a custom hostname

There are a few ways to ssh to a name instead of an IP address. One way is modify your hosts file:

```bash
sudo nano /etc/hosts
```

Add the ip address and the name you want to use to this list. For example:

```text
10.0.0.15      unoSSD
```

Another way os to set up a `config` file in `~/.ssh`:

```bash
touch ~/.ssh/config
```

The contents should be configured like:

```text
Host unoSSD
    Hostname 10.0.0.16
    User pi
    Port 22

Host dos
    Hostname 10.0.0.17
    User pi
    Port 22

Host tres
    Hostname 10.0.0.18
    User pi
    Port 22
```

## Copy Files via SSH

`scp` performs a plain linear copy from one location to another.

To copy a local directory to another computer via ssh:

```bash
scp -r /path/to/local/file username@hostname:/path/to/remote/file
```

For example:

```bash
scp -r /Users/jessicarush/Documents/Coding jessica@rush-imac:/Users/jessica/Documents/Coding

scp /Users/jessicarush/Documents/app.db jessica@138.197.151.122:/home/jessica/activity-log/

scp -r my_project_dir/ pi@tres:/home/pi/
```

To copy from the remote location to your local directory:

```bash
scp -r username@hostname:/path/to/remote/file /path/to/local/file
```

For example:

```bash
scp -r jessica@138.197.151.122:/home/jessica/activity-log/app.db /Users/jessicarush/Documents
```

## Transfer Files with rsync

`rsync` copies files from one location to another using a special transfer algorithm. Most notably, it will check files sizes and modification timestamps to only copy over files that don't match up. It also offers many more command line options to fine tune its behaviour.

To copy from macbook to imac:

```bash
rsync -r -a -v -e ssh --delete /Users/jessicarush/Documents/Coding jessica@rush-imac:/Users/jessica/Documents/
```

To copy from imac to macbook:

```bash
rsync -r -a -v -e ssh --delete /Users/jessica/Documents/Coding jessicarush@rush-mb:/Users/jessicarush/Documents/
```

Note that the `--delete` flag allows files to be deleted on they target machine.
To do a dry-run, add the `--dry-run` flag:

```bash
rsync -r -a -v -e ssh --delete --dry-run /...
```

see: <https://kyup.com/tutorials/copy-files-rsync-ssh/>

## Manage remote files using Mac OS Finder

You can view files on a remote computer (i.e. a Raspberry Pi) using the 'Connect to Server...' command in Mac OS. So far I've only tried this with a Raspberry Pi.

1. ssh into your pi and install [netatalk](http://netatalk.sourceforge.net/)

  ```bash
  sudo apt-get install netatalk
  ```

2. open the config file

  ```bash
  sudo nano /etc/netatalk/afp.conf
  ```

3. edit the [Homes] section and note that the ';' comments out a line, so you'll need to uncomment the two lines:

  ```text
  ; Netatalk 3.x configuration file
  ;

  [Global]
  ; Global server settings

   [Homes]
   basedir regex = /home

  ; [My AFP Volume]
  ; path = /path/to/volume

  ; [My Time Machine Volume]
  ; path = /path/to/backup
  ; time machine = yes
  ```

Close and save the file.

4. Restart Netatalk

  ```bash
  sudo systemctl restart netatalk
  ```

  Note you can also stop and start netatalk with these commands:

  ```bash
  sudo /etc/init.d/netatalk stop
  sudo /etc/init.d/netatalk start
  ```

5. From your Mac: Go > Connect to Server... and enter the address

  ```text
  afp://tres.local
  afp://10.0.0.17
  ```

  Where tres is my pi's Host name from my .ssh/config. You should also be able to type the ip address here.

  You'll need to enter the user name (pi) and password, then that should be it.


## Manage remote files with SSHFS

See: [Mapping Network Drive Over SSH in Windows](http://makerlab.cs.hku.hk/index.php/en/mapping-network-drive-over-ssh-in-windows)

TLDR:

1. Install [WinFsp](https://github.com/billziss-gh/winfsp/releases)

2. Install [SSHFS-Win](https://github.com/billziss-gh/sshfs-win/releases)

3. Map a network drive with `\\sshfs\username@hostanme`, for example:

```text
\\sshfs\pi@10.0.0.17
```

Done.

Note: Some times sshfs will just randomly stop working. This is a result of the ``winFSB.launcher`` service stopping. To start it up again search ``services`` in the windows menu, find winFSP.launcher and right-click to start it again. See [this issue on github](https://github.com/winfsp/sshfs-win/issues/214) for details.


## SSH Key Forwarding

See: [Linux command line: passing keys using ssh-agent](https://www.youtube.com/watch?v=7Log2jJtQqg)

This is useful when you are ssh-ing into a remote linux box and working with a cloned private github repo. The goal is that we want to use ssh keys to avoid having to type usernames and passwords every time we pull (also, [GitHub will no longer allow this](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/) as of August, 2021). Rather than adding ssh keys from the remote linux box to our GitHub account, we'd rather use the keys from our local machine, which we'll assume have already been added to our GitHub account.

First start the ssh-agent in the command line:

```bash
eval $(ssh-agent -s)
```

Then add a key to it that we will forward:

```bash
ssh-add ~/.ssh/id_rsa
```

Confirm that it's been added with the list flag:

```bash
ssh-add -l
```

Now ssh into your remote linux box using the `-A` flag. This flag passes in the keys you just added with `ssh-add`:

``` bash
ssh -A username@Hostname
```

You can run `ssh-add -l` on the remote box to confirm the keys got passed in.

That's it. You should now be able to pull, fetch, clone from GitHub and it will use the keys passed in from your local machine.

To take things further, we can shorten this process by adding the instructions to pass in keys to our `.ssh/config`.

```text
Host activity-log
    Hostname 138.197.151.122
    user jessica
    ForwardAgent yes
```

You would still need to run `eval $(ssh-agent -s)` and `ssh-add ~/.ssh/id_rsa` though. If you want to automate that too, add those commands to you `.bashrc`:

```bash
# For SSH Key forwarding
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
```

Don't forget to `.source ~/.bashrc` afterwards.
