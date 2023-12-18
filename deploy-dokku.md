# Dokku LimeSurvey

## Installing Docker & Dokku

```shell
# Installing Docker
apt remove docker docker-engine docker.io containerd runc
apt update
apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - apt-key fingerprint 0EBFCD88
apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
apt update
apt install -y docker-ce docker-ce-cli containerd.io

# To test Docker installation, run:
docker run --rm hello-world

# Installing Dokku
wget -nv -O - https://packagecloud.io/dokku/dokku/gpgkey | apt-key add -
export SOURCE="https://packagecloud.io/dokku/dokku/ubuntu/"
export OS_ID="$(lsb_release -cs 2>/dev/null || echo "bionic")"
echo "xenial bionic focal" | grep -q "$OS_ID" || OS_ID="bionic"
echo "deb $SOURCE $OS_ID main" | tee /etc/apt/sources.list.d/dokku.list
apt update
apt install -y dokku
dokku plugin:install-dependencies --core

# Installing Dokku plugins
sudo dokku plugin:install https://github.com/dokku/dokku-mariadb.git mariadb
sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
```

## Configuring Dokku

You must add your SSH public key to Dokku so you can deploy applications to it
using git. Put your public key inside a file and run:

```shell
dokku ssh-keys:add admin path/to/pub_key
```

## Creating the Application

On Dokku server, set the correct values for the variables:

```shell
export ADMIN_EMAIL="admin@example.com"
export ADMIN_NAME="Admin"
export APP_NAME="limesurvey"3c899a41f09e
export DOMAIN="limesurvey.example.com"
export MARIADB_NAME="mariadb_limesurvey"
```

Now, run the commands to create and configure the app:

```shell
# Create app
dokku apps:create $APP_NAME
dokku checks:disable $APP_NAME
dokku config:set --no-restart $APP_NAME DOKKU_LETSENCRYPT_EMAIL=$ADMIN_EMAIL
dokku domains:add $APP_NAME $DOMAIN

## Set Proxy map
dokku proxy:ports-set limesurvey http:80:80

# Create database services
dokku mariadb:create $MARIADB_NAME

# Link database server to app
dokku mariadb:link $MARIADB_NAME $APP_NAME

# Add initial data/config:
sudo mkdir -p /var/lib/dokku/data/storage/$APP_NAME/{config,upload}
sudo chown -R www-data:www-data /var/lib/dokku/data/storage/$APP_NAME/{config,upload}
dokku storage:mount $APP_NAME /var/lib/dokku/data/storage/$APP_NAME/config:/var/www/html/application/config
dokku storage:mount $APP_NAME /var/lib/dokku/data/storage/$APP_NAME/upload:/var/www/html/upload
```
## TODO - Populate database config with environment variables

Replace code below with some getenv() data.
```php
'db' => array(
  'connectionString' => 'mysql:host=dokku-mariadb-mariadb-limesurvey;port=3306;dbname=mariadb_limesurvey;',
  'emulatePrepare' => true,
  'username' => 'mariadb',
  'password' => 'b972ced9c6bb704e',
  'charset' => 'utf8mb4',
  'tablePrefix' => 'lime_',
),
```

## First Deployment

Since you've created and configured the app on the server, you can switch to
your local machine and execute the first app deploy! After cloning the
repository (and inside its directory), execute:

```shell
git remote add dokku dokku@<server-ip>:<app-name>
git push dokku master
```

## Finishing Installation

To finish the installtion you need to configure the SSL certificate, mount the
data path and import

Finally, on server again:

```shell
# Create/configure SSL certificate:
dokku letsencrypt:enable $APP_NAME
```

## Importing data

TODO
