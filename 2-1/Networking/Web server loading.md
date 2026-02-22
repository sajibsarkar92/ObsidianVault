name server as web server

apache ? httpd daemon

## apache server load

```bash
ss -tlpn
```

# install

```shell
sudo apt install apache2
```

# check

in browser type localhost and enter

![[Pasted image 20260222124213.png]]


# documentation file 

```bash
cd var/www/html/



```


# move file or rename?


```bash

sudo mv index.html index.html.orig

```


# create new file

```bash
sudo nano index.html

```

in this html, what you write will be displayed in the localhost and it will be viewed upon refresh


# default page

didnt catch it


# virtual site


ubuntu server documenttion

how to guides

web server


[guide](https://ubuntu.com/server/docs/how-to/web-services/install-apache2/#install-apache2)

go to part 2 of configuration settings


```
cd /etc/apache2
```


basic directives section

```
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/mynewsite.conf
```

```
cd /etc/apache2/
```

# open the copied file


```
sudo nano mynewsite.conf
```

