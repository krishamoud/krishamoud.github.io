---
layout: post
title: Deploying a MeteorJS app to ECS
excerpt: "Fast Deployment with Docker"
modified: 2015-07-04
tags: [docker, meteor, tutorial]
---

## Goals

Deploying a meteor app to ECS is actually extremely easy.  Here is what we have to do.

* Create a meteor app
* build it
* dockerize it
* push to docker hub
* deploy it

## Step 1: Create a new meteor app

Just run `meteor create ecs` and we'll use that for now.

## Step 2: Build it

cd into your `ecs` directory and run `meteor build ../build && cd ../build && tar -zxvf ecs.tar.gz && cd bundle && touch Dockerfile` 

This looks much more complicated than it really is.  First it is running the meteor build command to turn the meteor app into a vanilla node app.

Then it changes directories into the build directory which is where the build command put the node app.  It then unzips the tarball changes into the unzipped bundle and adds a dockerfile with nothing in it.

## Step 3: Dockerize it

Here is the dockerfile that I use to deploy to ecs.  It is very generic and will work with any meteor app as long as you followed the build command above.

{% highlight javascript %}
FROM    centos:centos6

# Enable EPEL for Node.js
RUN     rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
# Install Node.js and npm
RUN     yum install -y npm

# Bundle app source
COPY . /bundle
# Install app dependencies
RUN (cd /bundle/programs/server && npm install)


EXPOSE  80
CMD ["node", "/bundle/main.js"]
{% endhighlight %}

Save it and run your docker build command.

`docker build -t username/ecs .`

Now push it to the docker hub.

`docker push username/ecs`

## Step 4: Deploy it

Go to your AWS console and go to the EC2 Container Service link.

It should have a getting started button at the bottom.  If it doesn't then go to this link.

<https://us-west-2.console.aws.amazon.com/ecs/home?region=us-west-2#/getStarted>

(use your correct region if us-west-2 isn't your region)

It will ask if you want a sample of custom task.  Select custom and click next.

You should now be on step 2 of the `Getting Started with Amazon EC2 Container Service`

In the builder tab you should type in your task definition name.  I am going to use `ecs`.

Click `add container definition`

Give your container a name.  Again I'm going to use `ecs`

Give the image name that you pushed to the docker hub.  For me it is `khamoud/ecs`

Give CPU and Memory allocations.  This will depend on what you need but for this tutorial I'm going to put 300 for both `memory` and `CPU units`.

Make sure `essiential` is checked.

Under `port mappings` have both `Host Port` and `Container Port` be set to 80.

Scroll down until you see `environment variables`

Add one that is `MONGO_URL` and have it set to whatever your mongo url is.

Add another that is `ROOT_URL` and set it to your url.  If you don't have a domain name just put something in like `http://tempdomain.com` and it will still work.

Add a last one that is `PORT` and have it set to 80

In the footer of the modal click `add`

Click next step.

You should now be on Step: 3

Make sure that `desired number of tasks` is set to 1 and change the `service name` to whatever you want (cannot have multiple with the same name in the same cluster).

Make sure `Container: Port` is set to `No ELB`

Click next

You should now be on `Step 4: Configure cluster`

Set your number of instances to whatever you want.

Set your instance size to whatever you want.

Set your Key pair name to some `.pem` that you have access to.  
**This is important because if you don't have access to a .pem then you will not be able to ssh into the instance to troubleshoot if anything goes wrong**

Click `create IAM role` which will open a new tab.  Click allow in the bottom corner of the new tab.

Make sure that the new role you created is selected in the `ECS Instance Role` and click `Review and Launch`

Make sure that everything looks good in the review then click `Launch Instance and Run Service`.  In a few minutes if everything worked properly your meteor app will be deployed to your newly created instance and it will be publicly available on port 80.  

#Good Job!

