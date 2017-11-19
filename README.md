# Thinkphp3.2-WechatShare
thinkphp3.2结合微信公众号的分享接口实现朋友、朋友圈的分享（这里是用测试号为例子）
### 第一步、在微信公众号填写接口配置信息
* URL项填写你控制器里的微信入口方法，如下：
```php
http://www.test.com/test/index.php/Home/test/wechat
```
* 控制器的微信入口方法
```php
 public function wechat(){
    //获得参数 signature nonce token timestamp echostr
    $nonce     = $_GET['nonce'];
    $token     = 'token';
    $timestamp = $_GET['timestamp'];
    $echostr   = $_GET['echostr'];
    $signature = $_GET['signature'];
    //形成数组，然后按字典序排序
    $array = array($nonce, $timestamp, $token);
    sort($array);
    //拼接成字符串,sha1加密 ，然后与signature进行校验
    $str = sha1( implode( $array ) );
    if( $str  == $signature && $echostr ){
        //第一次接入weixin api接口的时候
        echo  $echostr;
        exit;
    }else{
        $this->reponseMsg();
    }
}
```          
* Token项填写入口方法的token值，如上：token
### 第二步、编写访问入口方法(这里会用到oauth2协议)
```php
public function entrance(){
  echo "<script language='javascript'>"; 
  // 授权过后跳转的地址
  echo " location='https://open.weixin.qq.com/connect/oauth2/authorize?appid=你的     appid&redirect_uri=http://www.test.com/test/index.php/Home/wechat/oauth2&response_type=code&scope=snsapi_userinfo&state=1&from=singlemessage#wechat_redirect ';"; 
  echo "</script>";
}
``` 
### 第三步、编写oauth2方法
```php
public function oauth2(){
  session_start();
         //关闭notice提示
  error_reporting(E_ALL || ~E_NOTICE); 
  $code = $_GET["code"];
  //echo $code;
  $appid = "你的appid";//公众号的appid
  $appsecret = "你的appsecret"; //公众号的appsecret
  //以下为获取全局access_token
  $url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid=$appid&secret=$appsecret&code=$code&grant_type=authorization_code";//使用code获取access_token
  $ch = curl_init();
  curl_setopt($ch, CURLOPT_URL, $url);  // 设置你需要抓取的URL
  curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);  //这两句用于验证第三方服务器与微信服务器的安全性
  curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);  //这两句用于验证第三方服务器与微信服务器的安全性
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  // 设置cURL 参数，要求结果保存到字符串中还是输出到屏幕上。1是不输出在页面上
  $output = curl_exec($ch);   //得到的access_token赋值给$output
  curl_close($ch);  
  $jsoninfo = json_decode($output, true);  
  $access_token = $jsoninfo["access_token"];  
  $openid = $jsoninfo["openid"];
  
  //以下为获取openid//当获取全局access_token的时候，需要这段代码
  $url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid=$appid&secret=$appsecret&code=$code&grant_type=authorization_code";
  $ch = curl_init();
  curl_setopt($ch, CURLOPT_URL, $url);  // 设置你需要抓取的URL
  curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);  //这两句用于验证第三方服务器与微信服务器的安全性
  curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);  //这两句用于验证第三方服务器与微信服务器的安全性
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  // 设置cURL 参数，要求结果保存到字符串中还是输出到屏幕上。1是不输出在页面上
  $output = curl_exec($ch);   //得到的access_token赋值给$output
  curl_close($ch);  
  $jsoninfo = json_decode($output, true);

  function https_request($url)  
  {         
      $curl = curl_init();         
      curl_setopt($curl, CURLOPT_URL, $url);         
      curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, FALSE);         
      curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, FALSE);         
      curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);         
      $data = curl_exec($curl);         
      if (curl_errno($curl)) {return 'ERROR '.curl_error($curl);}         
      curl_close($curl);         
      return $data;  
  }   
  function getInfo($access_token,$openid){  
  //$url = "https://api.weixin.qq.com/cgi-bin/user/info?access_token=$access_token&openid=$openid&lang=zh_CN";  
  $url = "https://api.weixin.qq.com/sns/userinfo?access_token=$access_token&openid=$openid&lang=zh_CN";  
  $output = https_request($url);  
  $jsoninfo = json_decode($output);  //获取到的用户信息，数组型

  //存入session
  $_SESSION['headimgurl'] = $jsoninfo -> headimgurl;
  $_SESSION['openid'] = $jsoninfo -> openid;
  $_SESSION['nickname'] = $jsoninfo -> nickname;
  $_SESSION['sex'] = $jsoninfo -> sex;
  $_SESSION['language'] = $jsoninfo -> language;
  $_SESSION['country'] = $jsoninfo -> country;
  // $_SESSION['subscribe'] = $jsoninfo -> subscribe;//是否关注公众号，只有在全局access_token的时候才可以获取
  }  

  getInfo($access_token,$openid);
      // echo $_SESSION['openid'];
  echo "<script language='javascript'>"; 
  // 授权过后跳转的地址
  // echo " location='http://www.lambdass.com/Lambdass-UI/h5/index.php';"; 
  echo " location='http://www.test.com/test/index.php/Home/wechat/fenxiang';"; //这是回调地址
  echo "</script>";      
    }
```
### 第四步、解压jssdk文件，里面有三个文件，把jssdk.class.php放在Thinkphp\Library\Org\Util目录下
### 第五步、把剩下的access_token.txt和jsapi_ticket.txt放在Public\Home\jssdk目录下(本例子为示范),目的是记录token值和ticket值
### 第六步、在common\common\function目录下编写获取或新端的配置消息
```php
function wx_share_init() {  
    import("Org.Util.Jssdk");
    $wxconfig = array();   
    $config = APP_DEBUG ? C("WECHAT_SDK_TEST") : C("WECHAT_SDK"); //这里配置了微信公众号的AppId和AppSecret  
    $jssdk = new JSSDK("你的appid", "你的appsecret");
    $wxconfig = $jssdk->GetSignPackage();  
    return $wxconfig;  
} 
```
### 第七步、编写回调方法fenxiang()
```php
public function fenxiang(){
    $wxconfig = wx_share_init(); //这里调用了function.php里面的自定义方法
    $this->assign('fenxiang', $wxconfig);//把值传到视图
    $this->display('fenxiang');//分配视图
}
```
### 第八步、在视图编写微信配置的方法和分享的方法
```javascript
<script src="//res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>  
<script>  

    wx.config({  
        debug: false, // 是否开启调试模式，这个很有用，分享不成功是可以开启看看什么问题
        appId: '{$wxconfig["appId"]}', // 必填，微信号AppID  
        timestamp: '{$fenxiang["timestamp"]}', // 必填，生成签名的时间戳  
        nonceStr: '{$fenxiang["nonceStr"]}', // 必填，生成签名的随机串  
        signature: '{$fenxiang["signature"]}',// 必填，签名，见附录1  
        jsApiList: [
        'onMenuShareTimeline', //分享到朋友圈  
        'onMenuShareAppMessage', //分享给朋友  
        'onMenuShareQQ' //分享到QQ  
        ] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2  
    });  

    wx.ready(function(){  
        var options = {  
            title: '点开看看呗', // 分享标题  
            link: 'http://www.test.com/test/index.php/Home/wechat/fenxiang', // 分享链接，记得使用绝对路径，不能用document.URL
            imgUrl: 'http://wx.qlogo.cn/mmopen/vi_32/rKziaiauzDTSYzLUR7lMIY0c7LibwrNMUoBicU8yC9rPjJWibz5MMTVXciaHmMvXWoYiaakHn7fMt9GvmAeLSpFAeo8WQ/0', // 分享图标，记得使用绝对路径  
            desc: '测试', // 分享描述  
            success: function () {  
                console.info('分享成功！');  
                // 用户确认分享后执行的回调函数  
            },  
            cancel: function () {  
                console.info('取消分享！');  
                // 用户取消分享后执行的回调函数  
            }  
        }  
        wx.onMenuShareTimeline(options); // 分享到朋友圈  
        wx.onMenuShareAppMessage(options); // 分享给朋友  
        wx.onMenuShareQQ(options); // 分享到QQ  
    });  
</script>  
```
