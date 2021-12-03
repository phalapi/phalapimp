## 微信扩展 \PhalApi\WeiXin

微信公众号、企业号等开发扩展, 基于[Eastwechat](https://www.easywechat.com)。

1. 支持公众号、企业微信等，具体参考[Eastwechat文档](https://www.easywechat.com/docs),传入不同的Service就可以调用不同的服务。
2. 只是简单的提供了一个调用方法，内置默认服务。
3. 目前该扩展我主要是用于微信订阅号，本文的例子也是微信订阅号，传入的服务名为`officialAccount`;
4. 扩展msg目录里面的类是为了支持IDE友好提示而设立的。

notice:修正啦[phalapi-weixin](https://github.com/chenall/phalapi-weixin)的错误。

### 安装

在项目下直接运行以下命令即可完成扩展安装。

```
composer require phalapi/phalapimp:dev-master
```

### 配置

根据要使用的不同的服务类型添加相应的配置文件 配置文件名 **wx_Service.php**
本例为订阅号

配置文件./config/wx_officialAccount.php
```php
<?php
return array(
    /**
     * 扩展类库 - 微信公众号配置
     * 注： 具体配置参数请参考 https://www.easywechat.com/docs
     */

    'app_id'  => '',         // AppID
    'secret'  => '',     // AppSecret
    'token'   => '',          // Token
    'aes_key' => '',                    // EncodingAESKey，兼容与安全模式下请一定要填写！！！

    /**
     * 指定 API 调用返回结果的类型：array(default)/collection/object/raw/自定义类名
     * 使用自定义类名时，构造函数将会接收一个 `EasyWeChat\Kernel\Http\Response` 实例
     */
    'response_type' => 'array',

    /**
     * 日志配置
     *
     * level: 日志级别, 可选为：
     *         debug/info/notice/warning/error/critical/alert/emergency
     * path：日志文件位置(绝对路径!!!)，要求可写权限
     */
    'log' => [
        'level' => 'debug',
        'file' =>'../runtime/wechat/'.date("Ym").'/'.date("Ymd").'.log',
    ],

);
```

### 注册

在./config/di.php文件中，注册微信服务(可以按需注册多个服务);

```php
//订阅号
$di->officialAccount = function(){
    $weixin = new \PhalApi\WeiXin\Lite('officialAccount');
    return $weixin->wxApp();
};
```
### 服务端访问入口(消息接收处理,需要单独的访问入口) 

需要对接收的信息进行处理时才需要使用

```php
//./Public/WeiXin.php
/**
 * 统一访问入口
 */
require_once dirname(__FILE__) . '/init.php';

$server = new \PhalApi\WeiXin\Lite('officialAccount');
//默认使用内置的消息处理 \PhalApi\WeiXin\Server
$server->response();
```

也可以使用自己定义的消息处理接口

```php
<?php
//./Public/WeiXin.php
/**
 * 统一访问入口
 */
require_once dirname(__FILE__) . '/init.php';

$server = new \PhalApi\WeiXin\Lite('officialAccount');
//默认使用内置的消息处理 \PhalApi\WeiXin\Server
$server->Server('\\App\\WeiXin\\WeiXinServer');
```
```php
<?php
//./src/WeiXin/WeiXinServer.php
namespace App\WeiXin;
use PhalApi\WeiXin\Server;

class WeiXinServer extends Server {
    
   /**
        * 收到文本消息
        * @param  $message
        * @return mixed 
        */
 public function text($message){
     
     return'自定义回复消息';
 }
  /**
     * 事件推道
     * @param  $message
     * @return mixed 
     */
    public function event($message){
        switch ($message['Event'])
		{
		case 'subscribe':
		    //被关注时自动回复
  		 return'你好欢迎关注';
  		break;  
		default:
  			 return'收到事件消息';
}
    }
  
  
}

```
