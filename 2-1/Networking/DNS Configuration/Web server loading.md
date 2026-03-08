tags:

- webserver
    
- apache
    
- ubuntu
    
- linux
    
    aliases:
    
- Apache Setup Guide
    

# Apache Web Server (httpd)

Apache (often referred to as the `httpd` daemon in some Linux distributions) is a popular open-source web server.

## 1. Checking Server Load / Ports

To check listening ports and see if a web server is currently running, use:

```
ss -tlpn
```

> **🧠 Why are we doing this?**
> 
> Before installing a web server, it's good to check if port 80 (the default port for web traffic) is already being used by another program. If another web server (like Nginx) is already running, Apache will fail to start because two programs can't listen on the same exact port at the same time.

## 2. Installation

To install Apache on Ubuntu:

```
sudo apt install apache2
```

> **🧠 Why are we doing this?**
> 
> This command downloads and installs the Apache web server software. Once installed, it runs as a background service (daemon) waiting for people to ask for web pages.

### Verification

1. Open your web browser.
    
2. Type `localhost` (or the server's IP address) in the address bar and press **Enter**.
    
3. You should see the Apache2 Ubuntu Default Page.
    

![[Pasted image 20260222124213.png]]

> [!info] The Default Page
> 
> The page you see initially is simply a placeholder `index.html` file provided by Ubuntu to confirm Apache is working correctly. It also contains basic configuration and directory structure information.

## 3. Managing Web Content

By default, Apache serves files from the `/var/www/html/` directory.

> **🧠 Why `/var/www/html/`?**
> 
> A web server shouldn't let people on the internet browse your entire computer. It needs a specific, locked-down folder that is deemed "safe for the public." By default, Apache uses `/var/www/html/` as its public "storefront."

### Navigating to the Web Root

```
cd /var/www/html/
```

### Backing up the Default Page

Instead of deleting the default page, it's good practice to rename (move) it so you have a backup:

```
sudo mv index.html index.html.orig
```

> **🧠 Why are we doing this?**
> 
> The default page actually has a lot of useful documentation about how Apache is set up on Ubuntu. By renaming it to `.orig` (original), we hide it from the web server (which only looks for exactly `index.html`) but keep it on our drive just in case we want to read it later.

### Creating Your Own Page

Create a new `index.html` file:

```
sudo nano index.html
```

_Note: Whatever HTML you write and save in this file will now be displayed in your browser at `localhost` upon refreshing the page._

## 4. Setting up a Virtual Site (VirtualHosts)

Virtual Hosts allow you to run multiple websites on a single Apache server.

> **🧠 Why use Virtual Hosts?**
> 
> Imagine you own `myblog.com` and `mystore.com`, but you only have one physical server. Virtual Hosts let Apache look at the incoming request, say "Ah, you're asking for the blog," and serve files from a completely different folder than if the user asked for the store.

**Reference Documentation:** [Ubuntu Server Docs: Install Apache2](https://ubuntu.com/server/docs/how-to/web-services/install-apache2/#install-apache2 "null")

### Navigate to Apache Configuration

Apache configuration files are stored in `/etc/apache2/`.

```
cd /etc/apache2
```

### Create a New Site Configuration

Copy the default site configuration file (`000-default.conf`) to create a template for your new site:

```
sudo cp sites-available/000-default.conf sites-available/mynewsite.conf
```

> **🧠 Why are we doing this?**
> 
> Writing a configuration file from scratch is tedious and error-prone. The default configuration file already has the correct syntax and structure. Copying it gives us a perfectly working template that we just need to tweak.

### Edit the New Configuration

Open the newly copied configuration file to adjust your site's settings:

```
sudo nano sites-available/mynewsite.conf
```

Inside this file, you need to update a few key directives. It should look something like this:

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName mynewsite.local
    DocumentRoot /var/www/mynewsite

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

> **🧠 What do these settings mean?**
> 
> - `VirtualHost *:80`: Listen on port 80 (standard HTTP) for all incoming connections.
>     
> - `ServerName`: The actual domain name someone types in to reach this specific site.
>     
> - `DocumentRoot`: The folder where the files for _this specific site_ are stored.
>     

## 5. Create the Directory Structure

Now that we told Apache to look for our site at `/var/www/mynewsite` (in the `DocumentRoot` above), we need to actually create that folder.

```
sudo mkdir -p /var/www/mynewsite
```

Next, assign ownership of the directory so you can edit files inside it without needing `sudo` every time:

```
sudo chown -R $USER:$USER /var/www/mynewsite
```

> **🧠 Why change ownership?**
> 
> Directories created with `sudo` are owned by the root administrator. If you don't change the ownership to your regular user (`$USER`), you'll be blocked every time you try to save a new web page or upload images via FTP/SFTP.

Create a simple test page for your new site:

```
nano /var/www/mynewsite/index.html
```

_(Add some basic HTML like `<h1>Welcome to My New Site!</h1>` and save)._

## 6. Enable the New Site

Apache comes with built-in tools to enable and disable virtual hosts.

Enable your new site using the `a2ensite` (Apache 2 Enable Site) command:

```
sudo a2ensite mynewsite.conf
```

> **🧠 Why are we doing this?**
> 
> Apache stores configurations in `sites-available`, but it doesn't automatically use them all (you might have test sites or inactive sites in there). The `a2ensite` command tells Apache, "I'm ready for this site to go live." (It technically creates a shortcut to your file in a folder called `sites-enabled`).

_(Optional)_ Disable the default site if you only want your new site to load:

```
sudo a2dissite 000-default.conf
```

## 7. Test and Restart Apache

Before restarting the web server, it is best practice to test your configuration files for any syntax errors:

```
sudo apache2ctl configtest
```

_(You should see `Syntax OK`. If not, review your `.conf` file for typos)._

> **🧠 Why are we doing this?**
> 
> If you made a typo (like forgetting a `>` bracket) in your config file, restarting Apache will cause the whole web server to crash and refuse to start back up, taking all your websites offline. Testing prevents this.

Finally, reload Apache to apply the changes:

```
sudo systemctl reload apache2
```

> **🧠 Restart vs Reload?**
> 
> `reload` is better than `restart`. A full restart kills the server and boots it back up, meaning anyone currently downloading a file from your site gets disconnected. `reload` gracefully finishes current connections while applying the new settings in the background.

## 8. Local Testing (Hosts File)

Because `mynewsite.local` isn't a real registered domain name on the internet (like https://www.google.com/search?q=google.com), your computer doesn't know how to find it. You need to map it to your local machine.

Edit your local `hosts` file:

```
sudo nano /etc/hosts
```

Add the following line right under `127.0.0.1 localhost`:

```
127.0.0.1   mynewsite.local
```

> **🧠 Why edit the hosts file?**
> 
> The `hosts` file is like your computer's personal address book. Before your computer asks the global internet where a domain is, it checks this file first. By adding `127.0.0.1 mynewsite.local`, you are tricking your computer into pointing that fake domain directly back to your own machine so you can test your new site locally.

Now, if you open your browser and go to `http://mynewsite.local`, you should see your custom test page!