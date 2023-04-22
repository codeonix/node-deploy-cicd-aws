## Launch an EC2 instance and install Node.js, Nginx, and PM2:

```
sudo apt-get update
sudo apt-get install -y nodejs npm nginx
sudo npm install -g pm2
```

## Clone your app from your Git repository onto the EC2 instance

```
git clone repo

```

## Install NVM : 
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

## Install node version required : 
```nvm install 16 ```

## Install the required dependencies using npm install.

```
cd your-sveltekit-app
yarn
```

## Build your SvelteKit app using npm run build.

```
yarn build
```

## Start your app using PM2 by running pm2 start npm --name "app-name" -- start.

```
pm2 start npm --name "app-name" -- start

```

## Configure Nginx as a reverse proxy to forward requests to your app.

```sudo nano /etc/nginx/conf.d/your-domain-name.conf

```

## Paste the following content into the file, replacing your-domain-name.com with your actual domain name or your EC2 instance's public IP address:

```server {
  listen 80;
  server_name your-domain-name.com;
  location / {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

## Restart Nginx to apply the configuration changes:

```
sudo service nginx restart

```

## To install Certbot on Ubuntu, you can run the following commands:

```
sudo apt update
sudo apt install certbot

```

## Run Certbot with the certonly command to obtain the SSL certificate. For example, to obtain a certificate for the domain xyz.com, you would run:

```
sudo certbot certonly --webroot -w /var/www/html -d xyz.com

```

## Update Nginx config :

```
server {
    listen 443 ssl;
    server_name xyz.com;

    ssl_certificate /etc/letsencrypt/live/adsv2.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/adsv2.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

## Restart Nginx to apply the configuration changes:

```
sudo service nginx restart

```

## Write workflow script : .github/main.yml

```
name: Deployment

on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16.x]

    steps:
      - uses: actions/checkout@v2
      - name: Use node js
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: npm install and build
        run: |
          npm install
          npm run build
        env:
          CI: true

  deploy:
    needs: [build]
    runs-on: ubuntu-latest

    steps:
      - name: SSH deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST}}
          username: ${{ secrets.USER}}
          key: ${{ secrets.KEY}}
          port: ${{ secrets.PORT}}
          script: |
            curl -o-   https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh    | bash
            . ~/.nvm/nvm.sh
             nvm instal 16
             export NVM_DIR=$HOME/.nvm;
             source $NVM_DIR/nvm.sh;
             nvm use 16
             cd $HOME/blink
             git fetch --all
             git reset --hard origin/master
            npm install
            npm run build
            pm2 restart app-name
```

## Add Credentials :

- SSH Private Key : ssh-keygen on instance and set that private key in repo secrets. And public key to ~/.ssh/authorized_keys file

- HOST : public IP of instance
- USER : ubuntu
- PORT : 22 ( for ssh )
