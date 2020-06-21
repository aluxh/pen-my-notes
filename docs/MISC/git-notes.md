# Git notes

* [How to git push to an AWS EC2 remote using a PEM file](#How-to-git-push-to-an-AWS-EC2-remote-using-a-PEM-file)
* [How to deploy your project with a git push on your EC2 instance](#How-to-deploy-your-project-with-a-git-push-on-your-EC2-instance)

## How to git push to an AWS EC2 remote using a PEM file

I learn the following from [alphacoder](https://alphacoder.xyz/git-push-to-an-aws-ec2-remote-using-a-pem-file/)

AWS provides a .pem file when creating EC2 instance. We can use this file to generate SSH keys for accessing the instance without using the PEM file.

To begin, you should use Git Bash if you are using Windows 10, like I do

### Copy private key to .ssh folder

```
$ cp /path/to/aws-ec2-instance.pem ~/.ssh/id_rsa_ec2
```

### Generate and save public key

Go to ~/.ssh folder, and run

```
$ ssh-keygen -y -f aws-ec2-instance.pem > ~/.ssh/id_rsa_ec2.pub
```

### Add private key to ssh-agent

Start ssh-agent

```
$ eval "$(ssh-agent -s)"
```

Then, add your key to the agent

```
$ ssh-add ~/.ssh/id_rsa_ec2
```

After that you can access your instance without the PEM file

```
$ ssh ubuntu@ec2-ip-address
```

## How to deploy your project with a git push on your EC2 instance

I learn the following from [kirankoduru.github.io](https://kirankoduru.github.io/git/setup-git-on-your-server.html)

Using the notes from the website, I can host codes on Github, then deploy the code to server by doing git push to EC2 instance. This makes the deployment fast and simple.

To begin, you should setup the [git push to AWS EC2 using PEM file](#How-to-git-push-to-an-AWS-EC2-remote-using-a-PEM-file)

### Login to your EC2 machine through ssh

```
$ ssh ubuntu@ec2-ip-address
```

### Setup a git bare repo

This folder/repo will hold the code when you git push to EC2. For example, I'm creating a *test* project, and the bare git repo is **test.git**. And, in this example, I'm going to store my project in my home folder: **/home/ubuntu/**

```
sudo mkdir test.git
cd test.git
git init --bare
```

### Create a directory where you will deploy your project

Go to the **/home/ubuntu/**, and create the project folder

```
sudo mkdir test
```

### Setup Git Hook

After the above is done, go to the **test.git** folder, and create a file **post-update** using **post-update.sample**.

```
cd test.git
cd hooks
mv post-update.sample post-update
```

Then, edit the file

```
vim post-update
```

In the file, you should include the following:

```
#!/bin/sh
GIT_WORK_TREE=/home/ubuntu/test/ git checkout -f
```

### Setup the git remote after the above are completed

Logout from the server, and go to your local git repo for project test, and add a remote

```
git remote add production ubuntu@ec2-ip-address:/home/ubuntu/test.git
```

Then, you are all done. You can push master branch to remote production and checkout the /home/ubuntu/test directory for the files.

```
git push production master
```

### In case your push fails because of permission

```
λ git push production master
Enumerating objects: 176, done.
Counting objects: 100% (176/176), done.
Delta compression using up to 4 threads
Compressing objects: 100% (172/172), done.
Writing objects: 100% (176/176), 660.67 KiB | 4.00 MiB/s, done.
Total 176 (delta 89), reused 0 (delta 0)
error: remote unpack failed: unable to create temporary object directory
To ec2-ip-address:/home/ubuntu/test.git
 ! [remote rejected] master -> master (unpacker error)
error: failed to push some refs to 'ubuntu@ec2-ip-address:/home/ubuntu/test.git'
```

You can go to the folder in EC2 and give permission using `chmod`. I'm not very good at this yet, so I use `777` for the folder

```
chmod -R 777 test.git
```
