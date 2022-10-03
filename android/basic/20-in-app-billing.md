# In App Billing

## 内购 API

- 使用设备上安装的 Google Play 应用的暴露的 API，Google Play 应用负责在 Google Play Service 和你的应用之间，使用 IPC 进行通信
- 必须通过 Google Play 发布 Google Play 应用必须可以访问 Google Play 服务器

### Version 3 功能

- 高效 API 请求
- 同步通知
- 所有商品可以被跟踪，用户不能拥有多个同样的产品，一个产品同时只能拥有一个
- 购买的产品可以被消耗，消耗后，产品恢复到‘未拥有’状态，用户可以再次购买
- 提供订阅功能

### 内购产品

- 产品是你应用内的数字产品
- 只可以销售数字内容，不能销售物理产品，个人服务，或其他需要物理传输的物品
- 内购产品没有退款
- Google Play 不提供任何形式的内同传输，你有义务将内购的数字内容传输给用户
- 内购产品通常只显式关联于一个 app一个应用不可以销售其他发布应用中的内购产品

### 产品类型

- 通过开发者控制台定义产品类型
- 可以指定两种产品类型
  - 可管
  理的内购产品
  - 订阅
- Google Play 会基于每一个用户，处理跟踪你应用内的两种产品类型的所有权

### 开发者控制台

- 发布内购应用，管理内购产品等
- 创建一个产品列表，关联到应用可以定义如下信息：
  - 唯一产品ID，SKU
  - 产品类型
  - 描述
  - Google Play 处理跟踪产品
- 可以添加价格模板，集中管理产品价格。使用价格模板的时候，可以包含本地税款，或者可以提供一个包含的税款的价格
- 可以修改价格模板大众的价格，例如刷新汇率，会设置用到链接到价格模板的 ap

### Google Play 购买流程

- 统一的使用体验
- 必须有 Google 付款商账户，才能使用内购服务
- app 发送一个内购产品的订单请求，Google Play 处理所有包括请求、证实付款表单交易的细节，并处理金融交易
- 处理完成，Google Play 发送给 app 购买详情，例如订单编号、时间、付款价格

## 示例程序

SDK 中提供了一个示例程序。TrivialDrive for the Version 3 API 展示了使用 Version 3 的内购 API 来实现一个赛车游戏的内购。包括如何发送请求，处理 Google Play 的同步响应，记录产品票据。例子中列举了很多方便处理内购操作和自动验证签名的类

### 低版本迁移建议

内购 Version 2 API 于 2015 年停止维护。必须使用 Version 3。

- 之前在控制台中定义的，已经托管的产品和订阅，会和之前一样。
- 没有托管的产品和订阅，会认为是已经托管的。前提是使用 Version 3 进行购买请求。不需要创建一个新的产品。

## API 说明

### 商品类型

- 通过 Google Play Developer Console 可以定义商品类型，SKU，价格，描述等。
- Verison 3 API 支持托管的内购商品和订阅

### 托管的内购产品

- 指的是 Google Play 管理和追踪所有权信息的商品。当用户购买一个托管的内购商品，Google Play 会基于每个用户存储每个商品的信息。这样可以让你在任何时候通过查询 Google Play 来恢复一个特定用户的购买状态。该信息会一直保存在 Google Play 服务器，即使用户卸载了应用或者更换了设备

- 如果你使用的是 Version 3 API，还可以消耗掉托管的内购商品。通常情况用于可以多次购买的商品。用户购买后，托管的商品不能被再次购买，直到你发送消耗请求给 Google Play，消耗了这个商品

### 订阅

- 可以让你周期性的出售内容、服务、或者功能
- 订阅不可以被消耗掉

## 购买商品流程

- 你的应用想 Google Play 发送一个`isBillingSupported`的请求，来确认目标的内购 API 版本是否支持
- 当你的应用启动或者用户登入时，向 Google Play 检查，商品是否属于用户。通过发送`getPurchases`请求。如果请求成功，Google Play 会返回一个`Bundle`，包含了一个购买商品的列表、一个各商品购买详情的列表、一个购买签名的列表
- 通常，将需可以购买的商品通知用户，查询商品在 Google Play 定义的详细信息，通过发送`getSkuDetails`请求。请求时必须指定一个商品ID列表，如果成功，Google Play 会返回一个`Bundle`，包含了商品的详细信息
- 如果一个商品不属于该用户，此时你可以初始化该商品的购买。开始一个购买请求，通过发送`getBuyIntent`请求，指定要购买的商品 ID。在开发者控制台中创建新的内购商品是，你应该记录下商品 ID
  - Google Play 返回一个`Bundle`，包含了一个`PendingIntent`，该 PendingIntent 会启动购买的检查 UI
  - 你的应用启动该 PendingIntent，通过调用`startIntentSenderForResult`
  - 当检查流程完成，Google Play 发送一个响应`Intent`，回调你的`onActivityResult`。result code 标识了购买是否成功。响应 Intent 包含了商品的信息，并且有一个 Google Play 生成的`purchaseToken`唯一字符串，表标识本次交易。Intent 也包含了购买的签名，通过私有的开发者 key 签的

