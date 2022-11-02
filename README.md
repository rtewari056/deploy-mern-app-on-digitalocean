# Deploying MERN Stack App on DigitalOcean
In this tutorial, we'll deploy a MERN Stack application on a DigitalOcean server in the cloud with SSL/HTTPS encryption and a custom domain.

Furthermore, Nginx will be used as a reverse proxy and Let's Encrypt and Certbot will be used to configure SSL/HTTPS encryption for your application.

When the Node.js application is running on the server, we'll use PM2 to keep the application running forever without any downtime.

### Let's get started!

## Step 1 - Create & Configure A New DigitalOcean Server

Before we can do anything, we need to create and configure a VPS (Virtual Private Server) in the cloud to host our application.

### Create New Droplet On DigitalOcean

After logging in or successfully signing up for a new account, open the Create drop-down menu and click the Droplets link.

In the first section, select the Ubuntu operating system for your server.

![Choose an image](./assets/choose_an_image.png)

Then, choose the $6 per month standard plan, which will give your application plenty of computing power to start with. You can easily upgrade in the future if needed.

![Choose a plan](./assets/choose_a_plan.png)

Next, they allow you to choose the datacenter region for your server. This is the physical location for your server and, therefore, you should choose the one closest to the people visiting your website.

![Choose a datacenter region](./assets/choose_a_datacenter_region.png)

In the Authentication section, make sure the SSH keys option is selected for more security and passwordless login to your server.

![Authentication](./assets/authentication.png)

### Setting up SSH in DigitalOcean Droplet

Click on the `New SSH Key` button and a pop up will open to add a public SSH key.

![New SSH Key](./assets/new_ssh_key.png)

You need to generate an SSH key on your local machine to login to your server remotely. Open your terminal and type:

```bash
$ ssh-keygen
```

By default, it will create your public and private key files in the .ssh directory on your local machine and name them **id_rsa** and **id_rsa.pub**. You can change this if you want, just make sure when it asks, you put the entire path to the key as well as the filename. I am using **digital_ocean_key**.

Once you do that, you need to copy the public key. You can use the `cat` command and then copy the key

```bash
$ cat ~/.ssh/digital_ocean_key.pub
```

