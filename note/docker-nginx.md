### nginx on vm1
```
docker run \
-d \
--restart=always \
--p 80:80 \
--v /path/to/default.conf:/etc/nginx/conf.d/default.conf \
nginx:1-alpine

```

### php-fpm on vm2
```
docker run \
-d \
--restart=always \
-p 9000:9000 \
\
# must same as default.conf root
-v /path/to/code:/var/www/html \
mikechernev/php7-fpm-laravel
```

### default.conf
```
upstream remote_fpm {
    server fpm:9000;
}
server {
    listen 80;

    server_name localhost;

    # must same as php-fpm folder.
    root /var/www/html;
    location / {
        rewrite ^ /index.php break;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass remote_fpm;
        fastcgi_index index.php;

        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```
