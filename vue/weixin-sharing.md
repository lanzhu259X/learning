# vue 微信分享

> 这两天搞了下微信分享的功能，也大概了解了下微信 js-sdk 的调用逻辑。

## 微信公众号配置信息

当项目需要在微信中做分享页面时，需要在微信公众号管理平台中开通一些功能。

做分享操作时，需要早微信公众号平台开通开发者账号的功能，获取公众号 AppId 和 AppSecret, 配置 IP 白名单信息。同时确认有分享接口调用的权限。

同时，需要在 `设置->公众号设置->功能设置` 中配置 JS 接口安全域名，并且按配置安全域名的要求下载一个`txt`文件放到服务器上，能通过配置的域名获取到这个`txt`文件。

## vue 端

在项目中需要引入 weixin-js-sdk

```shell
npm install weixin-js-sdk --save
```

下面是写微信分享的一个 wxsharing.js 的文件, 创建一个微信分享的方法功能，可以在需要分享的页面进行导入.

```js
import wx from "weixin-js-sdk"; //引入weixin-js-sdk
import util from "./util"; //获取ajax的请求功能

/* vm 调用的页面component (this)*/
/* param: {title: "分享标题", desc: "描述内容", link:"分享连接", imgUrl: "分享图标URL", localUrl: "分享连接做签名的URL"} */
export default {
  sharing: (vm, param) => {
    let sharingUrl = param.localUrl;
    util.ajax
      .get("/wecat/config", { params: { sharingUrl: sharingUrl } }) //通过服务后台获取url签名信息
      .then(response => {
        let config = response.data;
        console.log(config);
        //微信会在config验证通过后，会执行ready中的方法，修改分享菜单信息
        wx.config({
          debug: false, //是否开启debug模式，开启后，在微信页面会把每一步的信息会Alert出来
          appId: config.appId, //公众号的APPID
          timestamp: config.timestamp, //签名生成的时间
          nonceStr: config.nonceStr, //签名的随机串
          signature: config.signature, //签名串
          jsApiList: ["onMenuShareAppMessage", "onMenuShareTimeline"] //需要获取微信的接口方法名
        });
        //签名验证失败
        wx.error(function(response) {
          console.log("微信签名验证失败:");
          console.log(response);
          vm.$Message.error("微信签名验证失败."); //toast显示错误信息
        });
        //签名验证成功，会执行ready方法
        wx.ready(function() {
          // 分享到朋友圈
          wx.onMenuShareTimeline({
            title: param.title, //分享标题
            link: param.link, // 分享链接
            imgUrl: param.imgUrl, //分享图标
            success: function(res) {
              console.log("分享到朋友圈成功返回的信息：", res);
              vm.$Message.success("分享到朋友圈成功");
            },
            cancel: function(res) {
              //用户取消分享后执行的回调函数
              console.log("取消分享到朋友圈返回的信息:", res);
            }
          });

          //分享给朋友
          wx.onMenuShareAppMessage({
            title: param.title, // 标题
            desc: param.desc, // 描述内容
            link: param.link,
            imgUrl: param.imgUrl,
            type: "link",
            success: function(res) {
              console.log("分享到朋友圈成功返回的信息：", res);
              vm.$Message.success("分享到朋友圈成功");
            },
            cancel: function(res) {
              //用户取消分享后执行的回调函数
              console.log("取消分享到朋友圈返回的信息:", res);
            }
          });
        });
      })
      .catch(error => {
        console.log(error);
        util.errorProcessor(error); //请求签名配置信息失败时的处理逻辑，主要是显示错误信息
      });
  }
};
```

接下来是需要在需要分享的页面中，在页面初始化的时候，调用该 wxsharing.js 中的 sharing 方法来初始化分享信息。

如下在页面的生命周期钩子函数的 actieved 或者 mounted 中调用，也就是在页面初始化的时候就要调用。

我这里由于是使用同一个分享的页面模板，只是路径参数进行修该了，所以我在路由变化的时候会初始化页面信息的时候就调用微信分享信息初始化:

