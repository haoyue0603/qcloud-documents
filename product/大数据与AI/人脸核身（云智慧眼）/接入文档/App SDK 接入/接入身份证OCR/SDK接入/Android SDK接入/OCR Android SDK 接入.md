## 开发准备
### 权限检测
SDK 需要用到相机权限。
#### Android 6.0 及以上系统
SDK 运行时检测权限，需要用户授权。
#### Android 6.0 以下系统
- Android 没有运行时权限，检测权限只能靠开关相机进行。
- 由于 SDK 的使用时间较短，为避免快速频繁开关相机可能会导致的手机抛出异常，故对 Android 6.0 以下手机没有做权限的检测。
- 为进一步提高用户体验，在 Android 6.0 以下系统上，我们建议合作方在拉起 SDK 前，帮助 SDK 做以下权限检测：
   - 相机 
 
>?提示用户确认打开权限后再进行身份证 OCR 识别，可以使身份证识别体验更快更好。
	 
### CPU 平台设置
目前 SDK 支持 armeabi、armeabi-v7a、arm64-v8a，为了防止在其他 CPU 平台上 SDK Crash，建议在您的 App 的 build.gradle 里加上 abiFilter，如下所示：

```
defaultConfig {
	ndk {
          //设置支持的 so 库框架
          abiFilters 'armeabi-v7a', 'armeabi', 'arm64-v8a'
     }
}
```

## 配置流程
### 接入配置
- 云 OCR SDK（WbCloudOcr）最低支持到 Android API 14: Android 4.0（ICS），请在构建项目时注意。
- 云 OCR SDK 将以 AAR 文件的形式提供。另外 OCR SDK 同时需要依赖云公共组件 WBCloudNormal，同样也是以 AAR 文件的形式提供。
- 需要添加下面文档中所示的依赖（将提供的 AAR 文件加入到 App 工程的 libs 文件夹下面，并且在 build.gradle 中添加如下配置）：

```
android{
     //...
     repositories {
        flatDir {
            dirs 'libs' //this way we can find the .aar file in libs folder
        }
    }
}
//添加依赖
dependencies {
     //0. appcompat-v4
 compile 'com.android.support:appcompat-v4:23.1.1'
    //1. 云OCR SDK
      compile(name: 'WbCloudOcrSdk', ext: 'aar')
  //2.云公共组件
        compile(name: 'WbCloudNormal', ext: 'aar')    }

```

### 混淆配置
云 OCR 产品的混淆规则分为两部分，分别是云 OCR SDK 的混淆规则（v2.7.X之后的版WbCloudOcrSdk已经内置了混淆规则，接入方可以忽略不加载），云公共组件的混淆规则（接入方必须加载）。
- 云 OCR SDK 的混淆规则
```
######################云 ocr 混淆规则 ocr-BEGIN###########################
-keepattributes InnerClasses
-keep public class com.tencent.cloud.huiyansdkocr.net.*$*{
    *;
}
-keep public class com.tencent.cloud.huiyansdkocr.net.*{
    *;
}
-keep public class com.tencent.fujikoli.recdetectdemo.Jni.ScanRecNative{*;}

#######################云 ocr 混淆规则 ocr-END#############################
```
您可以将如上代码拷贝到您的混淆文件中。
- 云公共组件的混淆规则
```
#######################normal混淆规则-BEGIN#############################
#不混淆内部类
-keepattributes InnerClasses
-keepattributes *Annotation*
-keepattributes Signature
-keepattributes Exceptions

-keep public class com.tencent.cloud.huiyansdkface.normal.net.*$*{
    *;
}
-keep public class com.tencent.cloud.huiyansdkface.normal.net.*{
    *;
}
-keep public class com.tencent.cloud.huiyansdkface.normal.thread.*{
   *;
}
-keep public class com.tencent.cloud.huiyansdkface.normal.tools.*{
   *;
}
-keep public class com.tencent.cloud.huiyansdkface.normal.thread.*$*{
   *;
}

-keep public class com.tencent.cloud.huiyansdkface.normal.tools.secure.*{
   *;
}

-keep public class com.tencent.cloud.huiyansdkface.normal.tools.WLogger{
    *;
}
-keep class com.tencent.bugly.idasc.**{
    *;
}
#wehttp混淆规则
-dontwarn com.tencent.cloud.huiyansdkface.okio.**
-keep class com.tencent.cloud.huiyansdkface.okio.**{
    *;
}

# 里面 有 Java8的一些类 如 Duration
-dontwarn com.tencent.cloud.huiyansdkface.okhttp3.OkHttpClient$Builder
-keep class com.tencent.cloud.huiyansdkface.wehttp2.**{
    public <methods>;
}

-keep class com.tencent.cloud.huiyansdkface.okhttp3.**{
    public <methods>;
}

-keep interface com.tencent.cloud.huiyansdkface.wehttp2.**{
    public <methods>;
}
-keep public class com.tencent.cloud.huiyansdkface.wehttp2.WeLog$Level{
    *;
}
-keep class com.tencent.cloud.huiyansdkface.wejson.*{
    *;
}

-keep public class com.tencent.cloud.huiyansdkface.wehttp2.WeReq$ErrType{
    *;
}

-keep class * extends com.tencent.cloud.huiyansdkface.wehttp2.WeReq$WeCallback{
   public <methods>;
}

-keep class com.tencent.cloud.huiyansdkface.okio.**{
    *;
}
#######################normal混淆规则-END#############################
```
您可以将如上代码拷贝到您的混淆文件中，也可以将 SDK 中的`kyc-cloud-normal-proguard-rules.pro`拷贝到主工程根目录下，然后通过`-include kyc-cloud-normal-proguard-rules.pro`加入到您的混淆文件中。 

