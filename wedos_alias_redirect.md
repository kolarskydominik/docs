# WEDOS - add Aliases and Redirect them to main url, WordPress

sources:
- https://kb.wedos.com/cs/webhosting/nastaveni-domen-pro-lets-encrypt-certifikat/
- https://help.wedos.cz/navody/webhosting/wordpress/wordpress-zmena-nazvu-webu/
- https://kb.wedos.com/cs/webhosting/webhosting-presmerovani-domeny-pres-https/
- https://help.wedos.cz/navody/webhosting/wordpress/nastaveni-https-ve-wordpressu-wp/

## 1. Wordpress Admin

Go: `Setting/General`

Change `WordPress Address (URL)` and `Site Address (URL)` to new name.

## 2. WEDOS Admin

1. `https://client.wedos.com/webhosting/`
2. `Tools > Rename Webhosting`
3. [&check;] change DNS records
4. Rename
5. Add aliases if needed

## 3. FTP changes, Aliases

Go: `www/domains`

1. Rename directory to new name.
2. Create aliases directories and file .htaccess in them with
```
RewriteEngine On
RewriteRule (.*) https://novadomena.tld/ [R=301,L]
```
or 
```
RewriteEngine On
RewriteCond %{HTTP_HOST} ^(www\.)?staradomena\.tld$
RewriteRule (.*) https://novadomena.tld/$1 [R=301,L]
```



