# nginx-config-baseXshell
在xshell下配置nginx服务器

### 安装nginx1.8   

```wget http://nginx.org/packages/centos/6/x86_64/RPMS/nginx-1.8.0-1.el6.ngx.x86_64.rpm```   

```rpm -ivh nginx-1.8.0-1.el6.ngx.x86_64.rpm```   

```yum info nginx```   

```service nginx start```   

```service nginx reload```   


### 配置:(每次修改完配置都需要service nginx reload一遍)
加粗部分名字可以自行修改   

vi /etc/nginx/conf.d/**api.testing.com**.conf

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

——————————————————————————————————————————————————————————————————————————————————————
# PHP中利用PHPMailer配合QQ邮箱实现发邮件
### PHPMailer的获取：
PHPMailer项目地址：https://github.com/PHPMailer/PHPMailer 使用Git命令克隆到本地，或直接在该项目页面的右下方点击“ Download ZIP ”即可获取到完整的PHPMailer代码包，再到本地解压即可。

###  获取smtp的登陆密码
登陆个人邮箱，邮箱设置-账户-POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务，开启后记录好授权码

###  使我们的PHP能够使用QQ邮箱发送邮件
PHPMailer需PHP的socket扩展支持，而PHPMailer链接qq域名邮箱时需要ssl加密方式，故php还得openssl的支持。     
通过`phpinfo()`查看`Sockets Support`和`OpenSSL Support`是否是enable,如果未开启则不能发送邮件

###  编写发送邮件代码
```
<?php
/*发送邮件方法
 *@param $to：接收者 $title：标题 $content：邮件内容
 *@return bool true:发送成功 false:发送失败
 */

function sendMail($to,$title,$content){

    //引入PHPMailer的核心文件 使用require_once包含避免出现PHPMailer类重复定义的警告,存放
    路径自己定义   
    require_once("PHPMailer/class.phpmailer.php");
    require_once("PHPMailer/class.smtp.php");
    //实例化PHPMailer核心类
    $mail = new PHPMailer();

    //是否启用smtp的debug进行调试 开发环境建议开启 生产环境注释掉即可 默认关闭debug调试模式
    $mail->SMTPDebug = 1;

    //使用smtp鉴权方式发送邮件
    $mail->isSMTP();

    //smtp需要鉴权 这个必须是true
    $mail->SMTPAuth=true;

    //链接qq域名邮箱的服务器地址
    $mail->Host = 'smtp.qq.com';

    //设置使用ssl加密方式登录鉴权
    $mail->SMTPSecure = 'ssl';

    //设置ssl连接smtp服务器的远程服务器端口号，以前的默认是25，但是现在新的好像已经不可用了 可选465或587
    $mail->Port = 465;

    //设置smtp的helo消息头 这个可有可无 内容任意
    // $mail->Helo = 'Hello smtp.qq.com Server';

    //设置发件人的主机域 可有可无 默认为localhost 内容任意，建议使用你的域名
    //$mail->Hostname = 'http://www.lsgogroup.com';
    //$mail->Hostname = 'localhost';

    //设置发送的邮件的编码 可选GB2312 我喜欢utf-8 据说utf8在某些客户端收信下会乱码
    $mail->CharSet = 'UTF-8';

    //设置发件人姓名（昵称） 任意内容，显示在收件人邮件的发件人邮箱地址前的发件人姓名
    $mail->FromName = '测试内容';

    //smtp登录的账号 这里填入字符串格式的qq号即可
    $mail->Username ='564371347@qq.com';

    //smtp登录的密码 使用生成的授权码（就刚才叫你保存的最新的授权码） IMAP/SMTP
    $mail->Password = 'iardktdjnkgybefg';

    //设置发件人邮箱地址 这里填入上述提到的“发件人邮箱”
    $mail->From = '564371347@qq.com';

    //邮件正文是否为html编码 注意此处是一个方法 不再是属性 true或false
    $mail->isHTML(true);

    //设置收件人邮箱地址 该方法有两个参数 第一个参数为收件人邮箱地址 第二参数为给该地址设置的昵称 不同的邮箱系统会自动进行处理变动 这里第二个参数的意义不大
    $mail->addAddress($to,'lsgo在线通知');

    //添加多个收件人 则多次调用方法即可
    // $mail->addAddress('xxx@163.com','lsgo在线通知');

    //添加该邮件的主题
    $mail->Subject = $title;

    //添加邮件正文 上方将isHTML设置成了true，则可以是完整的html字符串 如：使用file_get_contents函数读取本地的html文件
    $mail->Body = $content;

    //为该邮件添加附件 该方法也有两个参数 第一个参数为附件存放的目录（相对目录、或绝对目录均可） 第二参数为在邮件附件中该附件的名称
    // $mail->addAttachment('./d.jpg','mm.jpg');
    //同样该方法可以多次调用 上传多个附件
    // $mail->addAttachment('./Jlib-1.1.0.js','Jlib.js');

    $status = $mail->send();

    //简单的判断与提示信息
    if($status) {
        return true;
    }else{
        return false;
    }
}

$flag = sendMail('564371347@qq.com','test mail','hello mimi');
if($flag){
    echo "发送邮件成功！";
}else{
    echo "发送邮件失败！";
}
```
