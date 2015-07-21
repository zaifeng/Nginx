#Nginx Study 

# 设置php-fpm使用socket文件

###1、修改php文件php-fpm.conf

>listen = 127.0.0.1:9000

修改为

>listen = /var/run/phpfpm.sock

找到
>;listen.owner = nobody

>;listen.group = nobody

>;listen.mode = 0660

去掉注释（;），修改成和nginx一样的用户和组，
否则会报错“nginx error connect to phpfpm.sock failed (13: Permission denied)”
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


# 获取Nginx编译阶段参数

> /usr/local/nginx/sbin/nginx -V 

> 注意V大写 小写v 是查看nginx版本


# 备忘

> Nginx配置文件分为好多块，常见的从外到内依次是「http」、「server」、「location」等等，缺省的继承关系是从外到内，也就是说内层块会自动获取外层块的值作为缺省值


##upstream 设置
//outside server block
upstream hellostream 
{
    server 127.0.0.1:8080;
    server 127.0.0.1:9000;    
    ip_hash;
}

// ip_hash 作用是对ip进行hash，使每次都分配到同一台服务器上

// in server -> location block
proxy_pass http://hellowstream


##负载均衡 中一台服务器宕机 处理情况

设置 timeout

    proxy_connect_timeout       1;
    proxy_read_timeout          1;
    proxy_send_timeout          1; 

借用网上一位哥们的话：

楼上说的不够准确，nginx的健康检查机制确实存在缺陷。
2个节点，其中一个宕机了，nginx还是会分发请求给它，但是发出后觉得不对劲，没有响应，顿了一下，然后发给另一个节点。默认1分钟内不会再发请求，一分钟后重复上述动作。这样的结果是网站时快时慢，间歇性抽风。

有时间还是要看下nginx源码
