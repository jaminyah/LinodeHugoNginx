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
| YOUR-TEMPLATE-NAME       | Replace with the Hugo theme that you downloaded       |
| YOUR-DOMAIN-NAME         | Replace with the domain name for your website         |
| username                 | Replace with Linode login user name                   |


Begin by completing the following guides if you are getting started with Linode for the first time.

1.  Familiarize yourself with our [Getting Started](/docs/getting-started) guide and complete the steps for setting your Linode's hostname and timezone. The guide explains how to sign-up, deploy a Ubuntu 16.04 image and connect to your Linode server using the secure shell protocol (SSH). For the purpose of installing Hugo and configuring Nginx, it is not necessary to do the "Setting the Hostname" steps.

2.  This guide will use `sudo` wherever possible. Complete the sections of our [Securing Your Server](/docs/security/securing-your-server) to create a standard user account, harden SSH access and remove unnecessary network services.


### Ubuntu Installation Update

Begin by updating Ubuntu if you have an existing Ubuntu installation and have previously completed the two steps above.

    sudo apt-get update


## Pointing your Domain to Linode

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


## Adding Blog Subdomain

One to the things you may want to do is add a subdomain to your domain and have this URL point to your blog articles. For example if your domain is `http://example.com` adding a blog subdomain creates `http://blog.example.com`. Log into your Linode server account and select the DNS Manager tab.

<p align="center">
  <img src="/images/AAAA.jpg" alt="AAAA records" /> 
</p>

In the section for A/AAAA Records select the link "Add a new A record".  For Hostname add `http://blog.example.com`. In the IP Address field enter the IP address of your Linode server. 

<p align="center">
  <img src="/images/EditAAAA.jpg" alt="Editing AAAA records" /> 
</p>

{{< note >}}
Your IP address can be found by selecting the Linodes tab. Replace `example.com` with your registered domain name. Save the Changes to the A/AAAA Record. Subdomain `blog` should now appear in the list of A/AAAA Records.
{{< /note >}}


## Logging into your Linode Server

Open a Terminal window and log in to your Linode server at the command line with SSH. A typical log in command should look as follows:

    ssh username@IPAddress

{{< note >}}
Enter the `username` that you use to log into your Linode server. This user must have `sudo` privileges.
Enter the `IPAddress` of your Linode server.
{{< /note>}}

Further clarification on logging in with SSH can be found in the [Getting started](/docs/getting-started) guide in the section titled "*Log In for the First Time*". 


## Installing Hugo

Create a directory named `hugo` and change into this directory.

    cd ~
    mkdir hugo
    cd hugo

