# Yii2微信扫码支付
适用于微信扫码支付接口`模式二`，具体使用方法如下：
## 安装
   ```
## 配置使用

### 一、配置Yii2组件
在common下的全局配置文件main.php中添加组件配置，请参考如下alipay的配置：
```php
'components' => [
        'wxpay' => [
            'class' => 'iqianfang\yii2wxpay\WxPay',
        ],
    ],
```
注意：一些固定配置/不常用配置放到组件类的init里了，有用到的可以去那里改。
### 二、微信支付配置
#### 1、去微信商户后台下载证书文件，并覆盖cert里的同名文件
#### 2、配置微信支付信息
在lib/WxPay.Config.php 中配置，具体详见该文件
### 三、逻辑处理
#### 1、支付请求提交
在需要发起支付请求的地方调用
```php
Yii::$app->wxpay->submit($order);
```
其中order是你的订单实例，通过它可以获取到`订单号`、`支付总价`等，如果你的逻辑用不到或者与此不同，请自行修改
#### 2、支付结果通知
回调地址的配置在WxPay.php 的`getQrcode`方法中的`SetNotify_url`，具体请根据自己的实际情况配置。
根据设置的notify_url编写控制器及action代码。以我的配置为例：
```php
class PaymentController extends Controller
{
    /**
     * 关闭csrf，以便异步通知可访问
     * @param \yii\base\Action $action
     * @return bool
     * @author WangTao <78861190@qq.com>
     */
    public function beforeAction($action)
    {
        $this->enableCsrfValidation = false;
        return parent::beforeAction($action); // TODO: Change the autogenerated stub
    }
    
    /**
     * 响应微信支付异步通知
     * @author WangTao <78861190@qq.com>
     */
    public function actionWxpayNotify()
    {
        $data = Yii::$app->wxpay->checkSign();
        $result = Yii::$app->wxpay->getResult();
        if ($data['return_code'] == 'SUCCESS' && $result['result_code'] == 'SUCCESS') {
            $order = Order::find()->where(['sn'=>$result['out_trade_no']])->one();
            $order->order_status = 1;
            $order->save();
        }

        Yii::$app->wxpay->replyNotify();
    }
```
## 其他
`submit`会返回一个二维码，该二维码即为扫码支付用的，具体的html代码在`getQrcode`中。如果你要调整二维码的大小，或者更换其他第三方生成二维码接口自行修改即可

别忘了配置urlManager，开启url美化，否则通知url会有参数，通知会失败