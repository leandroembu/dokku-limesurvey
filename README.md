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
export APP_NAME="limesurvey"
export DOMAIN="limesurvey.example.com"
export MARIADB_NAME="mariadb_limesurvey"
```

Now, run the commands to create and configure the app:

```shell
# Create app
dokku apps:create $APP_NAME
dokku checks:disable $APP_NAME
dokku config:set --no-restart $APP_NAME DOKKU_LETSENCRYPT_EMAIL=$ADMIN_EMAIL
dokku domains:set $APP_NAME $DOMAIN

## Set Proxy map
dokku ports:set $APP_NAME http:80:8080

# Create database services
dokku mariadb:create $MARIADB_NAME

# Link database server to app
dokku mariadb:link $MARIADB_NAME $APP_NAME

# Add initial data/config:
sudo mkdir -p /var/lib/dokku/data/storage/$APP_NAME/{config,upload}
sudo chown -R www-data:www-data /var/lib/dokku/data/storage/$APP_NAME/{config,upload}
dokku storage:mount $APP_NAME /var/lib/dokku/data/storage/$APP_NAME/config:/var/www/html/application/config
dokku storage:mount $APP_NAME /var/lib/dokku/data/storage/$APP_NAME/upload:/var/www/html/upload

# Environment variables
dokku config:set $APP_NAME DB_TYPE=mysql
dokku config:set $APP_NAME DB_HOST=<place-host-here>
dokku config:set $APP_NAME DB_PORT=3306
dokku config:set $APP_NAME DB_NAME=$MARIADB_NAME
dokku config:set $APP_NAME DB_TABLE_PREFIX=" "
dokku config:set $APP_NAME DB_USERNAME=mariadb
dokku config:set $APP_NAME DB_PASSWORD=<place-password-here>

dokku config:set $APP_NAME ADMIN_USER=admin
dokku config:set $APP_NAME ADMIN_NAME=Admin
dokku config:set $APP_NAME ADMIN_EMAIL=admin@example.com
dokku config:set $APP_NAME ADMIN_PASSWORD=admin123

dokku config:set $APP_NAME LISTEN_PORT=8080
```

## First Deployment

Since you've created and configured the app on the server, you can switch to
your local machine and execute the first app deploy! After cloning the
repository (and inside its directory), execute:

```shell
git remote add dokku dokku@<server-ip>:<app-name>
git push dokku main
```

## Finishing Installation

To finish the installtion you need to configure the SSL certificate, mount the
data path and import

Finally, on server again:

```shell
# Create/configure SSL certificate:
dokku letsencrypt:enable $APP_NAME
```
