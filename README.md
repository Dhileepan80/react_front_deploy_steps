
# Deploying a React App (CRA and Vite) on Apache Web Server

Follow these steps to deploy a React app built using either Create React App (CRA) or Vite on an Apache web server.

## Step 1: Build Your React App

Run the following command to build your React app:

```bash
npm run build
```

## Step 2: Install Apache Web Hosting

Ensure that Apache web hosting is installed on your system. If not, install it by running the following command:

```bash
sudo apt install apache2
```

## Step 3: Modify Apache Configuration

You need to modify the default Apache configuration file. The file path is:

```bash
/etc/apache2/sites-available/000-default.conf
```

Inside the `<VirtualHost *:80>` block, add the following code to allow configuration override using `.htaccess` files:

```bash
<Directory /var/www/html>
    AllowOverride All
</Directory>
```

### Enabling HTTPS with SSL Certificate

If you want to enable HTTPS and have a domain name with an SSL certificate, your Apache configuration file should look like this:

```bash
# HTTP VirtualHost (Port 80)
<VirtualHost *:80>
    ServerName yourdomain.local
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    # Optional: Redirect all HTTP traffic to HTTPS
    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R=301,L]
</VirtualHost>

# HTTPS VirtualHost (Port 443)
<VirtualHost *:443>
    ServerName yourdomain.local
    DocumentRoot /var/www/html

    SSLEngine on
    SSLCertificateFile /path/to/yourdomain.local.pem
    SSLCertificateKeyFile /path/to/yourdomain.local-key.pem

    <Directory /var/www/html>
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

## Step 4: Move Build Folder to Apache Directory

Once the build is complete, navigate to the root directory of your React app. You'll find a `build` folder. Move this folder to the Apache web server directory:

```bash
sudo mv build/ /var/www
```

Now, rename the `build` folder to `html`:

```bash
sudo mv build/ html
```

## Step 5: Add `.htaccess` for Client-Side Routing

To support client-side routing (ensuring that the app serves `index.html` for any route), create a `.htaccess` file in the `/var/www/html` directory:

```bash
nano /var/www/html/.htaccess
```

Add the following code to the `.htaccess` file:

```bash
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /

  # Redirect all requests to index.html except files or directories
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule ^ index.html [QSA,L]
</IfModule>
```

## Step 6: Restart Apache Service

Finally, restart the Apache service to apply the changes:

```bash
sudo systemctl restart apache2.service
```

Your React app should now be successfully deployed on Apache!
