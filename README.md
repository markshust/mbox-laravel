# 🟧 Mbox for Laravel

You've stumbled upon Mbox, a Docker development environment for Laravel.

<img src="https://img.shields.io/badge/maintained%3F-yes-brightgreen.svg" alt="Maintained - Yes" />

This repository is a streamlined Docker development environment for Laravel. It uses compact Docker images like Alpine, includes SSL and Xdebug support out of the box, and is aimed at providing you with a highly configurable environment.

Note that Docker images are not pre-built, but will be automatically built on your machine when you run `docker compose up -d` for the first time. This means that you do not need to rely on updates from remote repositories, and can easily customize the images to your liking. 

## Table of contents
- [Why?](#why)
- [Setup](#setup)
  - [Laravel](#laravel)
  - [dnsmasq](#dnsmasq)
  - [SSL/HTTPS](#sslhttps)
  - [Xdebug](#xdebug)
- [Usage](#usage)
- [Credits](#credits)
- [License](#license)

## Why?

Solutions like Valet and Sail are too bloated, cumbersome, and contain too much magic. They are also too restrictive, and cannot easily be ported to or used within production environments.

Mbox is meant to get your Laravel instance up & running with minimal hassle, and includes automated SSL termination with Traefik, as well as full support for Vite HMR over Docker + SSL.

## Setup

### Laravel

Copy these directories and files over to your Laravel project:
- `docker`: Docker images and configuration files
- `compose.yaml`: Docker Compose configuration
- `vite.config.js`: Vite configuration

**Important:** In `vite.config.js`, update the HMR host from `acme.test` to your actual domain:
```javascript
hmr: {
    host: 'yourproject.test',  // Replace with your domain
    protocol: 'wss',
    clientPort: 443,
    path: 'wss'
}
```

Copy the values from `.env.docker` over to your main `.env` file, and modify them as necessary.

- `DOCKER_TRAEFIK_IDENTIFIER`: set this to your project name, ex. `acme`
- `DOCKER_TRAEFIK_DOMAIN`: set this to your domain, ex. `acme.test`
- `FORWARD_PORT_*`: set these to the ports you want to forward to the respective services
- `MARIADB_*`: set these to the values you want for your MariaDB instance

You will also want to update your Laravel env vars, specifically `HOST` values, to match their respective Docker service names and ports. For example, `DB_HOST` should be changed from `localhost` to `mariadb`, as `mariadb` is the name of the service as defined within Docker.

> **Note:** You should still use either `localhost` or your local .test domain when connecting to services such as MySQL from your host machine, as Docker will automatically route requests to the correct service.

### dnsmasq

This project relies on dnsmasq to route requests made to `*.test` to `localhost`. You can easily set this up by running:

```bash
brew install dnsmasq
mkdir -pv $(brew --prefix)/etc/
echo 'address=/.test/127.0.0.1' >> $(brew --prefix)/etc/dnsmasq.conf
sudo brew services start dnsmasq
sudo mkdir -v /etc/resolver
sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/test'
```

### SSL/HTTPS

This project uses Traefik to terminate SSL, so you will need to generate an SSL certificate for your domain.

Install `mkcert` to generate local SSL certs:

```yaml
brew install mkcert
mkcert -install
```

Then generate a certificate for your domain (replace `acme.test` with your actual domain):

```yaml
mkcert -key-file docker/ssl/ssl.key -cert-file docker/ssl/ssl.crt "acme.test"
```

And set proper permissions on the cert and key:

```diff
chmod 600 docker/ssl/ssl.crt
chmod 400 docker/ssl/ssl.key
```

You'll also want to be sure to add the `/docker/ssl` directory in your `.gitignore` file.

### Xdebug

Xdebug is automatically enabled and set to `debug` mode by default. If you wish to disable or change this setting, modify the `docker/images/phpfpm/conf/php.ini` file and then restart the phpfpm container.

## Usage

- `docker compose up -d`: Start the containers
- `docker compose down`: Stop the containers
- `docker compose restart`: Restart the containers
- `docker compose restart phpfpm`: Restart the phpfpm container
- `docker compose up -d --build`: Rebuild the containers
- `docker compose exec phpfpm php`: Run PHP commands
- `docker compose exec node npm`: Run npm commands

## Credits

### M.academy

This repository is sponsored by <a href="https://m.academy" target="_blank">M.academy</a>, the simplest way to learn complex tech skills.

<a href="https://m.academy" target="_blank"><img src="https://m.academy/images/logo.png" alt="M.academy"></a>

## License

[MIT](https://opensource.org/licenses/MIT)
