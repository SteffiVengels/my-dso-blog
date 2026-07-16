# Your first Cloud-VM

This guide walks you through setting up your first cloud VM (V-Server) step by step. You will log in to the server, secure it with an SSH key, install a web server, and connect the server to GitHub. Every command is explained, so you can follow along even if this is your first time working with a Linux server.

## Table of Contents

- [Prerequisites](#prerequisites)
- [First-Time Login](#first-time-login)
- [Create an SSH-Key pair](#create-an-ssh-key-pair)
- [Add the public key to the VM](#add-the-public-key-to-the-vm)
  - [Verify that the correct key was added](#verify-that-the-correct-key-was-added)
  - [Disable Password-Logins](#disable-password-logins)
- [Install and start a web server](#install-and-start-a-web-server)
  - [Configuring an alternative start page](#configuring-an-alternative-start-page)
- [Connect the server to GitHub](#connect-the-server-to-github)
- [Simplify the login with an SSH client config](#simplify-the-login-with-an-ssh-client-config)

## Prerequisites
 
Before you start, you need:
 
- the IP address of your server
- a username and password for the server,

## First-Time Login

For the very first login, you use the username and password you received. Open PowerShell and run:

```bash
ssh <YOUR_USERNAME>@<YOUR_IP_ADRESS>
```
 
> [!NOTE]
> This command is just an example of how a login looks. Replace `your_username` and `your_ip_adress` with your own values. After running the command, you will be asked for your password. While typing the password, nothing appears on screen — this is normal, just type it and press Enter.

You are now logged in to your server. Type `exit` to return to your local machine.
 
> [!IMPORTANT]
> A password alone is not secure enough for a production server. In the next steps you will create an SSH key and use it instead of the password.

## Create an SSH-Key pair

An SSH key pair consists of two files: a **private key** (stays on your machine, never share it) and a **public key** (gets copied to the server). During login, the two keys are matched — no password needed.
 
```bash
ssh-keygen -t ed25519
```
 
`ssh-keygen` creates the key pair. The option `-t ed25519` selects the ED25519 algorithm, a modern alternative to RSA keys. Press Enter to accept the default file location (`C:\Users\<YOUR_USER>\.ssh\`). You can optionally set a passphrase to protect the key.
 
You now have two files in your `.ssh` folder:
 
- `id_ed25519` — your private key
- `id_ed25519.pub` — your public key

## Add the public key to the VM

Now copy the **public** key to the server, so the server knows it belongs to you. On Windows, run this in PowerShell:
 
```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh <YOUR_USERNAME>@<YOUR_IP_ADRESS> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```
 
What this command does: it reads your public key file (`type`) and sends it to the server via SSH, where it is appended to the file `~/.ssh/authorized_keys`. This file contains all public keys that are allowed to log in. You will be asked for your password one last time.
 
Now test the key login:
 
```bash
ssh <YOUR_USERNAME>@<YOUR_IP_ADRESS>
```
 
If everything worked, you are logged in **without** being asked for the server password (only the key passphrase, if you set one).

### Verify that the correct key was added
 
To check that exactly your key was added, compare the key on the server with your local public key.
 
On the server, show the authorized keys:
 
```bash
cat ~/.ssh/authorized_keys
```
 
On your local machine, show your public key:
 
```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```
 
The two outputs must match. Each key is one long line starting with `ssh-ed25519` and ending with a comment (usually `your_name@your_computer`), which helps you identify whose key it is.

### Disable Password-Logins

In this section information about disabling password-based logins will be provided.
Passwords are a potential security risk, which is why password logins should be disabled in favor of authenticating via SSH-Keys.

> [!WARNING]
> Make sure the SSH key login works **before** disabling password logins — otherwise you may lock yourself out of the server.

1. On the server, open the SSH configuration file and restart the SSH service afterwards (the second command runs automatically once you close the editor):

   ```bash
   sudo nano /etc/ssh/sshd_config && sudo systemctl restart ssh.service
   ```

1. Find and edit the line `#PasswordAuthentication yes` so that it reads 

    ```
   PasswordAuthentication no
    ```

1. Save and exit (`Ctrl+O`, Enter, `Ctrl+X`). The SSH service now restarts and loads the new configuration.

To verify, try logging in with password authentication forced:
 
```bash
ssh -o PubkeyAuthentication=no <YOUR_USERNAME>@<YOUR_IP_ADRESS>
```

## Install and start a web server

In this section, you will learn about running an `nginx` webserver on an Ubuntu cloud VM. After installing the webserver, you should be able to see the nginx server in your browser.

1. `apt update` refreshes the list of available software packages, so the newest version is installed.

   ```bash
    $ sudo apt update
    ```

1. `apt install nginx -y` installs nginx. The `-y` flag automatically confirms the installation, so you are not asked "Do you want to continue?".

   ```bash
    $ sudo apt install nginx -y
    ```

   nginx starts automatically after the installation. To test it, open your browser and go to:
 
    ```
    http://<YOUR_IP_ADRESS>
    ```
 
    You should see the default nginx welcome page ("Welcome to nginx!"). This confirms the web server is installed and running.

### Configuring an alternative start page

You should see the default nginx welcome page ("Welcome to nginx!"). This confirms the web server is installed and running.

Follow these steps to reconfigure the nginx installation on your cloud VM:

1. Create the HTML file `alternate-index.html` in that directory:
   ```bash
   sudo nano /var/www/alternatives/alternate-index.html
   ```
 
   You can use this snippet as content:
 
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

1. Create a new nginx configuration named `alternatives`:
   ```bash
   sudo nano /etc/nginx/sites-enabled/alternatives
   ```
 
   Add the following configuration. It tells nginx to listen on port `8081` and serve `alternate-index.html` from `/var/www/alternatives`:
 
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

1. Validate the configuration and restart nginx:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```
 
   `sudo nginx -t` tests the configuration for errors before applying it. If the test passes, the restart loads the new configuration.
 
1. Open your browser and go to:
   ```
   http://<YOUR_IP_ADRESS>:8081
   ```
 
   You should now see your own start page.
 
> [!IMPORTANT]
> Note that port Enter the IP address **together with the port number** in your browser, otherwise you will see the default nginx page instead of your own.

## Connect the server to GitHub

To clone and push Git repositories directly from your server, connect it to your GitHub account with a separate SSH key.
 
1. Install Git on the server and set your name and email. Use exactly the name and email of your GitHub account:

   ```bash
   sudo apt install git -y
   git config --global user.name "<YOUR_GITHUB_NAME>"
   git config --global user.email "<YOUR_GITHUB_EMAIL>"
   ```
 
2. Create a second SSH key pair — this time **on the server** — and display the public key:

   ```bash
   ssh-keygen -t ed25519
   cat ~/.ssh/id_ed25519.pub
   ```
 
> [!NOTE]
> This is a separate key pair from the one on your computer. The first key lets *you* log in to the server; this new key lets *the server* authenticate with GitHub.
 
3. Copy the output. Then go to GitHub → **Settings** → **SSH and GPG keys** → **New SSH key**, paste the key and save.

4. Test the connection from the server:

   ```bash
   ssh -T git@github.com
   ```
 
   Expected response: `Hi <name>! You've successfully authenticated...`
 
The server can now clone repositories via SSH, for example `git clone git@github.com:<YOUR_GITHUB_NAME>/<REPO>.git`.

## Simplify the login with an SSH client config
 
Instead of typing the full login command every time, you can define a short name for your server in the SSH client config. Afterwards, `ssh myserver` is enough to connect.
 
```powershell
$ code $env:USERPROFILE\.ssh\config
```
 
Add the following entry, using your own username and the path to your private key:
 
```
Host myserver
    HostName <YOUR_IP_ADRESS>
    User <YOUR_USERNAME>
    PreferredAuthentications publickey
    IdentityFile C:\Users\<YOUR_USER>\.ssh\id_ed25519
```

> [!CAUTION]
> The file must be saved **without a file extension** — its name is just `config`, not `config.txt`. Notepad may add `.txt` automatically; to prevent this, select "All files" as the file type when saving.
 
You can choose any name instead of `myserver` — just avoid spaces. No restart is required, the file is read on every `ssh` call. From now on, connect with:
 
```bash
ssh myserver
```