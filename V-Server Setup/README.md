# Your first Cloud-VM

In this README file you will find information on how to handle your first Cloud-VM. Apart from information related to the first-time login, this document also provides reading links for other interesting topics around Linux server environments or server administration.

## Table of Contents

- [Create an SSH-Key pair](#create-an-ssh-key-pair)
- [First-Time Login](#first-time-login)
- [Add the public key to the VM](#add-the-public-key-to-the-vm)
  - [Note for Windows users](#note-for-windows-users)
  - [Disable Password-Logins](#disable-password-logins)
- [How to configure and start a webserver](#how-to-configure-and-start-a-webserver)
  - [Configuring an alternative start page](#configuring-an-alternative-start-page)
- [Alternatives to logging in to my Cloud VM](#alternatives-to-logging-in-to-my-cloud-vm)
  - [1. Shell Alias (Linux/macOS only)](#1-shell-alias-linuxmacos-only)
  - [2. SSH Client Config](#2-ssh-client-config)

## Create an SSH-Key pair

In order to create an SSH-Key pair on your local machine, you can run the following command and follow the instructions in your terminal:

```bash
# create an ED25519 key pair
# modern alternative to RSA keys
$ ssh-keygen -t ed25519
```

## First-Time Login

When logging in to any application, we usually use username/password combinations. For this example we use a username/password combination for the first login only - for production-server environments, we want to use something safer than just a password alongside the username, we want to use a personalized key that allows us to connect to a remote shell on the server - this is the so-called `Secure Shell Protocol (SSH)` and the personalized key is called SSH-Key.

```bash
$ ssh root@203.0.113.10
password:
```

## Add the public key to the VM

```bash
# ssh-copy-id -i <path/to/your/key>.pub <user>@<hostname>
$ ssh-copy-id -i $HOME/.ssh/id_ed25519.pub example@203.0.113.10

/usr/local/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/example/.ssh/id_ed25519.pub"
/usr/local/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/local/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
example@203.0.113.10 password:

Number of key(s) added: 1

Now try logging into the machine with:     "ssh 'example@203.0.113.10' "
and check to make sure that only the key(s) you wanted were added.
```

### Note for Windows users

`ssh-copy-id` is not available on Windows. In PowerShell you can achieve the same result with:

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh example@203.0.113.10 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Disable Password-Logins

In this section information about disabling password-based logins will be provided.
Passwords are a potential security risk, which is why password logins should be disabled in favor of authenticating via SSH-Keys.

**Important:** Make sure that logging in with your SSH-Key works before disabling password logins - otherwise you may lock yourself out of the server.

1. Adjust the configuration under `/etc/ssh/sshd_config`
1. Find and edit the line `#PasswordAuthentication yes` so that it reads `PasswordAuthentication no`.
1. Save the file and exit
1. Restart the `sshd` service to reload the config changes

```bash
$ sudo nano /etc/ssh/sshd_config
$ sudo systemctl restart ssh.service
```

## How to configure and start a webserver

In this section, you will learn about running an `nginx` webserver on an Ubuntu cloud VM. After installing the webserver, you should be able to see the nginx server in your browser.

```bash
$ sudo apt update
$ sudo apt install nginx -y
```

### Configuring an alternative start page

After installing and testing our webserver, we now want to configure it to render an alternative `index.html` instead of the default `nginx` start page.

Follow these steps to reconfigure the nginx installation on your cloud VM:

1. create a new file `alternate-index.html` in the location `/var/www/alternatives/`
   - Ensure this directory exists by running `ls /var/www`
   - Create the directory if it does not yet exist with: `sudo mkdir /var/www/alternatives/`
1. add a configuration for the enabled sites for nginx under `/etc/nginx/sites-enabled/`, named `alternatives`
   - run `sudo nano /etc/nginx/sites-enabled/alternatives`
   - add the config from below<sup>1</sup>
1. Validate the configuration and restart the `nginx` service:
   - `sudo nginx -t`
   - `sudo systemctl restart nginx`
   - Open your browser with `http://<server-ip>:8081`<sup>2</sup> to see your configured alternative start page for the nginx webserver.

<sup>1</sup>nginx alternatives configuration file:

```nginx
server {
    listen 8081;
    listen [::]:8081;

    root /var/www/alternatives;
    index alternate-index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

<sup>2</sup>Note that the port differs from the HTTP standard port 80. Ensure you enter the ip address/domain information together with the port number, otherwise you may experience undesired behaviour.

For the HTML you can use the following snippet:

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello, Nginx!</title>
</head>
<body>
    <h1>Hello, Nginx!</h1>
    <p>I have just configured our Nginx web server on Ubuntu Server!</p>
</body>
</html>
```

## Alternatives to logging in to my Cloud VM

### 1. Shell Alias (Linux/macOS only)

An alias works like a shortcut for a long command such as
`ssh -i ~/.ssh/id_ed25519 sysops@203.0.113.10`.

Always use an absolute path for the key file, never a relative one.

```bash
$ alias myserver="ssh -i /Users/example/.ssh/id_ed25519 your_username@203.0.113.10"
```

Note: An alias only lasts for the current shell session. To make it
permanent, add the line to your `~/.bashrc`, e.g. by opening the file
in an editor:

```bash
$ code ~/.bashrc      # open in VS Code
```

Then reload the config so the alias is available immediately:

```bash
$ source ~/.bashrc
```

After that, the alias works in every new terminal session.

### 2. SSH Client Config

The SSH client config works on Linux, macOS, and Windows. It lets you define a short name for the server so you can connect with `ssh myserver` instead of the full command.

On Linux/macOS, check whether a config file already exists:

```bash
$ ls ~/.ssh
$ cat ~/.ssh/config   # view contents
$ vim ~/.ssh/config   # edit (create the file if it does not exist)
```

On Windows (PowerShell), the file is located at `%USERPROFILE%\.ssh\config`:

```powershell
$ code $env:USERPROFILE\.ssh\config   # open in VS Code (creates the file if it does not exist)
```

Note: the file has no extension - do not save it as `config.txt`.

Add an entry (use your key path, e.g. `C:\Users\example\.ssh\id_ed25519` on Windows):

```
Host myserver
    HostName 203.0.113.10
    User your_username
    PreferredAuthentications publickey
    IdentityFile /Users/example/.ssh/id_ed25519
```

No restart is required - the file is read on every `ssh` call. Afterwards you can connect with just:

```bash
$ ssh myserver
```