Copy the key. It will look something like this:

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDEwMkP0KHX19q2dM/9pB9dpB2B/FwdeP4egXCgdEOraJuqGvaylKgbu7XDFinP6ByqJQg/w8vRV0CsFXrnr+Lh51fKv8ZPvV/yRIMjxKzNn/0+asatkjrkOwT3f3ipbzfS0bsqfWTHivZ7UNMrOHaaSezxvJpPGbW3aoTCFSA/sUUUSiWZ65v7I/tFkXE0XH+kSDFbLUDDNS1EzofWZFRcdSFbC3zrGsQHN3jcit6ba7bACQYixxFCgVB0mZO9SOgFHC64PEnZh5hJ8h8AqIjf5hDF9qFdz2jFEe/4aQmKQAD3xAPKTXDLLngV/2yFF0iWpnJ9MZ/mJoLVzhY2pfkKgnt/SUe/Hn1+jhX4wrz7wTDV4xAe35pmnajFjDppJApty+JOzKf3ifr4lNeZ5A99t9Pu0294BhYxm7/mKXiWPsevX9oSZxSJmQUtqWWz/KBVoVjlTRgAgLYbKCNBzmw7+qdRxoxxscQCQrCpJMlat56vxK8cjqiESvduUu78HHE= rtewa@DESKTOP-HQRNFTC
```

Now paste that into the textarea and name it (eg. digital_ocean_key)

![Configure SSH Key](./assets/configure_ssh.png)

Also, you can choose a hostname for your server. This will give your server a name to remember it by.

![Finalize and Create](./assets/finalize_and_create.png)

When you're done selecting options, hit the Create Droplet button to kick off the creation of your new server.

![Create Droplet](./assets/create_droplet.png)

Your server is now up and running!

![Create Droplet](./assets/droplet.png)

In the next step, we'll complete the initial configuration process for the server. This will include logging into the server, setting up SSH access to the server, and creating a basic firewall.

### Access The Server Using Root

The first step is to gain access to the server using your root login.

To log into your server, open a terminal on your local machine and use the following command to SSH in as the root user (replace `SERVER_IP_ADDRESS` with your server's public IP address and `PRIVATE_KEY` with the private key file name):

```console
ssh -i ~/.ssh/PRIVATE_KEY_FILE_NAME root@SERVER_IP_ADDRESS
```

Accept the warning about host authenticity, if it appears.

The `root` user in a Linux environment has very broad privileges and, for that reason, you are discouraged from using it on a regular basis.

Therefore, in the next step, we are going to create an alternative account with limited scope that will be used for daily work.

### Create A New User

Right now you are logged in as `root` and it is a good idea to create a new account.

You can check your current user with the command:

```console
root@hostname:~# whoami
```

It will say `root` right now.

Let's add a new user. I am going to call my user `rohit`

```console
root@hostname:~# adduser rohit
```

Just hit enter through all the questions. You will be asked for a use password as well. You can just hit `ENTER` repeatedly to skip the rest of the questions.

![Create a New User](./assets/create_a_new_user.png)

### Give Your New User Root Privileges

You now have a new user account with regular account privileges. But you might occasionally need to do administrative tasks that require `root` privileges.

To do this, we need to add your new user to the `sudo` group on the machine.

As `root`, run the following command to add your user to the `sudo` group (substitute `rohit` with your username):

```console
root@hostname:~# usermod -aG sudo bob
```

Now your user can run commands with `root` privileges!

### Add SSH keys for new account

Assuming you generated an SSH key pair. Use the following command at the terminal of your local machine to print your public key (`digital_ocean_key.pub`):

```bash
$ cat ~/.ssh/digital_ocean_key.pub
```

Copy the key. It will look something like this:

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDEwMkP0KHX19q2dM/9pB9dpB2B/FwdeP4egXCgdEOraJuqGvaylKgbu7XDFinP6ByqJQg/w8vRV0CsFXrnr+Lh51fKv8ZPvV/yRIMjxKzNn/0+asatkjrkOwT3f3ipbzfS0bsqfWTHivZ7UNMrOHaaSezxvJpPGbW3aoTCFSA/sUUUSiWZ65v7I/tFkXE0XH+kSDFbLUDDNS1EzofWZFRcdSFbC3zrGsQHN3jcit6ba7bACQYixxFCgVB0mZO9SOgFHC64PEnZh5hJ8h8AqIjf5hDF9qFdz2jFEe/4aQmKQAD3xAPKTXDLLngV/2yFF0iWpnJ9MZ/mJoLVzhY2pfkKgnt/SUe/Hn1+jhX4wrz7wTDV4xAe35pmnajFjDppJApty+JOzKf3ifr4lNeZ5A99t9Pu0294BhYxm7/mKXiWPsevX9oSZxSJmQUtqWWz/KBVoVjlTRgAgLYbKCNBzmw7+qdRxoxxscQCQrCpJMlat56vxK8cjqiESvduUu78HHE= rtewa@DESKTOP-HQRNFTC
```

Select the public key, and copy it to your clipboard.

To enable the use of the SSH key to authenticate as the new remote user, you must add the public key to a special file called `authorized_keys` in the user's home directory.

**On your DigitalOcean server** and as the `root` user, enter the following command to temporarily switch to the new user (substitute `rohit` with your username):

```console
root@hostname:~# su - rohit
```

Now you will be in your new user's home directory.

Create a new directory called `.ssh` and restrict its permissions with the following commands:

```console
rohit@hostname:~$ mkdir ~/.ssh && chmod 700 ~/.ssh
```

Now create a new file in `.ssh` called `authorized_keys` with a text editor. We'll use `nano` to edit the file:

```console
rohit@hostname:~$ nano ~/.ssh/authorized_keys
```

Now insert your public key (which should be in your clipboard) by pasting it into the editor.

Hit `CTRL-X` to exit the file, then `Y` to save the changes that you made, then `ENTER` to confirm the file name.

Now restrict the permissions of the `authorized_keys` file with this command:

```console
rohit@hostname:~$ chmod 600 ~/.ssh/authorized_keys
```

Type this command once to return to the `root` user:

```console
rohit@hostname:~$ exit
```

Now your public key is installed, and you can use SSH keys to log in as your user.