## 接口调用
SDK 接口提供的是有 UI 模式：SDK 对接口进行封装并且实现了识别页面，合作方只需要调用接口，即可以快速拉起 SDK，接收结果回调。接入简单，适合快速接入。
### SDK 接口调用方法
SDK 代码调用的入口为 `com.tencent.cloud.huiyansdkocr.WbCloudOcrSDK` 这个类。
```
public class WbCloudOcrSDK{

/**
* 该类为一个单例，需要先获得单例对象再进行后续操作
   */
       public static synchronized WbCloudOcrSDK getInstance() {
       //    ...
       }
 /**
  * 初始化云Ocr SDK，完成登录
  * @param context        拉起SDK的context
  * @param wbocrtypemode  识别证件的模式，参数是枚举类型
  * @param data           第三方需要传入的参数
  * @param loginListener  登录SDK回调
  */
public void init(Context context,WBOCRTYPEMODE wbocrtypemode,Bundle data,OcrLoginListener loginListerner){
//    ...
}
   /**
     * 登录成功后，调用此函数拉起sdk页面
     * @param context                  拉起SDK的上下文
     * @param idCardScanResultListener 返回到第三方的接口
     */
    Public void startActivityForOcr(Context context,IDCardScanResultListener){
    // ...
 }
/**
  * 登录回调接口
  */
public interface OcrLoginListener {
        void onLoginSuccess();
        void onLoginFailed(String errorCode, String errorMsg);
    }

/**
  * 退出SDK,返回第三方的回调,同时返回ocr识别结果
  */
public interface IDCardScanResultListener{
        /**
         * @RARAM exidCardResult   SDK返回的识别结果的错误码  
* @RARAM exidCardResult   SDK返回的识别结果的错误信息    
         */
        void onFinish(String errorCode, String errorMsg);
}

```
 **NONCE 类型的 ticket，其有效期为120秒，且一次性有效，即每次启动 SDK 刷脸都要重新请求 NONCE ticket，重新算 sign。同时建议合作方做前端保护，防止用户连续点击，短时间内频繁启动 SDK。**

WbCloudOcrSdk.init() 的第二个参数用来传递数据.可以将参数打包到 data(Bundle) 中，必须传递的参数包括：
```
//这些都是WbCloudOcrSdk.InputData对象里的字段，是需要传入的数据信息
String orderNo;  //每次OCR识别请求的唯一订单号: 建议为32位字符串(不超过32位)
String openApiAppId;  //APP_ID 
String openApiAppVersion;  //openapi  Version
String openApiNonce; //32位随机字符串
String openApiUserId; //user id
String openApiSign; //签名信息

```
以上参数被封装在 WbCloudOcrSdk.InputData 对象中（它是一个 Serializable 对象）。

WbCloudOcrSdk.startActivityForOcr() 的第三个参数 type 是个枚举类 WBOCRTYPEMODE，决定第三方登录成功后以哪种模式进行身份证识别。
```
/**
  * 调OCR 的模式
  */
public enum WBOCRTYPEMODE {
WBOCRSDKTypeFrontSide,//人像面模式，直接进入身份证人像面识别
WBOCRSDKTypeBackSide,//国徽面模式，直接进入身份证国徽面识别
WBOCRSDKTypeContinus,//连续识别模式，识别完身份证人像面后，接着识别身份证国徽面，而不是退出相机页面
}

```
登录接口：
```
/**
  * 登录回调接口
  */
public interface OcrLoginListener {
        void onLoginSuccess();//登录成功
          /**
           * @PARAM errorCode 登录失败错误码
           * @PARAM errorMsg  登录失败错误信息
           */        
         void onLoginFailed(String errorCode, String errorMsg);
    }

```
返回第三方接口：
```
/**
  * 退出SDK,返回第三方的回调,同时返回ocr识别结果
  */
public interface IDCardScanResultListener{
        /**
         * 退出SDK,返回第三方的回调,同时返回ocr识别结果
         * @param errorCode        返回错误码，识别成功返回 0
         * @param errorMsg        返回错误信息，和错误码相关联         */
        void onFinish(String errorCode, String errorMsg);
}

```

