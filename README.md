#Nginx Study 

# 设置php-fpm使用socket文件

###1、修改php文件php-fpm.conf

>listen = 127.0.0.1:9000

修改为

>listen = /var/run/phpfpm.sock

>;listen.owner = nobody

>;listen.group = nobody

>;listen.mode = 0660

将上面3行注释（;）去掉，修改成 和nginx一样的用户和组，
否则会报“nginx error connect to phpfpm.sock failed (13: Permission denied)”错误
即，无法访问phpfpm.sock，如果想所有用户都能用修改下listen.mode
修改后为：

>listen.owner = www

>listen.group = www

>listen.mode = 0666


重启php-fpm 
>/usr/local/php/sbin/php-fpm restart

###2、配置nginx
在/usr/local/nginx/conf/nginx.conf中找到

>fastcgi_pass 127.0.0.1:9000;

改为

>fastcgi_pass unix:/var/run/phpfpm.sock;

重启nginx

>/usr/local/nginx/sbin/nginx -s reload
