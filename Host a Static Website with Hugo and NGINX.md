---
author:
  name: Linode Community
  email: docs@linode.com
description: Guide for hosting Hugo on Ubuntu 16.04 with Nginx as reverse proxy.
keywords: ["Hugo", "Ubuntu 16.04", "Nginx", "Reverse proxy"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2017-12-30
modified: 2017-12-30
modified_by:
  name: Linode
title: Host a Static Website with Hugo and NGINX
contributor:
  name:Patrick Allison
  link: '[GitHub](https://github.com/jaminyah)'
external_resources:
  - '[Hugo Getting Started](https://gohugo.io/getting-started/)'
  - '[Nginx docs](http://nginx.org/en/docs/)'
  - '[Running Hugo in the background](https://discourse.gohugo.io/t/run-server-as-background/3252)'
  - '[Setting appendPort and baseUrl](https://github.com/gohugoio/hugo/issues/852)'
  - '[Hugo styling not working](https://discourse.gohugo.io/t/hugo-static-styling-not-working-whereas-hugo-server-works/2916)'
  - '[Reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy)'
  - '[How to Develop and Deploy Your Applications Using Wercker](https://linode.com/docs/development/ci/how-to-develop-and-deploy-your-applications-using-wercker/)'
  - '[Deployment with Wercker](http://gohugo.io/hosting-and-deployment/deployment-with-wercker)'
  - '[Creating a Workflow](http://devcenter.wercker.com/docs/creating-a-workflow)'
  - '[Deployment with Rsync](https://gohugo.io/hosting-and-deployment/deployment-with-rsync/)'
---

Hugo is promoted as "...one of the most popular open-source static site generators." In this article we will discuss installing 
[Hugo](https://gohugo.io/) on Ubuntu 16.04, then discuss installing and configuring [Nginx](http://nginx.org/) as a reverse proxy. We will begin by pointing your registered domain name to Linode name servers. The ultimate goal is to enter a URL such as `http://blog.example.com` in your web browser to display your Hugo blog articles hosted on Linode.


## Before You Begin

{{< note >}}
The steps in this guide require root privileges. Be sure to run the steps below as `root` or with the `sudo` prefix. For more information on privileges, see our [Users and Groups](/docs/tools-reference/linux-users-and-groups) guide.
{{< /note >}}

{{< note >}}
Throughout this guide commands will be given with example variables to provide general instructions for all readers. Replace each example variable in each command as described in the table below.
{{< /note >}}

| Example Variable         | Action to Take                                        |
|--------------------------|-------------------------------------------------------|
| `http://blog.example.com`| Replace with the subdomain for your website address   |
| `http://example.com`     | Replace with your website address                     |
| your-template-name       | Replace with the Hugo theme that you downloaded       |
| your-domain-name         | Replace with the domain name for your website         |
| username                 | Replace with your Linode login user name              |
| `linode-IP-Address`      | Replace with IP address of your Linode server         |
| static-site-name         | Replace with the name of your Hugo static website     |
| github-username          | Replace with your GitHub user name                    |


Begin by completing the following guides if you are getting started with Linode for the first time.

1.  Familiarize yourself with our [Getting Started](/docs/getting-started) guide and complete the steps for setting your Linode's hostname and timezone. The guide explains how to sign-up, deploy a Ubuntu 16.04 image and connect to your Linode server using the secure shell protocol (SSH). For the purpose of installing Hugo and configuring Nginx, it is not necessary to do the "Setting the Hostname" steps.

2.  This guide will use `sudo` wherever possible. Complete the sections of our [Securing Your Server](/docs/security/securing-your-server) to create a standard user account, harden SSH access and remove unnecessary network services.

3.  A software version control platform is needed to manage the source code of your static website. In this guide [GitHub](https://github.com/) is discussed. Other version control platforms such as [Bitbucket](https://bitbucket.org/) or [GitLab](https://gitlab.com/) can be used. However, those platforms are not discussed in this guide. Sign up for a GitHub account if you don't have one.


### Ubuntu Installation Update

Begin by updating Ubuntu if you have an existing Ubuntu installation and have previously completed the two steps above.

    sudo apt-get update


## Guide Overview

This guide provides instructions on developing a static website, with Hugo static site generator, and building and deploying this static site to Linode production servers. When development and testing is completed locally, the site can be built, deployed to production, and kept updated several ways. In this guide two cases are considered. The first case discusses running Hugo server in production with Nginx as reverse proxy. The second case discusses using Nginx to serve the static site. A brief discussion of the major sections of this guide is presented next.

*  __Point Domain name to Linode Servers__ - Provides instructions for pointing a registered domain name to your Linode server. It also includes steps for adding a `blog` subdomain.

*  __Static Site Generator Principles__ - Having a big picture understanding of the general concepts of static site generators can be very useful to understanding what needs to be done in order to get a static site up and running on a production server. 

*  __Development Environment__ - Getting started with Hugo can be a challenging task. Creating a static site with Hugo begins with installing the Hugo application locally on a computer, such as a desktop or laptop. This section provides references to instructions on installing Hugo locally and creating the first blog post.

*  __Deployment Environment: Case I__ - In case I, the development files of the static site are deployed to the production server, without a build pipeline. Two techniques for transferring the static site to production are presented. First is using Rsync to deploy and update the static site on the production server. Second is using Wercker, continuous integration tool, for automated deployment of the static site to the production server.

*  __Production Environment: Case I__ - In this case, Nginx is configured to reverse proxy a Hugo server that is running in production. The site is available at URL `http://blog.example.com` .

*  __Deployment Environment: Case II__ - In case II, when development is completed locally and all the development files are checked into GitHub, build and deploy pipelines are created with Wercker. Wercker build pipeline generates the static files of the site in a folder named public. Wercker deploy pipeline, copy the static files to a location specified on the production server.

*  __Production Environment: Case II__ - Nginx is configured to serve the static files generated by Wercker. The site is available at URL `http://coder.example.com` .

*  __Learning more about Hugo__ - This section includes references to video tutorials and other resources that can be used to gain further practical understanding of using Hugo.


## Point Domain name to Linode Servers

There are two aspects to pointing your registered domain name to your Linode server. 
1. Linode DNS Manager needs to be configured with your domain name.
2. The domain registrar from which your domain name was purchased/registered needs to be configured with the Linode name servers.

Configuration steps for the Linode side can be found at [DNS Manager Overview](/docs/networking/dns/dns-manager-overview/). A general overiew of [DNS Records](/docs/networking/dns/dns-records-an-introduction/) is also available.

### Set Linode Name Servers on Web Hosting

The typical scenario is that you purchased and registered a domain name through one of the popular providers such as Bluehost, Godaddy, or Fatcow. Now, you need to point the domain name to your Linode server. Navigate to the Control Panel on your provider's Control Panel.

We will use Fatcow's Control Panel in this example.

<p align="center">
  <img src="/images/ControlPanel.jpg" alt="Control Panel" /> 
</p>

Click on the link that lists all the domains you have registered. In this case I'll click on DomainCentral located in the Domain Tools section.

<p align="center">
  <img src="/images/MyDomains.jpg" alt="Domain Listing" /> 
</p>

From the list of domains select the one you want to point to your Linode server. On selecting that domain name, an overview of various actions that can be taken on the domain name will be presented.

<p align="center">
  <img src="/images/Overview.jpg" alt="Overview of possible actions" /> 
</p>

In this case, I'll select Points to URL icon so as to enter the IP address of my Linode server.

<p align="center">
  <img src="/images/Pointers.jpg" alt="Point URL to Linode" /> 
</p>

Now that the URL is pointing to the IP address of your server on Linode, the Linode name servers need to be added.

<p align="center">
  <img src="/images/Nameservers.jpg" alt="Name servers" /> 
</p>

The following name server entries should be made:
* ns1.linode.com
* ns2.linode.com
* ns3.linode.com
* ns4.linode.com
* ns5.linode.com

The above is given as an example. Follow the instructions for your specific domain provider.


### Adding Blog Subdomain

One to the things you may want to do is add a subdomain to your domain and have this URL point to your blog articles. For example if your domain is `http://example.com` adding a blog subdomain creates `http://blog.example.com`. Navigate to [linode.com](https://www.linode.com/) and log into your Linode server account. Select the DNS Manager tab.

<p align="center">
  <img src="/images/DNSManager.jpg" alt="DNS Manager" /> 
</p>

DNS Manager lists domains linked to your Linode account. Let's assume that domains example.com and example.net are associated with this Linode account. Select the first domain in the list and focus on the section titled A/AAAA Records.

<p align="center">
  <img src="/images/AAAA.jpg" alt="AAAA records" /> 
</p>

In A/AAAA Records select the link "Add a new A/AAAA record".  For Hostname add `blog.example.com`. In the IP Address field enter the IP address of your Linode server. 

<p align="center">
  <img src="/images/EditAAAA.jpg" alt="Editing AAAA records" /> 
</p>

{{< note >}}
Your IP address can be found by selecting the Linodes tab. 
{{< /note >}}

Replace example.com with the domain name that was selected in the Domain Zone column of the DNS Manager tab. Use the `Default` setting for the TTL item. Save all changes to A/AAAA Record. Subdomain `blog` should now appear in the list of A/AAAA Records. 


### Logging into Linode Server with SSH

Open a Terminal window and log in to your Linode server at the command line with SSH. A typical log in command should look as follows:

    ssh username@linode-IP-Address

{{< note >}}
Enter the `username` that you use to log into your Linode server. This user must have `sudo` privileges.
Enter the `IPAddress` of your Linode server.
{{< /note>}}

Further clarification on logging in with SSH can be found in the [Getting started](/docs/getting-started) guide in the section titled "*Log In for the First Time*". 


## Static Site Generator Principles

*** TBD **


## The Development Environment: Case I

If you are getting started with creating static sites with Hugo, the best source of accurate and updated instructions is the Hugo site [Getting Started Quick Start Guide](https://gohugo.io/getting-started/quick-start/). Instructions begin at Step 2: *Create a New Site*. It is important to note that development and unit testing of the static website is performed on your local development machine. 

<p align="center">
  <img src="/images/sw_dev_sequence.jpg" alt="Software development sequence" /> 
</p>

Once code review and system testing are complete, it is then time to check the code into a version control system, such as GitHub. Build and deploy phases are performed next followed by release to production.


### Building and Updating a Static Site

Generally, developers focus on writing code, unit testing and incorporating code review changes. For the most part, many developers shy away from the inner details of deploying applications to the production environment. This task is usually handled by the DevOps team or backend crew. In this section, we will discuss moving the static site from the development environment to the production server.

<p align="center">
  <img src="/images/ProductionCycle.jpg" alt="Production Cycle" /> 
</p>

There are several approaches to deploying a static site, after it has been through the development cycle. [Travis-CI](https://travis-ci.org), [Jenkins](https://jenkins.io), Wercker and [Rsync](https://rsync.samba.org) are among the more popular tools, used to deploy to a production server such as Linode. Let's first discuss using `Rsync`.


## The Deployment Environment: Case 1

### Deploy with Rsync

You may decide that yours is a small operation and so you develop and test on your local machine as well as use a remote version control service such as GitHub to backup your local files. Then you wish to transfer the files of your static site directly to the Linode production server without an automated build and deploy tool. If this is the case, Rsync is a good tool for manually syncing your files with the production server.

<p align="center">
  <img src="/images/sw_dev_rsync.jpg" alt="Deploy with Rsync" /> 
</p>

Rsync is described in the [Wikipedia](https://en.wikipedia.org/wiki/Rsync) as a tool for "transferring and synchronizing files across computer systems." Consider a static website, mysite, on a local development machine. It is in the home directory with the following folder structure:

```bash
|__public
   |__sites
   |____mysite
      |_____index.html
      |_____css\style.css
      |_____js\script.js
```

Once all bugs are fixed, the website works as expected, and all files are checked into version control, then it's time to deploy the static website to the Linode production server. We can push the contents of the public directory to the production server home directory with the command `rsync -zP public username@linode-IP-Address:~/`.

```bash
                             rsync -zP public username@linode-IP-Address:~/
                                    ||   |                         |      | 
                          compress _||   |                         |      |__ Linode home directory
                    show progress ___|   |                         |__ Linode server IP address
                static site directory ___|
```

Here is a sample Terminal console output.

```bash
machine-name:~ username$ ls
public

machine-name:~ username$ rsync -zP public username@linode-IP-Address:~/
username@linode-IP-Address password: 
building file list ... 
12 files to consider
public/
public/.DS_Store
        6148 100%    0.00kB/s    0:00:00 (xfer#1, to-check=10/12)
public/sites/
public/sites/.DS_Store
        6148 100%  857.70kB/s    0:00:00 (xfer#2, to-check=8/12)
public/sites/mysite/

sent 1688 bytes  received 188 bytes  288.62 bytes/sec
total size is 19090  speedup is 10.18
machine-name:~ username$ 

```

Logging into your Linode server with the SSH command, at the Terminal console and using the `ls` command, should show the "public" directory. Included in this directory are the contents of the static site.

```bash
username@localhost:~$ ls
Downloads  hugo  public
username@localhost:~$ cd public/
username@localhost:~/public$ ls -R
.:
sites

./sites:
mysite

```

### Build and Deploy with Wercker 

A much better alternative to Rsync is a continuous integration, build automation tool. One such tool is [Wercker](http://www.wercker.com/). At present Wercker will build and deploy static site contents from GitHub, BitBucket and GitLab. Let's use GitHub. 


#### Creating a Wercker App

Begin by signing up for an account on the Wercker site using the "Get Started for free" option. Signing up using GitHub is the simplest option. 

<p align="center">
  <img src="/images/wercker/login.jpg" alt="Wercker" /> 
</p>


You will be greeted with a Welcome page, from which the first application can be created. 

<p align="center">
  <img src="/images/wercker/welcome.jpg" alt="Welcome" /> 
</p>


Create the application by selecting an owner and GitHub. The owner is your GitHub user name.

<p align="center">
  <img src="/images/wercker/create_new_app.jpg" alt="Create New Application" /> 
</p>


From the list presented, select the GitHub repository that has the contents of your static site.

<p align="center">
  <img src="/images/wercker/select_repo.jpg" alt="Select GitHub Repo" /> 
</p>

Review the status of the Wercker application, then create it.

<p align="center">
  <img src="/images/wercker/review.jpg" alt="Review Application" /> 
</p>


After creating the application, several options to select a programming language are presented. Explore a few language options to see the wercker.yml template that is created for each. Later in this guide, we will create a wercker.yml file with the default option. The Wercker yaml file will be placed in the static website directory on your local development machine. Then it will be pushed to GitHub. Now it's time to switch focus to creating workflows.

{{< note >}}
An option to trigger a build is available on this Wercker page. Do not trigger a build at this time since the wercker.yml file needs to be created.
{{< /note >}}

<p align="center">
  <img src="/images/wercker/nicely_done.jpg" alt="Nicely done" /> 
</p>


#### Creating Workflows

Let's examine the structure of the default wercker.yml file that will be added to your static site directory. Note that there is one pipeline shown; deploy.

```bash
box: debian

deploy:
  steps:
 
```

Wercker application, therefore, only has to have one pipeline in its workflow. By default, a build pipeline should already be created in the workflow. Let's change the existing build pipeline to match the deploy pipeline in our wercker.yml file. Begin by selecting the Wercker Workflows tab. Select the build pipeline in the list below the "Add new pipeline" button. Change the name to deploy-production and the YML Pipeline to deploy. 

  * __Name__: deploy-production
  * __YML Pipeline name__: deploy


<p align="center">
  <img src="/images/wercker/deploy_production_pipeline.jpg" alt="Deploy production pipeline" /> 
</p>

The workflow now has a deploy pipeline that corresponds with the deploy pipeline in the wercker.yml file. Note that the wercker.yml file will be created shortly.


#### Creating SSH Keys

Wercker will need to access your Linode server. To accomplish this task, the Wercker app will be used to generate a Public and Private Key pair. Select the Environment tab, then select the `Generate SSH Keys` option. For SSH key name, enter linode and for RSA select 4096 bit. Select `Generate` button.

  * __SSH key name__: linode
  * __RSA__: 4096 bit

<p align="center">
  <img src="/images/wercker/ssh_key.jpg" alt="Create SSH" /> 
</p>

Successful generation of the key-pair should show the public key, `linode_PUBLIC`. The private key will not be visible.

<p align="center">
  <img src="/images/wercker/linode_public.jpg" alt="Linode public key" /> 
</p>


#### Add Public Key to Linode User Account

Log into your Linode server with the Terminal console using the `ssh` command and enter your password.

` ssh username@linode-IP-Address`

{{< note >}}
username should have sudo privileges but for security purposes the username should not be `root`.
{{< /note >}}

Wercker app public key now needs to be added to the authorized_keys file in your Linode home directory. Let's check if there is already a `.ssh` directory by entering the command `ls -al`. There are three cases to consider:

1. There is a .ssh directory with no authorized_keys file
2. There is no .ssh directory
3. There is a .ssh directory with an authorized_keys file

| Command Name             | Action performed                                       |
|--------------------------|--------------------------------------------------------|
| `cd .ssh`                | Change into the .ssh directory                         |
| `ls -al`                 | Provide a long listing of files including hidden files |
| `ls`                     | List all files in the current directory                |


  * case 1: If there is a .ssh directory, change into it with the `cd .ssh` command. List the files with the command `ls` to check if there is an authorized_keys file. If there is no authorized keys file, continue with the steps in "Add Authorized Keys File" subsection.

  * case 2: A .ssh directory needs to be added. Add this directory with the command `mkdir .ssh`. Change into this directory with command `cd .ssh`. Continue with the steps in "Add Authorized Keys File" subsection.

  * case 3: Since there is already a .ssh directory and an authorized keys file, continue with the steps in "Copy the Public Key to authorized_keys" subsection.

##### Add Authorized Keys File

If there isn't one, create a blank file with the command, `vim authorized_keys`. Press the `i` key on your keyboard to enter "Insert" mode. Continue with the steps in "Copy the Public Key to authorized_keys" subsection.

##### Copy the Public Key to authorized_keys

Copy the `linode_PUBLIC` key from the Wercker app environment and paste it into the authorized_keys file. On your keyboard, press the **Escape** button to exit vim Insert mode. Save and exit the file by typing `:wq`. Complete the command with the **Return** key. 

{{< note >}}
The username, used to log into Linode and add the SSH public key, will be used in the scripts section of the wercker.yml file.
{{< /note >}}


#### Creating wercker.yml

Create a file, wercker.yml, on your local machine in the directory containing your static website. One option is to create the file at the command line with the command `touch wercker.yml`. Add the following contents to the file.

```bash
box: debian
deploy:
  steps:
    - install-packages:
        packages: openssh-client openssh-server

    - add-to-known_hosts:
        hostname: linode-IP-Address
        local: true

    - add-ssh-key:
        keyname: linode
 
    - script:
        name: Static site update on remote Linode
        code: |
          ssh username@linode-IP-Address git -C /home/username/sites/static-site-name/ pull
```

There are a few important items to note in the wercker.yml deploy pipeline.

* __hostname__: Enter the IP address of your Linode server.
* __keyname__: linode is the name of the public/private key pair created earlier with the Wercker App.
* __script__:__name__: Is the name of the script 
* __script__:__code__: username is your linode username that's used to login with SSH at the command line. Shortly, we will create a sites directory in your Linode home directory and then clone the static site GitHub repo.

{{< note >}}
Pushing the wercker.yml file to GitHub at this time will trigger the workflow in Wercker, which will result in an error.
{{ /note }}

There is one last thing to do. Log into your Linode server with the `ssh` command. In your home directory create a directory called sites. Change into the sites directory and clone the GitHub directory containing your static site.

```bash
ssh username@linode-IP-Address
mkdir sites
cd sites
git init
git clone https://github.com/github-username/static-site-name.git

```


#### Successful Wercker Build and Deployment

From your local development machine, push the wercker.yml file to your GitHub static site repository. Wercker will now trigger the workflow containing the deploy-production pipeline. If there are no issues, the workflow will execute successfully.

<p align="center">
  <img src="/images/wercker/deploy_production_success.jpg" alt="Deploy production success" /> 
</p>


#### Tips and Tricks with Wercker

1. If you are new to using Wercker, it is very possible that you may create a pipeline incorrectly. One such case is deleting the default build pipeline and adding a deploy-production pipeline whose wercker.yml file is configured incorrectly. This generates an error during the Wercker Run phase.

<p align="center">
  <img src="/images/wercker/deploy_error.jpg" alt="Workflow error" /> 
</p>

2. Using a home directory shortcut `~/sites/site-name/`, in the script code, can result in a "Permission denied" error in the Wercker console output.

```bash
deploy:
  steps:
    - install-packages:
        packages: openssh-client openssh-server

    - add-to-known_hosts:
        hostname: 12.34.56.789
        local: true

    - add-ssh-key:
        keyname: linode
 
    - script:
        name: Static site update on remote Linode
        code: |
          ssh username@linode-IP-Address git -C ~/sites/static-site-name/ pull
```

<p align="center">
  <img src="/images/wercker/script_path_error.jpg" alt="Home directory error" /> 
</p>


## The Production Enviroment: Case 1

Hugo's server component can be run as a web server in production. This requires that Nginx be configured to reverse proxy the Hugo server. Let's start by installing Nginx on the production server.

### Installing Nginx

Let's [install Nginx](http://nginx.org/en/linux_packages.html) and check that it is running.

    sudo apt-get update
    sudo apt-get install nginx
    systemctl status nginx

If the Nginx server is running as expected the following output should be displayed:

```bash
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2017-12-14 10:48:55 CST; 1 weeks 6 days ago
  Process: 2332 ExecStop=/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid (code=exited, status=0/SUCCESS)
  Process: 25525 ExecReload=/usr/sbin/nginx -g daemon on; master_process on; -s reload (code=exited, status=0/SUCCESS)
  Process: 2360 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
  Process: 2339 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
 Main PID: 2371 (nginx)
    Tasks: 2
   Memory: 2.7M
      CPU: 1.014s
   CGroup: /system.slice/nginx.service
           ├─2371 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
           └─2374 nginx: worker process  
```

There are a number of commands that will become necessary for starting, stopping, reloading, and checking the status of the Nginx server as changes are made to the Nginx server configuration files located at `/etc/nginx/sites-available`.

    // Check server status
    systemctl status nginx

    // Reload Nginx without stopping
    sudo systemctl reload nginx

    // Stopping Nginx server
    sudo systemctl stop nginx

    // Starting Nginx server
    sudo systemctl start nginx


### Configuring Nginx for Hugo Server

Hugo blog has a built-in server that runs, by default, on port 1313. This means that if Hugo was installed on your local machine, such as a desktop or laptop, it is accessible with `http://localhost:1313/`. 

Providing direct port access can increase vulnerabilities to web based attacks. Using Nginx protects your Linode server against common vulnerabilities and can be configured to make several server instances publicly accessible on port 80 by adding subdomains to your domain name. 

In our case we added `blog` subdomain `http://blog.example.com`.

<p align="center">
  <img src="/images/Nginxrevproxy.jpg" alt="Nginx as Reverse proxy" /> 
</p>

Nginx configuration files are located at `/etc/nginx/sites-available`. Use the following commands to list the existing configuration files.

```bash
cd ~/
cd /etc/nginx/sites-available
ls
```

For a new Nginx installation, there should all ready be a configuration file named `default`. Generally, the /sites/available directory will contain several Nginx configuration files. Directory /etc/nginx/sites-enabled will contain a symbolic link to each configuration file that will be active. See the default symbolic link by doing the following:

```bash
cd ~/
cd /etc/nginx/sites-enabled
ls
```

In this guide, two approaches to configuring Nginx will be shown. The first uses the existing default configuration file that is located in the /etc/nginx/sites-available directory. In the second approach, a new configuration file will be added to /etc/nginx/sites-available . A new symbolic link, to this configuration file, will then be added to the /etc/nginx/sites-enabled directory.

{{< note >}}
Both configuration techniques are shown here for reference purposes. Using both configuration techniques will result in Nginx run errors. Choose only one of the two presented.
{{< /note >}}

##### Modifying Default Configuration

Let's add a configuration, for the Hugo server, to the existing default configuration file. The arrangement below shows a default configuration file for two server applications. The first allows `http://example.com/` to access a server application running on port 8081. The second `http://blog.example.com/` will access a Hugo blog running on port 1313. 

```bash
# Default server configuration
#
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        #root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name www.example.com example.com;

        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_set_header Host $host;
                proxy_pass http://127.0.0.1:8081;
        }
}

server {
        listen 80;

        server_name blog.example.com;

        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_set_header Host $host;
                proxy_pass http://127.0.0.1:1313;
        }
}
```

For the purposes of this guide, begin by editing the Nginx server default configuration file with the [Vim](http://www.vim.org/) text editor:

    sudo vim /etc/nginx/sites-available/default

Add a second server block, as shown in the configuration above, to the default Nginx server configuration.

{{< note >}}
Replace `example.com` with your domain name.
{{< /note>}}

```bash
server {
        listen 80;

        #root /var/www/html;

        server_name blog.example.com;

        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_set_header Host $host;
                proxy_pass http://127.0.0.1:1313;
        }
}
```

To end editing of the config file press the **Escape** key, then enter `:wq`. Execute the command with the **Enter** key.

{{< note >}}
[Nano editor](https://www.nano-editor.org/) can also be used to edit the Nginx configuration file.
{{< /note >}}

###### Vim Editor Basics

The Vim editor has two modes of operation.

1. Command Mode

Press the **Escape** key to enter Command mode. Command mode allows several actions such as saving the file and exiting, deleting a character or a line of text, and exiting without saving changes to the file.

  * dd  - Delete a line of text
  * x   - Delete a single character
  * :wq - Save changes to the file and exit. Execute the command with the **Enter** key.
  * :q! - Exit the file without saving changes. Execute the command with the **Enter** key. 

2. Insert Mode

Press `i` to enter insert mode. Insert mode allows text to be added or deleted from the config file.


##### Creating a new Configuration File

The concept here is to create a new configuration file in /etc/nginx/sites-available directory. Then create a symbolic link to this configuration file in /etc/nginx/sites-enabled directory. Replace example.com with the domain name of your static site.

```bash
cd ~/
cd /etc/nginx/sites-available
sudo vim blog.example.com
```

This should create a blank file. Enable the VIM editor insert mode by selecting letter `i` on the keyboard and paste the configuration shown below into the file.

```bash
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        server_name blog.example.com;

        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_set_header Host $host;
                proxy_pass http://127.0.0.1:1313;
        }
}
```

Press the **Escape** key to exit editor Insert mode and enter the editor Command mode. Enter `:wq` and execute the command with the **Enter** key. This saves the file.


#### Adding a new Symbolic Link

The general command for creating a symbolic link is, `ln -s <SOURCE_FILE> <DESTINATION_FILE>`. To create a symbolic link to the newly created Nginx configuration file, named blog.example.com, do the following:

```bash
cd ~/
ln -s /etc/nginx/sites-available/blog.example.com /etc/nginx/sites-enabled/blog.example.com
```

A new symbolic link is now created in the sites-enabled directory. Verify as shown below.

```bash
cd ~/
cd /etc/nginx/sites-enabled
ls
```

Nginx configurations for reverse proxying the Hugo server, running on the Linode production server, is now complete. Before the static site can be accessed in a web browser at URL , `http://blog.example.com`, Hugo server needs to be installed on the Linode production servers. Let's install the latest version of Hugo.


### Installing Hugo on the Production Server

Log into your Linode production server with the SSH command. Use the `wget` command to get the latest Hugo release files. Latest releases are located at [Hugo releases](https://github.com/gohugoio/hugo/releases). An example command is shown:

    wget https://github.com/gohugoio/hugo/releases/download/v0.31.1/hugo_0.31.1_Linux-64bit.deb

{{< note >}}
To get the link to the release you wish to obtain right click on the release link and select Copy Link. Paste in the Terminal window.
{{< /note >}}

<p align="center">
  <img src="/images/CopyLink.jpg" alt="Copy release links" /> 
</p>

Install the package using the Terminal console with the following command:

    sudo dpk -i hugo *.deb

Installation of Hugo can be confirmed by checking the version of Hugo that was installed:

    hugo version

Another way to test the installation is with the Hugo help command:

    hugo help

Results should be similar to:

```bash
hugo is the main command, used to build your Hugo site.

Hugo is a Fast and Flexible Static Site Generator
built with love by spf13 and friends in Go.

Complete documentation is available at http://gohugo.io/.

Usage:
  hugo [flags]
  hugo [command]

Available Commands:
  benchmark   Benchmark Hugo by building a site a number of times.
  config      Print the site configuration
  convert     Convert your content to different formats
  env         Print Hugo version and environment info
  version     Print the version number of Hugo

Flags:
  -b, --baseURL string             hostname (and path) to the root, e.g. http://spf13.com/
  -D, --buildDrafts                include content marked as draft

```

### Running Hugo Server at the Command Line

Equally important to configuring Nginx correctly, is issuing the correct command to start the Hugo server. Consider the following command which runs blog articles in draft stage using the [Material Design](https://themes.gohugo.io/material-design/) theme.

    hugo server -t material-design -D

In the development environment the command above is fine. However, since the application is to be run in the production environment -D flag, to display drafts, should not be used. 

    hugo server -t material-design

Output from the Terminal console will be similar to:

```bash
Started building sites ...

Built site for language en:
1 of 1 draft rendered
0 future content
0 expired content
1 regular pages created
8 other pages created
0 non-page files copied
2 paginator pages created
0 tags created
0 categories created
total in 8 ms
Watching for changes in /home/username/hugo/{data,content,layouts,themes,static}
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

Note that the web server is available at `http://locahost:1313/`. If you attempt to enter your domain name `http://blog.example.com/` in a web browser, content from your blog will appear. However, it will appear without CSS formatting. 

<p align="center">
  <img src="/images/NoCSS.jpg" alt="Website without CSS" /> 
</p>

Viewing the web browser page source will show that links to page CSS files include `http://localhost:1313/`. 

<p align="center">
  <img src="/images/LocalhostCSS.jpg" alt="Page source with localhost CSS" /> 
</p>

Clearly, what we need is something similar to `http://example.com/css/style.css`. We can achieve the correct CSS links by adding the baseUrl and appendPort setting in the command to run the Hugo server.

    hugo server -t material-design -D --baseUrl="http://blog.example.com" --appendPort=false &

{{< note >}}
  1. If not already in the directory containing the Hugo files, change into that directory:

  2. Replace `example.com` in the command above with your domain name.
{{< /note >}}

    cd ~
    cd hugo
    hugo server -t your-template-name -D --baseUrl="http://blog.your-domain-name" --appendPort=false &

Running the Hugo server command outputs the listing below to the Terminal console.

```bash
Started building sites ...

Built site for language en:
1 of 1 draft rendered
0 future content
0 expired content
1 regular pages created
8 other pages created
0 non-page files copied
2 paginator pages created
0 tags created
0 categories created
total in 8 ms
Watching for changes in /home/username/hugo/{data,content,layouts,themes,static}
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://blog.example.com/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

Hugo is now available at `http://blog.example.com/` and not `http://localhost:1313` as shown previously. 

{{< note >}}
Notice the use of the & at the end of the command to run Hugo. This keeps the Hugo server running and allows us to logout and close the console Terminal. Start typing a few characters then press the `Return` key to display the command prompt without stopping the Hugo server.
{{< /note >}}

We should now be able to access our blog article in a web browser with `http://blog.example.com/`. Viewing the page source now shows CSS links such as `http://blog.example.com/css/style.css`.

<p align="center">
  <img src="/images/CSSLink.jpg" alt="Page source with correct CSS" /> 
</p>


### Tips and Tricks

Setting up Hugo can present unexpected challenges. A few of the most typical issues are addressed here.

#### Determine if Hugo is Running

From time to time it will be necessary to determine if Hugo server is running on port 1313. Use the command shown to obtain the status of the Hugo server on port 1313.

    sudo netstat -tulpn | grep 1313

Typical useage is:

    username@localhost:~$ sudo netstat -tulpn | grep 1313
    [sudo] password for username: 
    tcp        0      0 127.0.0.1:1313          0.0.0.0:*               LISTEN      4752/hugo  

Let's stop the Hugo application on port 1313.

    username@localhost:~$ pkill hugo

Then run the netstat command:

    username@localhost:~$ sudo netstat -tulpn | grep 1313

Result:

    username@localhost:~$


#### Config File not Found Error

Running the command to start the Hugo server from outside the Hugo directory will result in a "Config file not found error". To solve this issue, change into the Hugo directory before attempting to start the Hugo server.

    Error: Unable to locate Config file. Perhaps you need to create a new site.
       Run `hugo help new` for details. (Config File "config" Not Found in "[/home/username]")

#### Failed to Stop Nginx.Service

It is very possible that you may attempt to stop or reload the Nginx server and see the following console output:

    Failed to stop nginx.service: The name org.freedesktop.PolicyKit1 was not provided by any .service files
    See system logs and 'systemctl status nginx.service' for details.

To start, stop, or reload Nginx it is necessary to use the `sudo` prefix along with the command to stop, reload or start Nginx. That is, `sudo systemctl stop nginx`, `sudo systemctl reload nginx`, `sudo systemctl start nginx`.

#### Hugo Server did not Shutdown Using **CTRL+C**

There are times when shutting down the server with **CTRL+C** doesn't do the trick. Quite often, this is only apparent the next time we attempt to startup the Hugo server. For example, the Terminal console may display output to the effect, "Web Server is available at `http://localhost:12345`". 

Here is a scenario that can result in this condition:
1. **CTRL+C** was used to shutdown the Hugo server. 
2. The command prompt returned. This gives the impression that the server has stopped running.
3. At some point later, we issue the command to start the Hugo server. Now, "Web Server is available at `http://localhost:12345`" appears in the console output.

In this scenario the server did not shutdown as expected with **CTRL+C**. As a result an error statement was issued and a new server instance was created on port 12345. 
    
    ERROR 2017/12/30 12:18:30 port 1313 already in use, attempting to use an available port
    Started building sites ...

{{< note >}}
For the purposes of this guide, Hugo running on any port other than port 1313 is not desired because our Nginx server configuration is set to `proxy_pass http://127.0.0.1:1313;`.
{{< /note >}}

It is now necessary to shutdown the old server instance running on port 1313. Shutting down the process on port 1313 can be done with the following command:

    sudo kill $(sudo lsof -t -i:1313)

Terminal output should be similar to:

    [1]-  Terminated     hugo server -t material-design -D --baseUrl="http://blog.example.com" --appendPort=false

However, stopping the process on port 1313 is not enough since a new Hugo instance was fired up on port 12345 when port 1313 was not available. In this situation, running command `ps aux | grep hugo` should show that there is more than one Hugo servers running. Each has a different process id.

{{< note >}}
`username` will be replaced with the username you used to login to Linode.
{{< /note >}}

    username@localhost:~$ ps aux | grep hugo
    username 15831  0.0  2.2  39340 22744 ?        Sl   12:18   0:01 hugo server -t material-design -D --baseUrl=...
    username 15935  0.0  2.2  40396 22768 ?        Sl   12:42   0:01 hugo server -t material-design -D --baseUrl=...
    username 16546  0.0  0.0  14224   940 pts/1    S+   15:08   0:00 grep --color=auto hugo

{{< note >}}
--baseUrl=`http://blog.example.com` --appendPort=false was shorted above to --baseUrl=...
{{< /note >}}

A better approach is to use the `pkill` command.

    pkill hugo

Enter the `ps aux | grep hugo` command to show that no hugo servers are runnng as shown:

    username 16628  0.0  0.1  14224  1020 pts/1    S+   15:29   0:00 grep --color=auto hugo

#### Web Browser Returns Bad Gateway

"Bad Gateway" can be returned by the web browser if the Hugo server is not running, or is not running on port 1313. 

<p align="center">
  <img src="/images/BadGateway.jpg" alt="Bad Gateway web page" /> 
</p>

In this case, `ps aux | grep hugo` command can be used to verify that the server is not running on port 1313.


## Development Environment: Case 2

Case 2 demonstrates developing the static site locally then pushing the site files to GitHub. A second Wercker application will needed with a new wercker.yml configuration file. Since this new Wercker app will need to link to a new GitHub repo, it is necessary to develop a new Hugo static site and push it a new GitHub repo.

## Deployment Environment: Case 2

Wercker is then used to generate the files of the static site in the build pipeline. During the build phase, Wercker generates a folder named, public, which contains the files of the generated static site.

### Creating wercker.yml file

Create a file, wercker.yml, on your local machine in the directory containing your static website. One option is to create the file at the command line with the command `touch wercker.yml`. Add the following contents to the file.

```bash
box: debian
build:
  steps:
    - arjen/hugo-build@1.25.2: 
        version: "0.32.3"
        theme: material-design
        flags: --buildDrafts=true
        disable_pygments: true
        clean_before: true
        prod_branches: master

deploy:
  steps:
 
```

Let's discuss the build pipeline in the wercker.yml file. `- arjen/hugo-build@1.25.2:` refers to a set of Hugo build instructions created by [ArjenSchwarz](https://github.com/ArjenSchwarz/wercker-step-hugo-build). It is available in the Wercker Registry. Select a new tab in your web browser and enter the URL ` https://app.wercker.com/explore`. Search for `hugo` on the Registry page and select arjen / hugo-build. A description of additional Hugo build configurations are provided in this Registry item.

<p align="center">
  <img src="/images/wercker/arjen.jpg" alt="Arjen App" /> 
</p>


### Tips and Tricks with Wercker

1. It may be tempting to look at a Wercker console error output and guess that the username in the wercker.yml script should be "wercker" as in `export WERCKER_STEP_OWNER="wercker"`. An incorrect username, in the script code, can result in a "public key error" output in the Wercker console output. 

```bash
deploy:
  steps:
    - install-packages:
        packages: openssh-client openssh-server

    - add-to-known_hosts:
        hostname: 12.34.56.789
        local: true

    - add-ssh-key:
        keyname: linode
 
    - script:
        name: Static site update on remote Linode
        code: |
          ssh wercker@linode-IP-Address git -C /home/username/sites/static-site-name/ pull
```

{{< note >}}
The username should be the username that was used to SSH into the Linode server and add the linode_PUBLIC key to the .ssh/authorized file. This step was performed in the "Add Public Key to Linode User Account" section.
{{< /note >}}

<p align="center">
  <img src="/images/wercker/publickey_error.jpg" alt="Public key error" /> 
</p>



## The Production Environment: Case 2

Now that the files of the static site have been created by the Wercker build pipeline and deployed to the Linode production server with the Wercker deploy-production pipeline, the next issue is making the site accessible to the public using your domain name. This example, uses subdomain `http://coder.example.com` . Follow the instructions in the section titled, "Adding Blog Subdomain", to create a new subdomain, named `coder`.


### Serving Static Site with Nginx

Here two approaches to presenting the contents of the static site so that it is accessible with a web browser is presented. The first and most efficient approach is serving the files generated by the Hugo static site generator with Nginx. 

## Learning more about Hugo

Now that your basic blog site is accessible to the world using your domain name, you will want to further increase the depth of your knowledge about the mechanics of creating and updating your Hugo blog. [Hugo docs](https://gohugo.io/documentation/) is a great place to start on this quest. [Giraffe Academy](https://www.youtube.com/watch?v=qtIqKaDlqXo) also has a series of great Hugo video tutorials.