Use the `wget` command to get the latest Hugo release files. Latest releases are located at [Hugo releases](https://github.com/gohugoio/hugo/releases).

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
  gen         A collection of several useful generators.
  help        Help about any command
  import      Import your site from others.
  list        Listing out various types of content
  new         Create new content for your site
  server      A high performance webserver
  undraft     Undraft resets the content's draft status
  version     Print the version number of Hugo

Flags:
  -b, --baseURL string             hostname (and path) to the root, e.g. http://spf13.com/
  -D, --buildDrafts                include content marked as draft
  -E, --buildExpired               include expired content
  -F, --buildFuture                include content with publishdate in the future
      --cacheDir string            filesystem path to cache directory. Defaults: $TMPDIR/hugo_cache/
      --canonifyURLs               if true, all relative URLs will be canonicalized using baseURL
      --cleanDestinationDir        remove files from destination not found in static directories
      --config string              config file (default is path/config.yaml|json|toml)
  -c, --contentDir string          filesystem path to content directory
      --debug                      debug output
```


## Creating a Blog Article

If you are getting started with creating blog articles with Hugo the best source of accurate and updated instructions is the Hugo site [Getting Started Quick Start Guide](https://gohugo.io/getting-started/quick-start/). Instructions begin at Step 2: *Create a New Site*.


## Building and Updating a Static Site

In general, developers focus on writting code, unit testing and incorporating code review changes. For the most part, many developers shy away from the inner details of deploying applications to the production environment. This task is usually handled by the DevOps team or backend crew. In this section, we will discuss building a static website and moving it from the development environment to the production server. Then, we will address issues related to keeping the site updated.

<p align="center">
  <img src="/images/ProductionCycle.jpg" alt="Production Cycle" /> 
</p>



## Installing Nginx

Now that you have a basic Hugo blog post created, the next issue is making it accessible to the public using your domain name. Hugo blog has a built-in server that runs, by default, on port 1313. This means that if Hugo was installed on your local machine, such as a desktop or laptop, it is accessible with `http://localhost:1313/`. 

Providing direct port access can increase vulnerabilities to web based attacks. Using Nginx protects your Linode server against common vulnerabilities and can be configured to make several server instances publicly accessible on port 80 by adding subdomains to your domain name. 

In our case we added `blog` subdomain `http://blog.example.com`.

<p align="center">
  <img src="/images/Nginxrevproxy.jpg" alt="Nginx as Reverse proxy" /> 
</p>

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


## Configuring Nginx

There are a number of commands that will become necessary for starting, stopping, reloading, and checking the status of the nginx server as changes are made to the nginx server configuration file `/etc/nginx/sites-available/default`.

    // Check server status
    systemctl status nginx

    // Reload Nginx without stopping
    sudo systemctl reload nginx

    // Stopping Nginx server
    sudo systemctl stop nginx

    // Starting Nginx server
    sudo systemctl start nginx

Making your Hugo blog publicly available with Nginx can be challenging. Most important is getting the Nginx configuration correct. The arrangement below shows a configuration for two server applications. The first allows `http://example.com/` to access a server application running on port 8081. The second `http://blog.example.com/` will access a Hugo blog running on port 1313. 

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

For the purposes of this guide, begin by editing your Nginx server configuration with the [Vim](http://www.vim.org/) text editor:

    sudo vim /etc/nginx/sites-available/default

Add the second server block, as shown in the configuration above, to your existing Nginx server configuration.

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

### Vim Editor Basics

The Vim editor has two modes of operation.

1. Command Mode

Press the **Escape** key to enter command mode. Command mode allows several actions such as saving the file and exiting, deleting a character or a line of text, and exiting without saving changes to the file.

  * dd  - Delete a line of text
  * x   - Delete a single character
  * :wq - Save changes to the file and exit. Execute the command with the **Enter** key.
  * :q! - Exit the file without saving changes. Execute the command with the **Enter** key. 

2. Insert Mode

Press `i` to enter insert mode. Insert mode allows text to be added or deleted from the config file.


## Running Hugo Server at the Command Line

An equally important issue to configuring Nginx correctly, is issuing the correct command to start the Hugo server. Consider the following command which runs blog articles in draft stage using the [Material Design](https://themes.gohugo.io/material-design/) theme.

    hugo server -t material-design -D

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
    hugo server -t YOUR-TEMPLATE-NAME -D --baseUrl="http://blog.YOUR-DOMAIN-NAME" --appendPort=false &

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


## Tips and Tricks

Setting up Hugo can present unexpected challenges. A few of the most typical issues are addressed here.

### Config File not Found Error

Running the command to start the Hugo server from outside the Hugo directory will result in a "Config file not found error". To solve this issue, change into the Hugo directory before attempting to start the Hugo server.

    Error: Unable to locate Config file. Perhaps you need to create a new site.
       Run `hugo help new` for details. (Config File "config" Not Found in "[/home/username]")

### Failed to Stop Nginx.Service

It is very possible that you may attempt to stop or reload the Nginx server and see the following console output:

    Failed to stop nginx.service: The name org.freedesktop.PolicyKit1 was not provided by any .service files
    See system logs and 'systemctl status nginx.service' for details.

To start, stop, or reload Nginx it is necessary to use the `sudo` prefix along with the command to stop, reload or start Nginx. That is, `sudo systemctl stop nginx`, `sudo systemctl reload nginx`, `sudo systemctl start nginx`.

### Hugo Server did not Shutdown Using **CTRL+C**

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

### Web Browser Returns Bad Gateway

"Bad Gateway" can be returned by the web browser if the Hugo server is not running, or is not running on port 1313. 

<p align="center">
  <img src="/images/BadGateway.jpg" alt="Bad Gateway web page" /> 
</p>

In this case, `ps aux | grep hugo` command can be used to verify that the server is not running on port 1313.


## Learning more about Hugo

Now that your basic blog site is accessible to the world using your domain name, you will want to further increase the depth of your knowledge about the mechanics of creating and updating your Hugo blog. [Hugo docs](https://gohugo.io/documentation/) is a great place to start on this quest. [Giraffe Academy](https://www.youtube.com/watch?v=qtIqKaDlqXo) also has a series of great Hugo video tutorials.