#### 身份证识别结果类
身份证识别结果, 封装在 EXIDCardResult 类中，通过 WbCloudOcrSDK.getInstance().getResultReturn() 获得，该类属性如下所示：

```
public int type;//拉起SDK的模式所对应的int 值，也就是 startActivityForOcr 方法中 WBOCRTYPEMODE type 的枚举值 value
    // 识别人像面返回的信息
    public String cardNum;  //身份证号码
    public String name;//姓名
    public String sex;//性别
    public String address;//住址
    public String nation;//民族
    public String birth;//出生年月日
    public String frontFullImageSrc;// 人像面图片在本地的存放路径
	public String frontWarning;//人像面告警码

    //识别国徽面返回的信息
    public String office;//签发机关
    public String validDate;//有效期限
    public String backFullImageSrc;//国徽面图片在本地的存放路径
	public String backWarning;//国徽面告警码
	public String sign;//签名
	public  String orderNo; //每次OCR识别请求的唯一订单号: 建议为32位字符串(不超过32位)
	public String ocrId;//识别的唯一标识
	
	//新增的返回字段
	public String frontMultiWarning;//人像面多重告警码
	public String backMultiWarning;//国徽面多重告警码
	public String frontClarity;//人像面清晰度得分
	public String backClarity;//国徽面清晰度得分
	public String frontCrop; //身份证正面切边照 base64
	public String backCrop; //身份证反面切边照 base64

```
#### 接口参数说明
 **NONCE 类型的 ticket，其有效期为120秒，且一次性有效，即每次启动 SDK 刷脸都要重新请求 NONCE ticket，重新算 sign。同时建议合作方做前端保护，防止用户连续点击，短时间内频繁启动 SDK。**
InputData 是用来给 SDK 传递一些必须参数所需要使用的对象（WbCloudOcrSdk.init() 的第二个参数），合作方需要往里塞入 SDK 需要的一些数据以便启动 OCR SDK。
其中 InputData 对象中的各个参数定义如下表，请合作方按下表标准传入对应的数据。

|参数 | 说明 | 类型 | 长度 | 是否必填|
|------| ------ | ------| ----- | ----------|
|orderNo | 每次 OCR 识别请求的唯一订单号: 建议为32位字符串(不超过32位) | String | 32 | 是 |
|openApiAppId | 业务流程唯一标识，即 wbappid，可参考 获取 WBappid 指引在人脸核身控制台内申请 | String | 8 | 是|
|openApiAppVersion | 接口版本号，默认填 1.0.0 | String | 20 | 是 |
|openApiNonce | 与服务端生成签名的随机数保持一致 | String | 32 | 是|
|openApiUserId | User Id，每个用户唯一的标识 | String | 30 | 是|
|openApiSign | 合作方后台服务器通过 ticket 计算出来的签名信息 | String | 40 | 是|

### 个性化参数设置
WbCloudOcrSdk.init() 里 Bundle data，除了必须要传的 InputData 对象之外，还可以由合作方传入一些个性化参数，量身打造更契合自己 App 的 SDK。如果合作方未设置这些参数，则以下所有参数按默认值设置。

#### 设置 SDK 的扫描识别的时间上限
合作方可以设置 SDK 的扫描识别时间的上限。SDK 打开照相机进行扫描识别的时间上限默认是20秒，20秒内若扫描识别成功则返回到 SDK 的第一个界面，否则直到20秒直接退出扫描界面。第三方可对其个性化设置，设置的时间上限不能超过60秒，建议第三方采用默认值，不要修改这个参数。设置代码如下：
```
# 在 MainActivity 中单击某个按钮的代码逻辑：
  //先将必填的 InputData 放入 Bundle 中
  data.putSerializable(WbCloudFaceVerifySdk.INPUT_DATA, inputData);
  //设置 SDK 扫描识别身份证的时间上限，如果不设置则默认 20 秒；设置了则以设置为准
  //此处设置 SDK 的扫描识别时间的上限为 20 秒
   data.putLong(WbCloudOcrSDK.SCAN_TIME, 20000);

```

## OCR Android 错误码
### 终端返回|错误码

