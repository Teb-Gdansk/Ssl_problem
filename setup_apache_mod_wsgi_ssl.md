
# Tutorial: Setting Up Apache with mod_wsgi and SSL from Cloudflare

This step-by-step tutorial will guide you through the process of setting up Apache with mod_wsgi to host your Python app, and enabling SSL (HTTPS) via Cloudflare.

## Step 1: Install Apache and mod_wsgi

1. **Update your package list**:
   ```bash
   sudo apt update
   ```

2. **Install Apache**:
   ```bash
   sudo apt install apache2
   ```

3. **Install mod_wsgi** (to run Python apps):
   ```bash
   sudo apt install libapache2-mod-wsgi-py3
   ```

4. **Enable the mod_wsgi module**:
   ```bash
   sudo a2enmod wsgi
   ```

5. **Restart Apache**:
   ```bash
   sudo systemctl restart apache2
   ```

## Step 2: Configure Apache for Your Python App

1. **Create a WSGI file** (e.g., `myapp.wsgi`):
   - This file tells Apache how to run your Python app.

   Create the file at `/var/www/myapp/myapp.wsgi`:
   ```python
   import sys
   sys.path.insert(0, '/var/www/myapp')

   from main import app as application  # Assuming your app is in main.py
   ```

2. **Configure Apache VirtualHost**:
   - Create a new Apache configuration file for your site (e.g., `/etc/apache2/sites-available/myapp.conf`):
   ```apache
   <VirtualHost *:80>
       ServerName yourdomain.com
       ServerAlias www.yourdomain.com

       WSGIDaemonProcess myapp threads=5
       WSGIScriptAlias / /var/www/myapp/myapp.wsgi

       <Directory /var/www/myapp>
           Require all granted
       </Directory>

       Alias /static /var/www/myapp/static
       <Directory /var/www/myapp/static/>
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/myapp_error.log
       CustomLog ${APACHE_LOG_DIR}/myapp_access.log combined
   </VirtualHost>
   ```

3. **Enable the VirtualHost configuration**:
   ```bash
   sudo a2ensite myapp.conf
   sudo systemctl reload apache2
   ```

4. **Place Your Python App**:
   - Ensure your app files (including `main.py`) are in `/var/www/myapp`.

## Step 3: Set Up SSL with Cloudflare

1. **Go to Cloudflare Dashboard**:
   - Log into your Cloudflare account and select your domain.

2. **Set SSL/TLS to "Full"**:
   - Go to the **SSL/TLS** tab.
   - Choose **Full** (not Full (strict)) to allow Cloudflare to connect to your server even if it doesn't have a valid SSL certificate yet.

3. **Install a Cloudflare Origin Certificate** (optional but recommended):
   - In the **SSL/TLS** section, go to **Origin Certificates**.
   - Generate a free Cloudflare Origin Certificate.
   - Download the certificate file (`.pem`) and the private key (`.key`).

4. **Configure Apache for SSL**:
   - Modify your VirtualHost to handle HTTPS by creating or editing the file `/etc/apache2/sites-available/myapp-ssl.conf`:
   ```apache
   <VirtualHost *:443>
       ServerName yourdomain.com
       ServerAlias www.yourdomain.com

       WSGIDaemonProcess myapp threads=5
       WSGIScriptAlias / /var/www/myapp/myapp.wsgi

       <Directory /var/www/myapp>
           Require all granted
       </Directory>

       Alias /static /var/www/myapp/static
       <Directory /var/www/myapp/static/>
           Require all granted
       </Directory>

       SSLEngine on
       SSLCertificateFile /etc/ssl/certs/cloudflare_origin_cert.pem
       SSLCertificateKeyFile /etc/ssl/private/cloudflare_origin_key.pem

       ErrorLog ${APACHE_LOG_DIR}/myapp_ssl_error.log
       CustomLog ${APACHE_LOG_DIR}/myapp_ssl_access.log combined
   </VirtualHost>
   ```

   - Place the downloaded Cloudflare Origin Certificate and private key in the appropriate directories (e.g., `/etc/ssl/certs/cloudflare_origin_cert.pem` and `/etc/ssl/private/cloudflare_origin_key.pem`).

5. **Enable SSL Modules and Site**:
   ```bash
   sudo a2enmod ssl
   sudo a2ensite myapp-ssl.conf
   sudo systemctl reload apache2
   ```

## Step 4: Redirect HTTP to HTTPS (Optional)

If you want to force all traffic to use HTTPS, you can configure a redirect from HTTP to HTTPS in your VirtualHost configuration:
```apache
<VirtualHost *:80>
    ServerName yourdomain.com
    ServerAlias www.yourdomain.com

    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R=301,L]
</VirtualHost>
```

Enable the `rewrite` module:
```bash
sudo a2enmod rewrite
sudo systemctl reload apache2
```

## Step 5: Test Your Configuration

1. **Check Apache Configuration**:
   - Run the following command to ensure there are no syntax errors:
   ```bash
   sudo apachectl configtest
   ```

2. **Restart Apache**:
   ```bash
   sudo systemctl restart apache2
   ```

3. **Verify SSL**:
   - Open your browser and visit `https://yourdomain.com`.
   - The connection should be secured with SSL, and you should see the padlock symbol indicating a valid HTTPS connection.

---

### Final Notes / Ostateczne uwagi

- Make sure to replace `yourdomain.com` with your actual domain name throughout the tutorial.
- If you have any issues, check the Apache error logs for more information.
- Remember that SSL might take a few minutes to propagate, so if it doesn't work immediately, give it some time.
