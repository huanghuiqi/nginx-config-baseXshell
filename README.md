# nginx-config-baseXshell
在xshell下配置nginx服务器

###安装nginx1.8
```wget http://nginx.org/packages/centos/6/x86_64/RPMS/nginx-1.8.0-1.el6.ngx.x86_64.rpm```   

```rpm -ivh nginx-1.8.0-1.el6.ngx.x86_64.rpm```   

```yum info nginx```   

```service nginx start```   

```service nginx reload```   


###配置:(每次修改完配置都需要service nginx reload一遍)
加粗部分名字可以自行修改
```vi /etc/nginx/conf.d/**api.testing.com**.conf```  

```
server {
        listen 80;
        server_name  api.testing.com;
        root  /data/www/api.testing.com;      
	//root 默认为是你的公网ip地址比如 120.25.79.66 如果在api.testing.com下创建一个test文件夹里面一个index.html   
	//则通过 http:// 120.25.79.66/test/index.html打开
        charset utf8;

	location / {
		index index.html index.htm index.php;
	}
	location ~ \.php$ {
		fastcgi_split_path_info ^(.+\.php)(/.+)$;
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_index index.php;
		include fastcgi_params;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	}

        access_log  /data/logs/nginx/ api.testing.access.log;
        error_log  /data/logs/nginx/ api.testing.error.log;
}
```