|错误码 | 错误描述|
|- | -|
|IDOCR_LOGIN_PARAMETER_ERROR = "-20000"; | 传入参数有误|
|IDOCR_ERROR_CANCELED_AUTH="200100"; | 用户授权时取消|
|IDOCR_USER_CANCEL="200101"; |	用户取消操作|
|IDOCR_RECOGNISE_TIME_OUT="200102";|识别超时|
|IDOCR__ERROR_USER_NO_NET="100101"; | 无网络|
|IDOCR_USER_2G="100102"; | 不支持 2G 网络|
|IDOCR_ERROR_PERMISSION_CAMERA="100103"; | 无相机权限|
|IDOCR_ERROR_PERMISSION="100105"; | 权限异常|
|IDOCR_LOGIN__ERROR="-10000"; | 登录错误|
|SERVER_FAIL="-30000"; | 内部服务错误|
|DECODE_FAIL="300101"|解码 emg 异常（包括 emg 为空）|


### 后台返回错误码
错误码 | 错误描述
- | -
INTERNAL_SERVER_ERROR="999999" | 网络不给力，请稍后再试
FRONT_INTERNAL_SERVER_ERROR="999998" | 网络不给力，请稍后再试
SERVICE_TIME_OUT="999997" | 网络不给力，请稍后再试
OAUTH_INVALID_REQUEST="400101" | 不合法请求
OAUTH_INVALID_LOGIN_STATUS="400102" | 不合法请求
OAUTH_ACCESS_DENIED="400103" | 服务器拒绝访问此接口
OAUTH_INVALID_PRIVILEGE="400104" | 无权限访问此请求
OAUTH_REQUEST_VALIDATE_ERROR="400105" | 身份验证不通过
OAUTH_TPS_EXCEED_LIMIT="400501" | 请求超过最大限制
OAUTH_INVALID_VERSION="400502" | 请求上送版本参数错误
OAUTH_INVALID_FILE_HASH="400503" | 文件校验值错误
OAUTH_REQUEST_RATE_LIMIT="400504" | 请求访问频率过高


## 接入示例
```
# 在 MainActivity 中单击某个按钮的代码逻辑：
//先填好数据 
  Bundle data = new Bundle();
        WbCloudOcrSDK.InputData inputData = new WbCloudOcrSDK.InputData(
                orderNo,
                appId,
                openApiAppVersion,
                nonce,
                userId,
                sign);
        data.putSerializable(WbCloudOcrSDK.INPUT_DATA, inputData);
		//个性化参数设置，可以不设置，不设置则为默认选项。
//设置扫描识别的时间上限,默认 20 秒。用户有效设置范围是（0-60000）
  data.putLong(WbCloudOcrSDK.SCAN_TIME, 20000);
//初始化 SDK，得到是否登录 SDK 成功的结果 
        WbCloudOcrSDK.getInstance().init(MainActivity.this,WbCloudOcrSDK.WBOCRTYPEMODE.WBOCRSDKTypeContinus,data, new WbCloudOcrSDK.OcrLoginListener() data, new WbCloudOcrSDK.OcrLoginListener() {
            @Override
            public void onLoginSuccess() {  //登录成功,拉起 SDL 页面                              WbCloudOcrSDK.getInstance().startActivityForOcr(MainActivity.this,
      new  WbCloudOcrSDK.IDCardScanResultListener() {  //返退出 SDK 回调接口
                    @Override
                    public void onFinish(String resultCode, String resultMsg) {
										//身份证识别结果类
EXIDCardResult result=WbCloudOcrSDK.getInstance().getResultReturn();

           // resultCode为0，则识别成功；否则识别失败
               if (result.frontFullImageSrc!=null||result.backFullImageSrc!=null) {
							 //  识别结果类的正面图片或者反面图片有不为空 
						WLogger.d(TAG, "识别成功,识别身份证的结果是:"+WbCloudOcrSDK.getInstance().getResultReturn().toString());
               }else{  //  TODO:2017/10/30
               WLogger.d(TAG, "识别失败:"+resultCode+"--"+resultMsg);
               }
            }
});
}              
@Override
public void onLoginFailed(String errorCode, String errorMsg) {
if(errorCode.equals(ErrorCode.IDOCR_LOGIN_PARAMETER_ERROR)) {
Toast.makeText(MainActivity.this, "传入参数有误！" + errorMsg, Toast.LENGTH_SHORT).show();
} else {
Toast.makeText(MainActivity.this, "登录 OCR SDK 失败！" + "errorCode= " + errorCode + " ;errorMsg=" + errorMsg, Toast.LENGTH_SHORT).show();
}
}
})
```
