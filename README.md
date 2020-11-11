# SSH

This is a very brief collection of notes for working with and setting up SSH.

## Table of Contents

<!-- toc -->

- [Basic command](#basic-command)
- [Setting up SSH keys](#setting-up-ssh-keys)
- [SSH using a name](#ssh-using-a-name)
- [Copy Files via SSH](#copy-files-via-ssh)
- [Transfer Files with rsync](#transfer-files-with-rsync)
- [Manage remote files using Mac OS Finder](#manage-remote-files-using-mac-os-finder)
- [Side note: bash_aliases](#side-note-bash_aliases)

<!-- tocstop -->

## Basic command

To ssh into another machine:
```
$ ssh username@hostname
```
`username` is the username of the machine you're ssh-ing into.  
`hostname` can be a url, IP address, or a hostname on your local network.

For example:

```
$ jessica@123.456.7.89
$ pi@Rush-pi
$ git@github.com
```

By default, you will need to enter your password every time you ssh. This gets really tedious so what you want to do is set up ssh keys. This removes the requirement to enter passwords all the time.

*Note: to exit from ssh type `exit`*

## Setting up SSH keys

A directory called .ssh should be located in your home directory:
```
$ cd ~/.ssh
```

Inside this directory you will need 4 files:
 - **id_rsa** (your private key...do not share with anything)
 - **id_rsa.pub** (your public key)
 - **authorized_keys** (a file to which you can append trusted keys)
 - **known_hosts** (a file managed by the OS)

To create your keys if they don't already exist:
```
$ cd ~/.ssh
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

This will generate the keys which you can now see by typing:
```
$ cat ~/.ssh/id_rsa.pub
```

Copy the key and append it to the authorized_keys file (>> will create the file if it doesn't already exist):
```
$ cat ~/.ssh/id_rsa.pub >> authorized_keys
```

This copies the public key of the machine you're on and authorizes it for the same machine. To copy the key to a different machine use the pipe command and ssh to that machines authorized_keys file:
```
$ cat ~/.ssh/id_rsa.pub | ssh username@hostname "cat >> ~/.ssh/authorized_keys"
```

Now when you exit ssh and login again, you won't need to enter passwords for authorized machines. Note, for any machine to be accessed by ssh, you need to enable ssh. On the raspberry pi the option to enable ssh is located in the main preferences. On the Mac, it's located in:

- System Preferences > Sharing > Remote login

You can also copy your public key to places like:

- github (see settings > SSH and GPG keys)
- digital ocean (see settings > security > SSH keys)

## SSH using a name

There are a few ways to ssh to a name instead of an IP address. One way is modify your hosts file:
```
$ sudo nano /etc/hosts
```

Add the ip address and the name you want to use to this list. For example:

```
10.0.0.15      unoSSD
```

Another way os to set up an ssh `config` file in `~/.ssh`:

```
$ touch ~/.ssh/config
```

The contents should be configured like:

```
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
```
scp -r /path/to/local/file username@hostname:/path/to/remote/file
```

For example:
```
scp -r /Users/jessicarush/Documents/Coding jessica@rush-imac:/Users/jessica/Documents/Coding

scp /Users/jessicarush/Documents/app.db jessica@138.197.151.122:/home/jessica/activity-log/

scp -r my_project_dir/ pi@tres:/home/pi/
```

To copy from the remote location to your local directory:
```
scp -r username@hostname:/path/to/remote/file /path/to/local/file
```
For example:
```
scp -r jessica@138.197.151.122:/home/jessica/activity-log/app.db /Users/jessicarush/Documents
```

## Transfer Files with rsync

`rsync` copies files from one location to another using a special transfer algorithm. Most notably, it will check files sizes and modification timestamps to only copy over files that don't match up. It also offers many more command line options to fine tune its behaviour.

To copy from macbook to imac:
```
rsync -r -a -v -e ssh --delete /Users/jessicarush/Documents/Coding jessica@rush-imac:/Users/jessica/Documents/
```
To copy from imac to macbook:
```
rsync -r -a -v -e ssh --delete /Users/jessica/Documents/Coding jessicarush@rush-mb:/Users/jessicarush/Documents/
```
Note that the `--delete` flag allows files to be deleted on they target machine.

To do a dry-run, add the `--dry-run` flag:
```
rsync -r -a -v -e ssh --delete --dry-run /...
```

see: <https://kyup.com/tutorials/copy-files-rsync-ssh/>

## Manage remote files using Mac OS Finder

You can view files on a remote computer (i.e. a Raspberry Pi) using the 'Connect to Server...' command in Mac OS. So far I've only tried this with a Raspberry Pi.

1. ssh into your pi and install [netatalk](http://netatalk.sourceforge.net/)
  ```
  sudo apt-get install netatalk
  ```

2. open the config file
  ```
  sudo nano /etc/netatalk/afp.conf
  ```

3. edit the [Homes] section and note that the ';' comments out a line, so you'll need to uncomment the two lines:

  ```
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
  ```
  sudo systemctl restart netatalk
  ```
  Note you can also stop and start netatalk with these commands:
  ```
  sudo /etc/init.d/netatalk stop
  sudo /etc/init.d/netatalk start
  ```

5. From your Mac: Go > Connect to Server... and enter the address
  ```
  afp://tres.local
  ```
  Where tres is my pi's Host name from my .ssh/config. You should also be able to type the ip address here.

  You'll need to enter the user name (pi) and password, then that should be it.

## Side note: bash_aliases

There are a couple of different places were you can create aliases (command-line shortcuts). I use the `.bash_aliases` file.

```
$ nano ~/.bash_aliases
```

Add your alias like so:
```
alias sync-coding="rsync -r -a -v -e ssh --delete /Users/jessicarush/Documents/Coding jessica@rush-imac:/Users/jessica/Documents/
```
Reload the aliases in the command-line:
```
$ source ~/.bash_aliases
```
Run the command:
```
$ sync-coding
```
