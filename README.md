## Require

* [troglobit/redir: A TCP port redirector for UNIX](https://github.com/troglobit/redir)
  * Port mapping to caddy in rootless container
  *  `redir --lport=10080 --cport=80`
* Symlink Caddyfile
* Install composer
  * `podman run --rm -it   -v ./app:/var/www/html   -w /var/www/html   docker.io/library/composer:2   composer install`
  * Index page show error `Vendor path not found. Please execute "bin/composer.phar run install:prod" on the command line in the web root.`