## 内购商品

- 使用消耗机制来跟踪用户的内购商品所有权
- Version 3 中，所有内购商品都是托管的。Google Play 维护内购商品购买的用户所有权，你的程序可以在需要的时候进行查询。当一个商品被购买后，被认为是已经拥有，此状态的内购商品不可以再次购买。必须发送一个消耗请求，Google Play 才能将他重新变为可购买商品。消耗内购商品会改变商品状态为未拥有，并且删除之前的购买数据。
- 获取用户拥有的商品列表，通过调用`getPurchases`
- 进行一个消耗请求，通过`consumePurchase`，请求参数中，必须加上 Google Play 为此次交易生成的内购商品的唯一 `purchaseToken`字符串。Google Play 会返回一个状态码指明结果

### 非消耗型和消耗型内购商品

- 非消耗型商品。通常，对于只购买一次就能永久使用的商品，不应该设置为消耗型商品。当用户购买后，商品会永久的和用户的 Google 账号绑定
- 消耗型商品。通常，对于可以多次购买的商品，设置为消耗型商品。这类商品通常提供的是临时效果。在应用中分配所购买的商品效果或收益，称为内购商品的'配置'。你有责任控制并跟踪内购商品的配置
- 在配置可消耗商品之前，必须发送消耗请求给 Google Play 并且接受到成功的响应

### 管理消耗购买流程

- 启动购买流程`getBuyIntent`
- 获取响应 Bundle
- 如果购买成功，消耗商品，通过`comsumePurchase`
- 获取响应吗，确认是否消耗成功
- 如果消耗成功，进行内购商品配置

- 另外，当用户启动或者登录应用，应该检查用户是否拥有未消耗的内购商品。如果有，确保进行消耗并配置
- 推荐的程序启动流程
  - 发送`getPurchases`请求，查询是否持有内购商品
  - 如果有任何消耗型商品，进行消耗
  - 获取响应码
  - 如果消耗成功，进行配置

## 本地缓存

因为 Google Play 客户端缓存了内购信息，因此可以更频繁的查询

## 代码

### 添加 AIDL 文件

- SDK Manager 安装 Google Play Billing Library
- IInAppBillingService.aidl 文件定义了内购服务的接口。通过 IPC 方法调用来进行购买请求。文件位置为 `<sdk>`/extras/google/play_biling/
- src/main/ 下添加 aidl 目录
- aidl 目录下添加包 com.android.vending.billing
- 将文件拷贝进去
- 构建项目，在 /gen 目录下可以看到 IInAppBillingService.java

### 更新 Manifest

`<uses-permission android:name="com.android.vending.BILLING"/>`

### 创建 ServiceConnection

- onServiceConnected 获取 service 引用
- onCreate 中绑定 service
- 使用显式 Intent，通过设置包进行。包名是 Google Play 的包名
- onDestroy 中解绑

```java
    private IInAppBillingService mService;
    private ServiceConnection mServiceConn = new ServiceConnection(){

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mService = IInAppBillingService.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mService = null;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Intent serviceIntent = new Intent("com.android.vending.billing.InAppBillingService.BIND");
        serviceIntent.setPackage("com.android.vending");
        bindService(serviceIntent, mServiceConn, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mService != null) {
            unbindService(mServiceConn);
        }
    }
```

### 发送购买请求（IPC 方法）给 Google Play 应用

#### 查询商品是否可以购买

- 传递请求给内购 service，创建一个`Bundle`，包含商品 ID 的 ArrayList，key 为 "ITEM_ID_LIST"，每一个字符串是一个产品 ID。

```java
ArrayList<String> skuList = new ArrayList<String> ();
skuList.add("premiumUpgrade");
skuList.add("gas");
Bundle querySkus = new Bundle();
querySkus.putStringArrayList("ITEM_ID_LIST", skuList);
```

- 调用`getSkuDetail`，从 Google Play 获取返信息。

