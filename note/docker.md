### Usage
```
docker-compose up
```

### Floder tree
```
.
├── docker-compose.yml
└── nginx
    └── conf.d
        └── default.conf
```

### Q&A
* PATH_INFO?
  * `docker-compose exec wordpress grep -rnw '/var/www/html/' -e "PATH_INFO" -C5`
* PATH_TRANSLATED?
  * `docker-compose exec wordpress grep -rnw '/var/www/html/' -e "PATH_TRANSLATED" -C5`
  
### ref
* [https://docs.docker.com/compose/compose-file/compose-file-v2/#usernsmode](https://docs.docker.com/compose/compose-file/compose-file-v2/#usernsmode)
* [https://gist.github.com/md5/d9206eacb5a0ff5d6be0#file-wordpress-fpm-conf](https://gist.github.com/md5/d9206eacb5a0ff5d6be0#file-wordpress-fpm-conf)
* [https://www.nginx.com/resources/wiki/start/topics/examples/phpfcgi/](https://www.nginx.com/resources/wiki/start/topics/examples/phpfcgi/)

### default.conf
```
server {
    listen      80;
    listen [::]:80;
    server_name localhost;

    root /var/www/html;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

#    rewrite /wp-admin$ $scheme://$host$uri/ permanent;

    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        if (!-f $document_root$fastcgi_script_name) {
            return 404;
        }
        # Mitigate https://httpoxy.org/ vulnerabilities
        fastcgi_param HTTP_PROXY "";

        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;

        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO       $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
    }
}
```

### docker-compose.yml
```
version: '2'

services:

  wordpress:
    depends_on:
      - mysql
    image: wordpress:php7.1-fpm-alpine
    environment:
      WORDPRESS_DB_PASSWORD: admin

  mysql:
    image: mariadb:10
    environment:
      MYSQL_ROOT_PASSWORD: admin

  nginx:
    depends_on:
      - wordpress
    image: nginx:1-alpine
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
    volumes_from:
      - wordpress
    ports:
      - 8080:80
```
