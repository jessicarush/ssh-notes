# SSH

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

To ssh to a name instead of an IP address, modify your hosts file:
```
$ sudo nano /etc/hosts
```

Add the ip address and the name you want to use to this list.

## Copy Files via SSH

To copy a local directory to another computer via ssh:
```
scp -r /path/to/local/file username@hostname:/path/to/remote/file
```

for example:
```
scp -r /Users/jessicarush/Documents/Coding jessica@rush-imac:/Users/jessica/Documents/Coding
```

## Transfer Files with rsync

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

## Aliases

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
