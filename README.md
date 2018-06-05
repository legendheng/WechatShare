# WechatShare
thinkphp3.2结合微信公众号的分享接口实现朋友、朋友圈的分享（这里是用测试号为例子）
### 第一步、解压jssdk文件，里面有三个文件，把jssdk.class.php放在Thinkphp\Library\Org\Util目录下
### 第二步、把剩下的access_token.txt和jsapi_ticket.txt放在Public\Home\jssdk目录下(本例子为示范),目的是记录token值和ticket值
### 第三步、在common\common\function目录下编写获取或新端的配置消息
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
### 第四步、编写回调方法fenxiang()
```php
public function fenxiang(){
    $wxconfig = wx_share_init(); //这里调用了function.php里面的自定义方法
    $this->assign('fenxiang', $wxconfig);//把值传到视图
    $this->display('fenxiang');//分配视图
}
```
### 第五步、在视图编写微信配置的方法和分享的方法
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
            link: 'http://www.test.com/test/index.php/Home/wechat/fenxiang', // 注意这里的url必须是微信公众号配置js域名的目录下，否则即使提示分享成功，但图片、标题、链接都不能生效
            imgUrl: 'http://wx.qlogo.cn/mmopen/vi_32/rKziaiauzDTSYzLUR7lMIY0c7LibwrNMUoBicU8yC9rPjJWibz5MMTVXciaHmMvXWoYiaakHn7fMt9GvmAeLSpFAeo8WQ/0', // 分享图标，记得使用绝对路径  
            desc: '测试', // 分享描述  
            success: function () {  
              alert("分享成功");
            },  
            cancel: function () {  
               
            }  
        }  
        wx.onMenuShareTimeline(options); // 分享到朋友圈  
        wx.onMenuShareAppMessage(options); // 分享给朋友  
        wx.onMenuShareQQ(options); // 分享到QQ  
    });  
      wx.ready(function () {
         wx.onMenuShareTimeline({
             title: '测试', // 分享标题
            link: 'http://www.test.com/test/index.php/Home/wechat/fenxiang', // 注意这里的url必须是微信公众号配置js域名的目录下，否则即使提示分享成功，但图片、标题、链接都不能生效
             imgUrl: 'http://wx.qlogo.cn/mmopen/vi_32/rKziaiauzDTSYzLUR7lMIY0c7LibwrNMUoBicU8yC9rPjJWibz5MMTVXciaHmMvXWoYiaakHn7fMt9GvmAeLSpFAeo8WQ/0', // 分享图标，记得使用绝对路径
             desc: '测试分享功能', // 分享描述
             success: function () {
               alert("分享成功");
            },
            cancel: function () {
            }
         });
         //分享给朋友
         wx.onMenuShareAppMessage({
             title: '测试', // 分享标题
            link: 'http://www.test.com/test/index.php/Home/wechat/fenxiang', // 注意这里的url必须是微信公众号配置js域名的目录下，否则即使提示分享成功，但图片、标题、链接都不能生效
             imgUrl: 'http://wx.qlogo.cn/mmopen/vi_32/rKziaiauzDTSYzLUR7lMIY0c7LibwrNMUoBicU8yC9rPjJWibz5MMTVXciaHmMvXWoYiaakHn7fMt9GvmAeLSpFAeo8WQ/0', // 分享图标，记得使用绝对路径
             desc: '测试分享功能', // 分享描述
            success: function () {
                alert("分享成功");
            },
            cancel: function () {
            }
         });

         wx.onMenuShareQQ({
             title: '测试', // 分享标题
            link: 'http://www.test.com/test/index.php/Home/wechat/fenxiang', // 注意这里的url必须是微信公众号配置js域名的目录下，否则即使提示分享成功，但图片、标题、链接都不能生效
             imgUrl: 'http://wx.qlogo.cn/mmopen/vi_32/rKziaiauzDTSYzLUR7lMIY0c7LibwrNMUoBicU8yC9rPjJWibz5MMTVXciaHmMvXWoYiaakHn7fMt9GvmAeLSpFAeo8WQ/0', // 分享图标，记得使用绝对路径
             desc: '测试分享功能', // 分享描述
            success: function () {
                alert("分享成功");
            },
            cancel: function () {
            }
         });
     });
</script>  
```
