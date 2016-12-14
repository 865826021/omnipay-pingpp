# Omnipay: Pingpp

## Progress

开发中，预计全部完成时间，12月底

## Introduction

[Ping++](https://www.pingxx.com/api) 是国内领先的聚合支付服务商，集成了包括支付宝（APP、Wap、PC、即时到账、扫码、企业付款），微信（APP、公众号、红包），
银联网关、银联企业网银、Apple Pay、QQ 钱包、易宝支付、百度钱包、京东支付、京东白条、招行一网通、分期支付
等国内主流支付渠道。

[Omnipay](http://omnipay.thephpleague.com/) 是一个 PHP 支付处理库，拥有清晰一致的 API 标准和完善的单元测试，支持国内外多达数十个主流支付网关。

## Install

## Usage

### Preparation
```php
require './vendor/autoload.php';

use Omnipay\Omnipay;
use Omnipay\Pingpp\Common\Helpers;
use Omnipay\Pingpp\Common\Channels;

$skLiveKey = 'sk_test_iv5yr1HWLOqHjbjTq1KWLmD4';
$appId = 'app_9SSaPOaDuPCKvHSy';
$channel = Channels::ALIPAY_WAP;
```

## Initialize
```php
try {
    /**
     * @var $gateway \Omnipay\Pingpp\Gateway
     */
    $gateway = Omnipay::create('Pingpp');
    $gateway->initialize(array(
        'apiKey' => $skLiveKey,
        'privateKey' => file_get_contents(PINGPP_ASSET_DIR.'/sample_rsa_private_key.pem') // optional
    ));
} catch (\Exception $e) {
    echo $e->getMessage();
}
```

### Create Charge (创建 Charge)
```php
/**
 * @var \Omnipay\Pingpp\Message\PurchaseRequest $transaction
 */
$transaction = $gateway->purchase(array(
    'appId' => $appId,
    'transactionId' => Helpers::generateTransactionId(),
    'channel' => $channel,
    'channelExtraFields' => array( // optional
        'app_pay' => true
    ),
    'subject' => 'Demo subject',
    'body' => 'Demo body',
    'description' => 'Demo description', // optional
    'amount' => 0.01,
    'currency' => 'cny',
    'clientIp' => '127.0.0.1',
    'timeExpire' => time() + 3600, // optional
    'metadata' => array('foo' => 'bar'), // optional
    'returnUrl' => 'http://yourdomain.com/path/to/awesome/return.php', // optional
    'cancelUrl' => 'http://yourdomain.com/path/to/awesome/cancel.php', // optional
    'notifyUrl' => 'http://yourdomain.com/path/to/awesome/notify.php', // optional
));

/**
 * 以下 $response 的方法支持同上
 * @var \Omnipay\Pingpp\Message\Response $response
 */
$response = $transaction->send();
if ($response->isSuccessful()) {
    $reference_id = $response->getTransactionReference();
    echo "Transaction reference = " . $reference_id .PHP_EOL;
    echo json_encode($response->getData());die;
} else {
    echo $response->getMessage();
}
```

### Fetch Charge (查询单笔 Charge)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchTransactionRequest $transaction
 */
$transaction = $gateway->fetchTransaction();
$transaction->setTransactionReference('ch_DaHuXHjHeX98GO84COzbfTiP');
$response = $transaction->send();
```

### Fetch Charge List (查询 Charge 列表)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchTransactionListRequest $transactionList
 */
$transactionList = $gateway->fetchTransactionList(array(
    'appId' => $appId,
    'channel' => Channels::ALIPAY,
    'paid' => 0,
    'refunded' => 0,
    'createdFrom' => 1481116461,
    'createdTo' => 1477723630,
    'limit' => 2,
));
$response = $transactionList->send();
```

### Refund (创建退款)
```php
/**
 * @var \Omnipay\Pingpp\Message\RefundRequest $refund
 */
$refund = $gateway->refund(array(
    'amount'               => '10.00',
    'transactionReference' => 'ch_DaHuXHjHeX98GO84COzbfTiP',
    'description'          => 'Demo refund description',
    'metadata'             => array('foo' => 'bar'), // optional
));
$response = $refund->send();
```

### Fetch Refund (查询单笔退款)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchRefundRequest $refund
 */
$refund = $gateway->fetchRefund(array(
    'transactionReference' => 'ch_qDun9KKC0uz9G0KSGKaHKybP',
    'refundReference' => 're_Ouz5GSfv1Gm1S4WzTCaXXPSKs',
));
$response = $refund->send();
```

### Fetch Refund List (查询退款列表)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchRefundListRequest $refundList
 */
$refundList = $gateway->fetchRefundList(array(
    'transactionReference' => 'ch_qDun9KKC0uz9G0KSGKaHKybP',
    'limit' => 2,
));
$response = $refundList->send();
```

### Batch Refund (创建批量退款)
```php
/**
 * @var \Omnipay\Pingpp\Message\BatchRefundRequest $batchRefund
 */
$batchRefund = $gateway->batchRefund(array(
    'app'          => $appId,
    'batchRefundReference'      => Helpers::generateBatchRefundReference(),
    'chargeIdList' => array(
        'ch_L8qn10mLmr1GS8e5OODmHaL4',
        'ch_fdOmHaLmLmr1GOD4qn1dS8e5',
    ),
    'description'  => 'Demo batch refund description.', // optional
    'metadata'     => array('foo' => 'bar'), // optional
));
$response = $batchRefund->send();
```

### Fetch Batch Refund (查询单个批量退款批次号)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchBatchRefundRequest $batchRefund
 */
$batchRefund = $gateway->fetchBatchRefund();
$batchRefund->setBatchRefundReference('batch_refund_20160801001');
$response = $batchRefund->send();
```

### Fetch Batch Refund List (查询批量退款列表)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchBatchRefundListRequest $batchRefundList
 */
$batchRefundList = $gateway->fetchBatchRefundList(array(
    'appId' => $appId,
    'limit' => 2,
));
$response = $batchRefundList->send();
```

### Red Envelope (发送红包)
```php
/**
 * @var \Omnipay\Pingpp\Message\RedEnvelopeRequest $redEnvelope
 */
$redEnvelope = $gateway->redEnvelope(array(
    'appId' => $appId,
    'transactionId' => Helpers::generateRedEnvelopeTransactionId(),
    'channel'     => Channels::WX, // only support "wx", "wx_pub" channel
    'subject' => 'Demo subject',
    'body' => 'Demo body',
    'description' => 'Demo description', // optional
    'amount' => 0.01,
    'currency' => 'cny',
    'sender' =>  'Sender Name', // merchant name
    'receiver' => 'Wechat Openid',
    'metadata' => array('foo' => 'bar'), // optional
));
$response = $redEnvelope->send();
```

### Fetch Red Envelope (查询单笔红包)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchRedEnvelopeRequest $redEnvelopeTransaction
 */
$redEnvelope = $gateway->fetchRedEnvelope();
$redEnvelope->setTransactionReference('red_KCabLO58W5G0rX90iT0az5a9');
$response = $redEnvelope->send();
```

### Fetch Red Envelope List (查询红包列表)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchRedEnvelopeListRequest $redEnvelopeList
 */
$redEnvelopeList = $gateway->fetchRedEnvelopeList(array(
    'appId' => $appId,
    'limit' => 2,
));
$response = $redEnvelopeList->send();
```

### Transfer (创建转账)
```php
/**
 * @var \Omnipay\Pingpp\Message\TransferRequest $transfer
 */
$transfer = $gateway->transfer(array(
    'appId' => $appId,
    'channel' => Channels::WX_PUB, // only support "unionpay", "wx_pub" channel
    'channelExtraFields' => array( // optional, different by channel
        'user_name' => 'User Name',
        'force_check' => true
    ),
    'transactionId' => Helpers::generateTransferTransactionId(Channels::WX_PUB),
    'description' => 'Demo description',
    'amount' => 0.01,
    'currency' => 'cny',
    'type' => 'b2c',
    'receiver' => 'Wechat Openid', // optional, different by channel
    'metadata' => array('foo' => 'bar'), // optional
));
$response = $transfer->send();
```

### Cancel Transfer (取消转账)
```php
/**
 * @var \Omnipay\Pingpp\Message\CancelTransferRequest $cancel
 */
$cancel = $gateway->cancelTransfer();
$cancel->setTransactionReference('tr_0eTi1OGqr9iH0i9CePf1a9C0'); // only support "unionpay" channel
$response = $cancel->send();
```

### Fetch Transfer (查询单笔转账)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchTransferRequest $transfer
 */
$transfer = $gateway->fetchTransfer();
$transfer->setTransactionReference('tr_HqbzHCvLOaL4La1ezHfDWTqH');
$response = $transfer->send();
```

### Fetch Transfer List (查询转账列表)
```php
/**
 * @var \Omnipay\Pingpp\Message\FetchTransferListRequest $transferList
 */
$transferList = $gateway->fetchTransferList(array(
    'appId' => $appId,
    'limit' => 2,
));
$response = $transferList->send();
```


## FAQ

### Is it compatible with Ping++ official SDK?

Yes. It's 100% compatible with official API.

### Why use omnipay-pingpp instead of official SDK?

- Because it's simpler, more elegant, more consistantly designed
- Because the implementation to the official API is more covered than SDK
- Because it's fully unit tested

## Terminology

- `transactionId` is the Merchant’s reference to the transaction - so typically the ID of the payment record in the Merchant Site’s database. In Ping++, it's often called `order_no`.
- `transactionReference` is the Payment Gateway’s reference to the transaction. In Ping++, it's often called `Charge Id`, `Red Envelope Id`, `Transfer Id`.
- `returnUrl` is used by drivers when they need to tell the Payment Gateway where to redirect the customer following a transaction. Typically this is used by off-site ‘redirect’ gateway integrations. In Ping++, it's called differently by various payment channels.
- `notifyUrl` is used by drivers to tell the Payment Gateway where to send their server-to-server notification, informing the Merchant Site about the outcome of a transaction. In Ping++, it's called differently by various payment channels.
