---
layout: post
title: Minimal webserver setup for macOS Mojave
---
Minimal web server setup for macOS Mojave
I reinstalled my Mac on the first day of 2019. After years of development, my Mac was slow as hell because I installed too many tools and packages. I'm trying to avoid it with this reinstall, first is the web server.

There are many great choices available on the internet, including Valet, MAMP, XAMPP... I used Valet and like it in general. But I notice macOS Mojave comes with Apache and PHP 7.1 built in. Woohoo! A little config and installing MySQL are what I need to do to install my web server.

## Configure Apache
Edit Apache config file as root
```
sudo vi /etc/apache2/httpd.conf
```
Uncomment the following lines:
```
LoadModule php7_module libexec/apache2/libphp7.so
```
Find `<Directory />` and change the content insides that block:
```
<Directory />
 Options Indexes FollowSymLinks Includes ExecCGI
 AllowOverride All
 Order deny, allow
 Allow from all
</Directory>
```
Find `/Library/WebServer/Documents` and replace both line with your document root directory:
```
DocumentRoot "/Users/yourusername/workspace"
<Directory "/Users/yourusername/workspace">
```
Next, find `<IfModule unixd_module>`, comment out the default and replace it with our info:
```
<IfModule unixd_module>
#User _www
#Group _www
User yourusername
Group staff
</IfModule>
```
Save your file and start Apache:
```
sudo apachectl start
```
Now we have a running web server with no extra package. Woohoo!

## Install MySQL
Go to https://dev.mysql.com/downloads/mysql/ and download the `5.7` version then install it.
In general, the installation process will be: Next > Next > Next .. > Finish.
You will be provided a temporary root password. Remember it.
After installing the pkg, edit your `$PATH`:
```
# for .bashrc
export PATH=$PATH:/usr/local/mysql/bin

# for .zshrc
PATH="$PATH:/usr/local/mysql/bin"
```
Now open a new terminal window, log in to `mysql` then fill your password at previous step:
```
mysql -u root -p
```
Change your root password:
```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
```
Done!