```java
Bundle skuDetails = mService.getSkuDetails(3,
   getPackageName(), "inapp", querySkus);
```

- 如果请求成功，返回的`Bundle`包含一个响应 code `BILLING_RESPONSE_RESULT_OK`
- `getSkuDetails`不能在主线程调用，此方法会触发网络请求。
- 所有请求返回 code，见[文档](https://developer.android.com/google/play/billing/billing_reference.html)
- 请求结果存储在 ArrayList 中，key 为`DETAILS_LIST`，购买信息以 JSON 格式存储在字符串中。

```java
int response = skuDetails.getInt("RESPONSE_CODE");
if (response == 0) {
   ArrayList<String> responseList
      = skuDetails.getStringArrayList("DETAILS_LIST");

   for (String thisResponse : responseList) {
      JSONObject object = new JSONObject(thisResponse);
      String sku = object.getString("productId");
      String price = object.getString("price");
      if (sku.equals("premiumUpgrade")) mPremiumUpgradePrice = price;
      else if (sku.equals("gas")) mGasPrice = price;
   }
}
```

#### 购买一个商品

- 开始购买请求，调用`getBuyIntent`，传递商品 ID，商品类型（"inapp" / "subs"），和一个`developerPayload`字符串。该字符串用于 Google Play 发送回商品信息的时候，作为额外参数

```java
Bundle buyIntentBundle = mService.getBuyIntent(3, getPackageName(),
   sku, "inapp", "bGoa+V7g/yqDXvKRqq+JTFn4uQZbPiQJo4pf9RzJ");
```

- 如果请求成功，会返回一个包含 `BILLING_RESPONSE_RESULT_OK`返回码的 Bundle 和一个用于开始购买流程的`PendingIntent`。
- 下一步是从响应`Bundle`中根据 key`BUY_INTENT`提取`PendingIntent`
- 使用`PendingIntent`调用`startIntentSenderForResult`来完成交易。

```java
startIntentSenderForResult(pendingIntent.getIntentSender(),
   1001, new Intent(), Integer.valueOf(0), Integer.valueOf(0),
   Integer.valueOf(0));
```

- Google Play 在`onActivityResult`方法中发送请求响应。订单的商品数据在以 `INAPP_PURCHASE_DATA`为 key 的字符串中

```java
'{
   "orderId":"GPA.1234-5678-9012-34567",
   "packageName":"com.example.app",
   "productId":"exampleSku",
   "purchaseTime":1345678900000,
   "purchaseState":0,
   "developerPayload":"bGoa+V7g/yqDXvKRqq+JTFn4uQZbPiQJo4pf9RzJ",
   "purchaseToken":"opaque-token-up-to-1000-characters"
 }'
```

- Google Play 为商品生成一个 token，其他的方法需要传递这个 token。你必须存储和返回整个 token

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
   if (requestCode == 1001) {
      int responseCode = data.getIntExtra("RESPONSE_CODE", 0);
      String purchaseData = data.getStringExtra("INAPP_PURCHASE_DATA");
      String dataSignature = data.getStringExtra("INAPP_DATA_SIGNATURE");

      if (resultCode == RESULT_OK) {
         try {
            JSONObject jo = new JSONObject(purchaseData);
            String sku = jo.getString("productId");
            alert("You have bought the " + sku + ". Excellent choice,
               adventurer!");
          }
          catch (JSONException e) {
             alert("Failed to parse purchase data.");
             e.printStackTrace();
          }
      }
   }
}
```

- 当你发送购买请求时，创建一个唯一标识该请求的字符串，将该 token 包含在 developerPayload 中，你可以使用一个随机生成的字符串作为 token，当从 Google Play 接受到响应时，检查 `orderId`和`developerPayload`。更安全的措施是，将检查放在安全的服务器中。校验`orderId`是一个唯一的之前没有处理的，`developerPayload`和之前发送的一致

#### 查询已购商品

- 调用`getPurchases`方法，获取用户的已购信息

```java
Bundle ownedItems = mService.getPurchases(3, getPackageName(), "inapp", null);
```

- Google Play 服务器只会返回当前登录用户的购买。如果请求成功，返回 Bundle 会包含 0 响应码。也会包含一个商品 ID 的集合。
- 性能考虑，只会返回700个产品，当第一次调用`getPurchases`。如果用户拥有大量的商品，Google Play 返回 Bundle 会包含一个字符串 token，映射 key 为 INAPP_CONTINUATION_TOKEN，标识了更多商品可以获取。随后可以继续调用该方法，并且传递 token 作为参数。Google Play 会继续返回一个 token 直到所有商品发送。

```java
int response = ownedItems.getInt("RESPONSE_CODE");
if (response == 0) {
   ArrayList<String> ownedSkus =
      ownedItems.getStringArrayList("INAPP_PURCHASE_ITEM_LIST");
   ArrayList<String>  purchaseDataList =
      ownedItems.getStringArrayList("INAPP_PURCHASE_DATA_LIST");
   ArrayList<String>  signatureList =
      ownedItems.getStringArrayList("INAPP_DATA_SIGNATURE_LIST");
   String continuationToken =
      ownedItems.getString("INAPP_CONTINUATION_TOKEN");

   for (int i = 0; i < purchaseDataList.size(); ++i) {
      String purchaseData = purchaseDataList.get(i);
      String signature = signatureList.get(i);
      String sku = ownedSkus.get(i);

      // do something with this purchase information
      // e.g. display the updated list of products owned by user
   }

   // if continuationToken != null, call getPurchases again
   // and pass in the token to retrieve more items
}
```

#### 消耗商品

- 托管商品可以消耗，订阅不可以。
- 记录一个商品消耗，发送`consumePurchase`，传递`purchaseToken`，该 token 是 Google Play 服务对购买请求返回值的 `INAPP_PURCHASE_DATA`的一部分
- 不可在主线程调用

```java
int response = mService.consumePurchase(3, getPackageName(), token);
```

#### 订阅-

- 商品类型需设置为`subs`

```java
Bundle bundle = mService.getBuyIntent(3, "com.example.myapp",
   MY_SKU, "subs", developerPayload);

PendingIntent pendingIntent = bundle.getParcelable(RESPONSE_BUY_INTENT);
if (bundle.getInt(RESPONSE_CODE) == BILLING_RESPONSE_RESULT_OK) {
   // Start purchase flow (this brings up the Google Play UI).
   // Result will be delivered through onActivityResult().
   startIntentSenderForResult(pendingIntent, RC_BUY, new Intent(),
       Integer.valueOf(0), Integer.valueOf(0), Integer.valueOf(0));
}
```

- 查询是否有活动的订阅
- 返回 Bundle 包含了用户所有激活的订阅。如果过期，不会在返回值中

```java
Bundle activeSubs = mService.getPurchases(3, "com.example.myapp",
                   "subs", continueToken);
```

### 最佳安全最佳实践

- Google Play 会对 JSON 字符串使用 private key 签名，该 key 是关联了控制台的应用中出创建的签名。控制台为每个应用创建 RSA key。
- Google Play 生成的 public key 以 Base64 编码二进制编码，X.509 subjectPublicKeyInfo DER SEQUENCE 格式。
- 建议验证签名放在服务器。
- 在服务端执行签名认证，同时确保服务器和客户端的通信安全
- 解锁内容不要打包在 apk 中。内容可以保存在设备内存或者 sd 卡中。如果保存在 sd 卡，确保使用设备特定的 key 进行加密
- 混淆代码
  - 将方法嵌入其他方法中
  - 使用动态字符串
  - 使用反射调用方法
  - `-keep class com.android.vending.billing.**`
- 不直接使用示例代码
- 使用安全的随机串，`SecureRandom`，或者服务器生成随机串
- 设置 payload 字符串。用于标识是哪个用户进行购买。确保响应数据中的相应字段一致。该确认应放于服务端
- 当你发现内容被非法使用，可以发起'商标侵权通知'或'版权侵权通知'
- 实现可撤销性的解锁内容
- 保护 Google Play 的 public key。在运行时构造从多个片段这个，

### [API 文档](https://developer.android.com/google/play/billing/billing_reference.html)

## 内购管理

### 需要进行管理的任务

- 设置并维护商品列表
- 注册测试账号
- 处理退款
- 注册测试账号，需要有发布账号
- 创建商品列表，进行退款需要拥有商家账号

### 创建商品列表

- 每个发布的 app 包含一个商品列表
- 访问 app 商品列表，通过打开“内购商品”页。链接只有在拥有商家账号，并且 manifest 包含了 com.android.vending.BILLING 的权限
- 商品列表包含了内购商品，订阅，或者组合
- 商品列表只存储商品元数据，不存储任何数字内容
- 通常一个 app package 只有一个商品列表。如果创建一个 app 的商品列表时使用多应用功能来发布多个 apk，那么商品列表会应用于所有关联的 apk。在这个模式中不能为单独的 apk 创建商品列表。
- 商品列表添加商品有两种方式
  - 在“内购商品”页添加商品
  - 通过 csv 添加批量添加，不支持订阅
- 商品添加后，不能改变类型。订阅的价格在发布后不能更改
- 商品 ID 在一个 app 的命名空间下是唯一的。必须以小写或数字开头，创建后不可更改，不可重用
- 商品的发布状态有 激活 和 未激活 两个状态。只有激活并且 app 发布后，商品才能被用户看见
- 标题是用户看见的，最好不能超过 25 个字符，最多能有 55 字符
- 描述不超过 80 字符
- 价格设置后，会自动根据汇率生成所有价格。可以手动更改货币
- 设置订阅时，必须定义周期
- 如果新的订阅购买周期是季节性的，必须指定开始日期和结束日期。订阅只在两日期间是激活的
- 选择添加价格优惠比例，可以让用户在季节开始时以优惠价格进行订阅，可以为一个订阅在不同的时间点设置多个折扣
- 订阅激活后，不能改变订阅周期。对于季节性的订阅，不能更改开始日期和结束日期
- 可以设置试用周期，至少 7 天
- 可以设置订阅的奖励时间

### 批量添加产品列表

- 不可添加订阅，不可关联价格模板
- 可以覆盖已经存在的商品
- 第一行设置语法，下一行设置商品具体信息
- 每一行可以包含的的值有
  - Product ID
  - Publish State 必须是 published 或者 unpublished
  - Purchase Type 必须是 managed\_by\_android，因为不支持订阅
  - Auto Translate 必须是 false，如果需要显式指定 Locale 值
  - 具体见文档

### 价格模板

见文档

### 处理退款

- 退款只能由开发者在后台手动发出。
- Google Play 会收到通知，然后会向app 发送消息。
- Google Play Voided Purchases API 可以用来撤销购买，用户撤销的时候

### 订单号

- Google Play 将其作为 orderId 字段传回给你，在 PURCHASE\_STATE\_CHANGED intent 中。测试时， orderId 是空的，使用 purchaseToken 替代
- 订单号可用于交易的通用标识，可用于跟踪交易，以及用于客户支持。
- 由 Google 进行分配和管理

### 支持

[google 网上论坛](http://groups.google.com/group/android-developers)
[stack over flow Tag android](http://stackoverflow.com/questions/tagged/android)

### 内购测试相关

- 测试购买，可以让测试用户真正购买发布的内购商品，而不需要任何真实的充值
- Google Play 静态响应，用于早期开发
- 设备上 apk 的 version code 必须和 Google Play 上的一致

### 测试内购商品

两种方式

- Test Purchase，只能使用在 alpha/beta 版，测试用户可以测试
- Real Purchase，让正常用户进行购买

两种方式都需要发布 alpha/beta 版，来进行测试人员的管理

#### Test Purchase（内购沙盒）

- 没有 orderId，确保没有真实的充值
- 订阅是每天的，会忽略设置的周期
- 通过开发者控制台上传发布商品，可以在发布 apk 之前
- 创建许可测试用户。控制台中 Settings > Account Details，在 License Testing 添加 Gmail 账户
- app 发布 alpha/beta 后，需要使用提供的内部 url 进行测试
- 测试用户的账号必须在设备上。设备如果有多个账号，购买会关联在下载 app 的账号上。如果没有一个账号下载 app，那么购买会关联第一个账号。用户可以通过展开购买对话框来确认进行购买账号
- 测试商品没有 orderId，跟踪该商品，请使用 purchaseToken
- 认证的许可测试账户是和开发者账号关联的，和 apk 或者包名没关系
- Google Play 会对测试的产品进行标记
- 手动取消购买，打开 Play Stroe 中的 app 页即可。如果是订阅，需要使用 cancel() API，测试订阅不能使用 refund 和 revoke

#### 真实交易测试

- 通过 alpha/beta 测试组，真实的用户可以进行测试。使用真实的充值

### 静态响应

- 请求的时候，使用特殊商品的预留 ID，每个预留的 ID 都会响应静态的信息
- 预留 ID 有4个
  - android.test.purchased:成功购买
  - android.test.canceled:购买取消
  - android.test.refund:退款
  - android.test.item_unavaliable

### 设置测试商品

在测试完静态响应，确认签名认证正常之后，可以使用真实的商品测试内购。你可以进行端到端的测试，通过 alpha 发布通道。不能使用开发账号

- 上传到 alpha 发布通道
- 添加商品
- 安装程序
- 确认版本
- 进行购买
