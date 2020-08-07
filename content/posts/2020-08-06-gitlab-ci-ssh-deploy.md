---
layout: post
title: "The complete tutorial on deploying your app using Gitlab CI and SSH"
summary: "It's update-the-knowledgebase-a-clock. I need to deploy an app from Gitlab CI to a remote server. Since all the articles I could find online are of the 'draw the rest of the owl' type, here's another one. I hope this one's complete. I use Ubuntu (because I like looking at ads when I log in)." 
date: 2020-08-06 12:00:18
comments: false
author: Viorel
category: techy
slug: 2020-08-06-gitlab-ci-ssh-deploy
tags:
- development
---

It's update-the-knowledgebase-a-clock. I need to deploy an app from Gitlab CI to a remote server. Since all the articles I could find online are of the "[draw the rest of the owl](https://knowyourmeme.com/memes/how-to-draw-an-owl)" type, here's another one. I hope this one's complete. I use Ubuntu (because I like looking at ads when I log in).

*Before we start, I'd like to note that you should SSH only if you need to run commands on the remote server. If you're just copying files around, please turn around and look for a SFTP tutorial.*

**Description of the problem:** the code for my application is on a private Gitlab server. When I push code to the master branch, I want the app to be compiled and deployed to a separate server. 

Assumptions: 

1. You already set up Gitlab CI and there is a runner running. 
2. The Ubuntu Server is up and running and you have access to it.
3. This tutorial is agnostic in the sense that it doesn't care how the app should actually run on the server. It just takes the file(s) there.

## A new user on the server

Your app is going to be copied somewhere on the server. My friendly suggestion is *to have this user only have access to that particular folder.*

Let's call our user *lenny_deployman* and let's assume you want to deploy your app in `/home/myapp/www`.

Creating the user: `sudo adduser myapp`

Add the user to the list of allowed SSH users:

`sudo nano /etc/ssh/sshd_config` and then go to the `AllowUsers` line and add your user. It might look something like this: `AllowUsers myapp root`. Restart the service: `sudo systemctl restart sshd` 

## Consider locking your user in a Chrooted jail

What we're doing here is basically creating a user with SSH access to our server, creating a private/public key pair and setting that in Gitlab so Gitlab can connect to our server and do its thing there. This makes a lot of people uncomfortable, myself included. You have to ask yourself, what happens if my Gitlab server gets hacked or if I lose my database. So please consider restricting the user to a Chrooted Jail, [as explained in this article](https://www.tecmint.com/restrict-ssh-user-to-directory-using-chrooted-jail/). I'll consider that out of scope for this tutorial, as it's nicely explained in the link.

## Connect to the server using a passwordless SSH login

Go to **your computer** and open up a terminal. Not on the server, on your computer. To make this tutorial as OS agnostic as possible, I'm using the WSL Ubuntu shell in Windows 10.

We're now going to create the public/private keypair on your local computer. Since we're creating this public/private key for our own use, let's just create a separate user for what's about to happen. We'll delete it afterwards. That way we make sure this key is dedicated and won't be copied anywhere else.

`sudo adduser lenny_deployman`

Log in as this user: `sudo su lenny_deployman`

And now let the fun begin:

1. Generate the keys: `ssh-keygen -t rsa -b 4096`. We're using RSA here. It's not the best, not the worst, but it's backwards compatible, so let's choose the least headache at this point.
    1. Press "Enter" when asked for the file where to save the key
    2. **Important!** Press "Enter" when asked for the passphrase, so the passphrase is empty. This [is the recommended way to go in the Gitlab docs](https://docs.gitlab.com/ee/ci/ssh_keys/), otherwise the script will ask for a password. There are ways to also save the passphrase in a Gitlab variable and then passing it to the script, but they're cumbersome and IMHO the benefits do not match the effort.
2. Copy the key to the remote server: `ssh-copy-id myapp@server-ip`. You will be prompted for myapp's password.
3. Test the SSH connection: `ssh myapp@server-ip`. If everything went OK, you should get logged in automatically to the server and in your home folder.

## Gitlab's next

So what we're basically doing now is copying the private key we have on our computer to Gitlab and making it use it to deploy the app to our server. Connecting to the server via a key works like this: you create a pair of keys that are linked to each other: a private one on your computer, and a public one supposed to sit on the server. When you make the connection, the SSH command sends the private key to the server, the server checks which public keys are authorized for that particular user and sees if they match. If they do, you're in.

What this means is that we have to give Gitlab our private key, so it can use it to connect to the server.

First things first: let's create a variable in Gitlab and keep our private key there: go to *Settings > CI / CD > Variables* and press *Add Variable.* Name the variable `SSH_PRIVATE_KEY` and in the value field paste the value of your private key. You can get this value by running `cat ~/.ssh/id_rsa` in the local terminal and copying the output.

Since you're there already, go ahead and add a variable called `SERVER` that contains the IP of your server and one for the `PORT`. We're going to use those to connect.

I'm just going to paste my whole .gitlab-ci.yml file next. What it does it that it takes my files, runs npm install, runs webpack build and then deploys the app. I've commented what each line does in the file.

```yaml
image: node:lts-alpine # the docker image we're going to use to run commands. It's Linux Alpine with node installed

stages: # we've split the CI job in two parts
    - package # this one builds the app
    - deploy # this one deploys it

app-package:
    tags:
        - linux # that's just my runner
    stage: package # which stage are we in?
    script: # the scripts I'm running in this stage (they're run by the docker image)
        - npm install
        - npm run webpack:prod
    artifacts: # what I want to save for the next step
        paths:
            - node_modules
            - build/resources/main/static
        expire_in: 1 day
    only: # run only on this branch
        - master

app-deploy:
    tags:
        - linux
    stage: deploy
    only:
        - master
    script:
        - apk update # Update existing packages definitions in Alpine
        - 'which ssh-agent || ( apk add openssh-client )' # see if ssh-agent is installed and install it if not
        - eval $(ssh-agent -s) # start the ssh agent
        - echo "$SSH_PRIVATE_KEY" | ssh-add - # add your private key to the agent
        - mkdir -p ~/.ssh # emulate normal Linux SSH behaviour in Docker
        - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
        - scp -P"$PORT" -r build/resources/main/static myapp@"$SERVER":~/tmp # copy the files to the server
        - ssh -o StrictHostKeyChecking=no myapp@"$SERVER" -p "$PORT" 'rm -rf ~/tmp_old && mv ~/www ~/tmp_old && 
          rm -rf ~/www && mv ~/tmp ~/www && rm -rf ~/tmp' # replace the old files with the new files
	  
        # - whatever else you need
```

## Some cleanup

I don't want to have the copy of the private key on my computer, that's why I created it separately. Clean everything up by doing `sudo deluser lenny_deployman && sudo rm -rf /home/lenny_deployman`.

These articles where extremely helpful in figuring everything out:

1. [https://www.cyberciti.biz/faq/ubuntu-18-04-setup-ssh-public-key-authentication/](https://www.cyberciti.biz/faq/ubuntu-18-04-setup-ssh-public-key-authentication/)
2. [https://www.linuxbabe.com/linux-server/setup-passwordless-ssh-login](https://www.linuxbabe.com/linux-server/setup-passwordless-ssh-login)
3. [https://medium.com/@hfally/a-gitlab-ci-config-to-deploy-to-your-server-via-ssh-43bf3cf93775](https://medium.com/@hfally/a-gitlab-ci-config-to-deploy-to-your-server-via-ssh-43bf3cf93775)