```js
watch: {
    $route: "init"  // 监听路由的变化，在路由变化的时候就会触发init方法的调用.
  },
  methods: {
    init() {
        //这步判断一定要加上，因为每次路由的变化, vue都会触发该init方法的调用，就算是当前页不是在改页面也会触发。
        //这时候就需要判断下当前路由是否是当前页面的路由，如果不是直接退出不处理，如果是才进行数据初始化.
      if (this.$route.name !== "defaultsharing") {
        return;
      }
      this.sharingNo = this.$route.params.sharingNo; //分享内容的编号
      console.log(this.sharingNo);
      if (!this.sharingNo) {
        this.$Message.error("分享内容获取失败");
        this.isDown = true;
        return;
      }
      // 获取分享模本信息
      this.loadSharingModal();
    },
    loadSharingModal() {
      this.isDown = false;
      this.spinShow = true;
      util.ajax
        .get("/sharing/out/sharingmodal/" + this.sharingNo)
        .then(response => {
          this.spinShow = false;
          this.sharingModal = response.data;
          if (this.sharingModal.sharingTitle) {
            window.document.title = this.sharingModal.sharingTitle;
            //在分享模板信息获取成功后，初始化微信分享信息
            let param = {
              title: this.sharingModal.sharingTitle,  //分享的标题
              desc: this.sharingModal.sharingSubTitle, //分享的描述内容
              link: window.location.href,   //分享的页面路径，直接获取当前页的路径
              imgUrl: this.sharingModal.sharingImage
                ? this.sharingModal.sharingImage
                : "http://xxx.com/xxx.png",  //分享的小图标(这里设置了默认值)
              localUrl: window.location.href.split("#")[0]  //签名的url,去除vue路径#号后面的那部分
            };
            wxsharing.sharing(this, param);  //调用分享页信息的初始化
          }
          if (!this.sharingModal || this.sharingModal.status !== "NORMAL") {
            this.isDown = true;
          }
        })
        .catch(error => {
          this.spinShow = false;
          this.isDown = true;
          util.errorProcessor(this, error);
        });
    }
  }
```

## 服务器后台生成签名

生成分享的签名信息，需要用到微信的 `jsapi_ticket` 接口，生成 ticket 需要用到 access_token.

如下获取微信 access_token:

```java
@Slf4j
@Service
public class WXService {

    private static final String WX_ACCESS_TOKEN_CACHE_KEY = "WX_ACCESS_TOKEN";
    private static final String WX_JSAPI_TICKET_CACHE_KEY = "WX_JSAPI_TICKET";

    @Value("${wx.app.id}")
    private static final String wxAppId;
    @Value("${wx.app.secret}")
    private String wxAppSecret;
    @Autowired
    private StringRedisTemplate redisTemplate;
    @Autowired
    private RestTemplate restTemplate;

    private Lock lock = new ReentrantLock();

    public String getWxAccessToken() {
        ValueOperations<String, String> operations = redisTemplate.opsForValue();
        String cache = operations.get(WX_ACCESS_TOKEN_CACHE_KEY);
        if (StringUtils.isNotEmpty(cache)) {
            return cache;
        }

        //多线程拿时防止下多个请求同时去请求微信
        lock.lock();
        try {
            String wxAccessToken = getWxAccessTokenByWX();
            if (wxAccessToken != null) {
                //加入缓存 微信的token过期时间7200秒 (微信的access_token 每天的请求次数是有限的，需要自己做缓存,再快失效的时候去获取新的access_token)
                operations.set(WX_ACCESS_TOKEN_CACHE_KEY, wxAccessToken, 7000, TimeUnit.SECONDS);
            }
            return wxAccessToken;
        }finally {
            lock.unlock();
        }
    }

    private String getWxAccessTokenByWX() {
        // 获取微信access_token 的URL和参数, grant_type=client_credential固定值, appid 是你公众号的appId, secret 是你公共号的AppSecret的值。
        String url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=" + wxAppId + "&secret=" + wxAppSecret;
        try{
            ResponseEntity<String> response = restTemplate.getForEntity(url, String.class);
            if (response != null && response.getStatusCode() == HttpStatus.OK) {
                String body = response.getBody();
                log.info("get weixin access_token: {}", body);
                JSONObject json = JSON.parseObject(body);
                String result = (String) json.get("access_token");
                return result;
            }
        }catch (Exception e) {
            log.error("get weixin access_token fail.", e);
        }
        log.warn("get weixin access_token result is fail.");
        return null;
    }

}
```

获取微信的 access_token 后，可以根据 access_token 去获取 jsapi_ticket，代码如下:

```java
public String getWXJsapiTicket(String accessToken) {
    if (StringUtils.isEmpty(accessToken)) {
        return null;
    }
    ValueOperations<String, String> operations = redisTemplate.opsForValue();
    String cache = operations.get(WX_JSAPI_TICKET_CACHE_KEY);
    if (StringUtils.isNotEmpty(cache)) {
        return cache;
    }

    //多线程拿时防止下多个请求同时去请求微信
    lock.lock();
    try {
        String ticket = getWxjsapiTicketByWx(accessToken);
        if (ticket != null) {
            //加入缓存 微信的token过期时间7200秒（微信的jsapi_ticket的接口调用每日有次数限制，做好自己的缓存）
            operations.set(WX_JSAPI_TICKET_CACHE_KEY, ticket, 7000, TimeUnit.SECONDS);
        }
        return ticket;
    }finally {
        lock.unlock();
    }
}

private String getWxjsapiTicketByWx(String accessToken) {
    String url = "https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token="+ accessToken +"&type=jsapi";
    try{
        ResponseEntity<String> response = restTemplate.getForEntity(url, String.class);
        if (response != null && response.getStatusCode() == HttpStatus.OK) {
            String body = response.getBody();
            log.info("get weixin jsapi_ticket: {}", body);
            JSONObject json = JSON.parseObject(body);
            String result = (String) json.get("ticket");
            return result;
        }
    }catch (Exception e) {
        log.error("get weixin jsapi_ticket fail.", e);
    }
    log.warn("get weixin jsapi_ticket result is fail.");
    return null;
}
```

获取到 jsapi_ticket 后，就可以根据 jsapi_ticket 对 URL 进行签名，代码如下：

```java
public WXSignEntity getWXSignConfig(String url) throws BizException {
    String token = getWxAccessToken();
    String ticket = getWXJsapiTicket(token);
    if (StringUtils.isEmpty(ticket)) {
        log.warn("get weixin jsapi_ticket fail.");
        throw new BizException(ErrorCode.WX_JSAPI_TICKET_FAIL);
    }

    WXSignEntity entity = WXSignUtil.getWXSign(ticket, url);
    if (entity != null) {
        entity.setAppId(wxAppId);
        return entity;
    }else {
        throw new BizException(ErrorCode.WX_SIGN_CONFIG_FAIL);
    }
}
```

`WXSignEntity` 定义了需要返回给前端的字段信息

```java
@Setter
@Getter
public class WXSignEntity {
    private String url;  //签名的URL
    private String jsapi_ticket; //通过公众号access_token获取的jsapi_ticket
    private String nonceStr; //签名的随机码
    private String timestamp; //生成签名的时间戳,单位是秒
    private String signature; //生成的签名
    private String appId; // 公众号的appId
}
```

生成签名的算法如下:

```java
import com.yiban.sharing.entities.WXSignEntity;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.RandomStringUtils;
import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Formatter;

/**
 * 微信签名
 */
@Slf4j
public class WXSignUtil {

    /**
     * 微信分享url签名
     * @param jsapiTicket 通过微信access_token获取的jsapi_ticket
     * @param url 当前网页的URL，不包含#及其后面部分
     * @return
     */
    public static WXSignEntity getWXSign(String jsapiTicket, String url) {
        String nonce_str = RandomStringUtils.randomAlphanumeric(10); //随机码
        String timestamp = Long.toString(System.currentTimeMillis() / 1000); //签名时间 秒

        String signature = "";

        // 注意这里参数名必须全部小写，且必须有序
        String string1 ="jsapi_ticket=" + jsapiTicket + "&noncestr=" + nonce_str + "&timestamp=" + timestamp + "&url=" + url;
        try {
            MessageDigest crypt = MessageDigest.getInstance("SHA-1"); //需要使用SHA1加密
            crypt.reset();
            crypt.update(string1.getBytes("UTF-8"));
            signature = byteToHex(crypt.digest());
        } catch (NoSuchAlgorithmException e) {
            log.error("NoSuchAlgorithmException ", e);
        } catch (UnsupportedEncodingException e) {
            log.error("UnsupportedEncodingException ", e);
        }

        WXSignEntity entity = new WXSignEntity();
        entity.setJsapi_ticket(jsapiTicket);
        entity.setNonceStr(nonce_str);
        entity.setSignature(signature);
        entity.setTimestamp(timestamp);
        entity.setUrl(url);
        return entity;
    }

    //生成签名
    private static String byteToHex(final byte[] hash) {
        Formatter formatter = new Formatter();
        for (byte b : hash)
        {
            formatter.format("%02x", b);
        }
        String result = formatter.toString();
        formatter.close();
        return result;
    }
}
```

测试代码:

```java
@Test
public void getWXSignConfig() {
    try {
        // 签名的url不能有#号后面这一段 #/xxx
        WXSignEntity entity = wxService.getWXSignConfig("http://xxx.xxx.com/dist/");
        log.info(JSON.toJSONString(entity));
    } catch (BizException e) {
        log.error("BizException ", e);
    }
}
```
