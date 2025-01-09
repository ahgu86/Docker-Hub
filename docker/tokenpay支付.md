### token-pay支付 虚拟货币支付

#### 创建配置文件
```
mkdir -p tokenpay && cd tokenpay && touch appsettings.json TokenPay.db docker-compose.yaml
```

#### 编辑`appsettings.json`配置
```
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.Hosting.Lifetime": "Information"
      }
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DB": "Data Source=TokenPay.db;"
  },
  "TRON-PRO-API-KEY": "xxxxxx-xxxx-xxxx-xxxxxxxxxxxx", // 避免接口请求频繁被限制，此处申请 https://www.trongrid.io/dashboard/keys
  "BaseCurrency": "CNY", //默认货币，支持 CNY、USD、EUR、GBP、AUD、HKD、TWD、SGD
  "Rate": { //汇率 设置0将使用自动汇率
    "USDT": 0,
    "TRX": 0,
    "ETH": 0,
    "USDC": 0
  },
  "ExpireTime": 1800, //单位秒
  "UseDynamicAddress": false, //是否使用动态地址，设为false时，与EPUSDT表现类似；设为true时，为每个下单用户分配单独的收款地址
  "Address": { // UseDynamicAddress设为false时在此配置TRON收款地址，EVM可以替代所有ETH系列的收款地址，支持单独配置某条链的收款地址
    "TRON": [ "TLUF41C386Cxxxxxxxxxxxxxxxxxxxx" ],
    "EVM": [ "0x9966aA2xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" ]
  },
  "OnlyConfirmed": false, //默认仅查询已确认的数据，如果想要回调更快，可以设置为false
  "NotifyTimeOut": 3, //异步通知超时时间
  "ApiToken": "78907890", //异步通知密钥，请务必修改此密钥为随机字符串，脸滚键盘即可
  "WebSiteUrl": "https://tokenpay.xxxxx.com", //配置服务器外网域名
  "Collection": { //需要 UseDynamicAddress 为 true 才有使用归集功能的意义，静态地址收款TokenPay无法归集
    "Enable": false, //是否启用归集功能，false 表示关闭，true 表示启用
    "UseEnergy": true, //是否租用能量归集，降低归集成本，false 表示直接燃烧trx转账
    "ForceCheckAllAddress": false, //强制检查所有地址的余额
    "RetainUSDT": true, //归集USDT时是否保留0.000001，用于降低用户下次支付的成本
    "CheckTime": 1, //归集任务运行间隔，默认1小时运行一次，单位：小时
    "MinUSDT": 0.1, //只归集USDT余额大于此金额的地址
    "NeedEnergy": 65000, //归集USDT所需能量，此配置项通常不需要修改，发生变化时会在GitHub更新
    "EnergyPrice": 210, //波场当前能量单价，此配置项通常不需要修改，发生变化时会在GitHub更新
    "Address": "TLUF41C386Cxxxxxxxxxxxxxxxxxxxx" //归集收款地址，配置你自己的收款地址
  },
  "Telegram": {
    "AdminUserId": 12345678, // 你的TG账号ID，可在 https://t.me/creationdatebot 获取ID
    "BotToken": "1234567890:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" //从https://t.me/BotFather 创建机器人时，会给你BotToken
  },
  "RateMove": { //汇率微调，支持设置正负数，仅支持两位小数
    "TRX_CNY": 0,
    "USDT_CNY": 0
  },
  "DynamicAddressConfig": {
    "AmountMove": false, //使用动态地址收款时启用动态金额，支持非准确金额支付，用于优化中心化钱包或交易所提币扣除手续费后金额不匹配的情况，可自行决定是否开启，默认false表示不启用
    "TRX": [ 0, 2 ], //表示下浮0，上浮2，如果实际支付金额为100TRX，根据此配置，用户支付金额在100-102TRX订单都会成功
    "USDT": [ 1, 2 ], //表示下浮1，上浮2，如果实际支付金额为100USDT，根据此配置，用户支付金额在99-102USDT订单都会成功
    "ETH": [ 0.1, 0.15 ] //表示下浮0.1，上浮0.1，如果实际支付金额为0.5ETH，根据此配置，用户支付金额在0.4-0.65ETH订单都会成功
  }
}
```

>使用代理，添加配置
```
"WebProxy":"socks5://127.0.0.1:1080"
"Temegram":{
  ",,,"
}
```
## 运行
```
docker run -d \
  --restart always \
  -p 5000:8080 \
  -v ./appsettings.json:/app/appsettings.json \
  -v ./TokenPay.db:/app/TokenPay.db \
  dapiaoliang666/tokenpay
```

然后将外网域名反代到`5000`端口


如果需要重新部署需要清空`/TokenPay.db`文件里的内容

### 修改tokenpay付款金额小数后四位
添加到15行

改为2位示例
```
"Decimals:USDT_TRC20": 2,
```
如果需要定义其他币种，把USDT_TRC20换成相应币种



## `v2board`对接`TokenPay`

### 1. 将插件复制到`v2board`对应目录
### 2. 到`v2board`后台-**支付配置**中添加支付方式
注意事项
1. API地址末尾请不要有斜线，如`https://token-pay.xxx.com`  
2. 如果你要同时支持USDT和TRX付款，你需要添加两条支付方式，依此类推  

请参考此图填写
<img src="../png/v2board.png" alt="v2board支付方式配置"/>


### 插件代码

创建文件`TokenPay.php`复制到`v2board`支付目录

```
<?php

namespace App\Payments;

use \Curl\Curl;

class TokenPay {
    public function __construct($config)
    {
        $this->config = $config;
    }

    public function form()
    {
        return [
            'token_pay_url' => [
                'label' => 'API 地址',
                'description' => '您的 TokenPay API 接口地址(例如: https://token-pay.xxx.com)',
                'type' => 'input',
            ],
            'token_pay_apitoken' => [
                'label' => 'API Token',
                'description' => '您的 TokenPay API Token',
                'type' => 'input',
            ],
            'token_pay_currency' => [
                'label' => '币种',
                'description' => '您的 TokenPay 币种，如 USDT_TRC20、TRX',
                'type' => 'input',
            ]
        ];
    }

    public function pay($order)
    {
        $params = [
			"ActualAmount" => $order['total_amount'] / 100,
			"OutOrderId" => $order['trade_no'], 
			"OrderUserKey" => strval($order['user_id']), 
			"Currency" => $this->config['token_pay_currency'],
			'RedirectUrl' => $order['return_url'],
			'NotifyUrl' => $order['notify_url'],
        ];
        ksort($params);
        reset($params);
        $str = stripslashes(urldecode(http_build_query($params))) . $this->config['token_pay_apitoken'];
        $params['Signature'] = md5($str);

        $curl = new Curl();
        $curl->setUserAgent('TokenPay');
        $curl->setOpt(CURLOPT_SSL_VERIFYPEER, 0);
        $curl->setOpt(CURLOPT_HTTPHEADER, array('Content-Type:application/json'));
        $curl->post($this->config['token_pay_url'] . '/CreateOrder', json_encode($params));
        $result = $curl->response;
        $curl->close();

        if (!isset($result->success) || !$result->success) {
            abort(500, "Failed to create order. Error: {$result->message}");
        }

        $paymentURL = $result->data;
        return [
            'type' => 1, // 0:qrcode 1:url
            'data' => $paymentURL
        ];
    }

    public function notify($params)
    {
        $sign = $params['Signature'];
        unset($params['Signature']);
        ksort($params);
        reset($params);
        $str = stripslashes(urldecode(http_build_query($params))) . $this->config['token_pay_apitoken'];
        if ($sign !== md5($str)) {
            die('cannot pass verification');
        }
        $status = $params['Status'];
        // 0: Pending 1: Paid 2: Expired
        if ($status != 1) {
            die('failed');
        }
        return [
            'trade_no' => $params['OutOrderId'],
            'callback_no' => $params['Id'],
            'custom_result' => 'ok'
        ];
    }
}
```




### 原版部署

- [官方地址](https://github.com/LightCountry/TokenPay)

- `docker-compose.yaml`

```
services:
  token-pay:
    build: .
    restart: always
    ports:
      - "5000:8080"
    volumes:
      - ./appsettings.json:/app/appsettings.json
      - ./TokenPay.db:/app/TokenPay.db
      - ./Pay.cshtml:/app/Views/Home/Pay.cshtml
```
> `- ./EVMChains.json:/app/EVMChains.json` 更多区块链添加此文件映射进去



- `Pay.cshtml`美化代码

```
@{
    ViewData["Title"] = "支付页";
    var ExpireTime = ViewData.ContainsKey("ExpireTime") ? Convert.ToDateTime(ViewData["ExpireTime"]).ToUniversalTime().ToString("o") : DateTime.UtcNow.ToString("o");
}
@using TokenPay.Domains;
@using TokenPay.Extensions;
@using TokenPay.Models.EthModel;
@model TokenPay.Domains.TokenOrders;
@inject List<EVMChain> chain;

@if (Model == null)
{
    <div class="row align-items-center h-100">
        <div class="text-center">
            <h1 class="display-4">订单不存在！</h1>
        </div>
    </div>
}
else
{
    <link href="https://cdnjs.cloudflare.com/ajax/libs/tailwindcss/2.2.19/tailwind.min.css" rel="stylesheet">
    <style>
        /* 全局样式 */
        :root {
            --bs-gutter-x: 0.2rem;
            --primary-color: rgba(0, 0, 0, 0.7);
            --secondary-color: rgba(107, 70, 193, 0.9);
            --accent-color: rgba(0, 0, 0, 0.9);
            --text-color: rgba(31, 41, 55, 0.9);
            --gradient-color-1: rgba(255, 255, 255, 0.1);
            --gradient-color-2: rgba(233, 233, 233, 0.2);
            --gradient-color-3: rgba(200, 200, 200, 0.3);
        }

        blockquote, dd, dl, figure, h1, h2, h3, h4, h5, h6, hr, p, pre {
            margin: revert;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: var(--text-color);
            background: linear-gradient(-45deg, var(--gradient-color-1), var(--gradient-color-2), var(--gradient-color-3));
            background-size: 400% 400%;
            animation: gradientBG 15s ease infinite;
            min-height: 100vh;
        }

        .card {
            background-color: rgba(248, 248, 248, 0.8);
            border-radius: 1rem;
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.2);
            padding: 0.2rem;
            backdrop-filter: blur(15px);
            transition: all 0.3s ease;
        }

        .title {
            font-size: 1.5rem;
            color: rgb(0, 0, 0, 0.5);
            text-shadow: 2px 2px 6px rgba(0, 0, 0, 0.3);
        }

        .btn-primary {
            background-color: var(--primary-color);
            color: white;
            transition: all 0.3s ease;
            padding: 0.2rem 0.6rem;
            border: none;
            border-radius: 0.5rem;
        }

        .btn-primary:hover {
            background-color: var(--secondary-color);
        }

        .timer {
            font-size: 1.5rem;
            background: linear-gradient(45deg, var(--primary-color), var(--secondary-color));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        .warning {
            color: rgb(0, 0, 0);
            animation: pulse 2s infinite;
        }

        .qr-code {
            border-radius: 1rem;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
            margin: 1rem 0;
        }

        .text-black {
            color: black !important;
        }

        .center {
            text-align: center;
        }

        .my-4 {
            margin-top: 0.5rem;
            margin-bottom: 0.5rem;
        }

        /* 新增和修改的样式 */
        .copyable-container {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            margin-top: 0.5rem;
            cursor: pointer;
            position: relative;
            padding: 0.5rem;
        }

        .copyable-content {
            border: 2px dashed rgba(0, 0, 0, 0.9);
            border-radius: 0.5rem;
            padding: 0.2rem;
            position: relative;
            overflow: hidden;
        }

        .copyable-content::before {
            content: '';
            position: absolute;
            top: -2px;
            left: -2px;
            right: -2px;
            bottom: -2px;
            background: repeating-linear-gradient(
                45deg,
                rgba(0, 0, 0, 0.9),
                rgba(0, 0, 0, 0.3) 10px,
                transparent 10px,
                transparent 20px
            );
            animation: snake 20s linear infinite;
            pointer-events: none;
        }

        .popup {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 1rem 2rem;
            border-radius: 0.5rem;
            opacity: 0;
            transition: opacity 0.3s ease;
        }

        .popup.show {
            opacity: 1;
        }

        /* 使用 Razor 代码块来包装 keyframes */
        @{
            <text>
                @@keyframes snake {
                    0% {
                        background-position: 0 0;
                        background: repeating-linear-gradient(
                            45deg,
                            rgba(0, 0, 0, 0.05),
                            rgba(0, 0, 0, 0.05) 10px,
                            transparent 10px,
                            transparent 20px
                        );
                    }
                    100% {
                        background-position: 400px 0;
                        background: repeating-linear-gradient(
                            45deg,
                            rgba(0, 0, 0, 0.05),
                            rgba(0, 0, 0, 0.05) 10px,
                            transparent 10px,
                            transparent 20px
                        );
                    }
                }
            </text>
        }
    </style>

    <div class="container mx-auto px-2 h-screen flex flex-col">
        <div class="card animate__animated animate__fadeIn flex-grow h-auto">
            <h1 class="title text-center animate__animated animate__bounceIn">支付详情</h1>

            <div class="timer text-center mb-4 animate__animated animate__pulse animate__infinite time">
                剩余时间：<span class="text-black" id="remaining-time"></span>
            </div>

            <p class="warning text-center">请仔细核对币种和金额！</p>

            <div class="text-center my-4 flex flex-col items-center">
                <p class="text-black font-bold">区块链：<span class="text-red-500">@Model.Currency.ToBlockchainName(chain)</span> 币种：<span class="text-red-500">@Model.Currency.ToCurrency(chain)</span></p>
                <img src="data:image/png;base64,@ViewData["QrCode"]" class="qr-code border border-gray-300 my-2" alt="收款地址" width="200" height="200">
                <div class="copyable-container" onclick="copyToClipboard('@Model.ToAddress', '地址已复制')">
                    <div class="copyable-content">
                        <span class="text-black font-bold text-sm">@Model.ToAddress</span>
                    </div>
                </div>
            </div>

            <div class="text-center">
                <p class="text-black">支付金额：
                    <span class="copyable-container" onclick="copyToClipboard('@Model.Amount', '金额已复制')">
                        <span class="copyable-content text-black">@Model.Amount @Model.Currency.ToCurrency(chain)</span>
                    </span>
                </p>
                <p class="text-black">订单编号：<span class="text-black">@Model.OutOrderId</span></p>
            </div>
        </div>
    </div>

    <div id="popup" class="popup">复制成功</div>

    @section Scripts {
    <script>
        let timer;
        const endTime = new Date('@ExpireTime');

        function updateRemainingTime() {
            const now = new Date();
            const timeDiff = endTime - now;

            if (timeDiff <= 0) {
                clearInterval(timer);
                document.getElementById('remaining-time').textContent = '已过期';
                return;
            }

            const days = Math.floor(timeDiff / (1000 * 60 * 60 * 24));
            const hours = Math.floor((timeDiff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
            const minutes = Math.floor((timeDiff % (1000 * 60 * 60)) / (1000 * 60));
            const seconds = Math.floor((timeDiff % (1000 * 60)) / 1000);

            document.getElementById('remaining-time').textContent = 
                `${minutes.toString().padStart(2, '0')}分 ${seconds.toString().padStart(2, '0')}秒`;
        }

        function copyToClipboard(value, message) {
            const el = document.createElement('textarea');
            el.value = value;
            document.body.appendChild(el);
            el.select();
            document.execCommand('copy');
            document.body.removeChild(el);

            const popup = document.getElementById('popup');
            popup.textContent = message;
            popup.classList.add('show');
            setTimeout(() => {
                popup.classList.remove('show');
            }, 1000);
        }

        let checkTimer;
        function startCheck() {
            checkTimer = setInterval(Check, 1000);
        }

        function Check() {
            var RedirectUrl = "@(Model?.RedirectUrl)";
            $.get("/Check/@(Model?.Id)")
                .then(x => {
                    if (x === 'Pending') {
                        console.log('待支付');
                    } else if (x === 'Expired') {
                        clearInterval(checkTimer);
                        clearInterval(timer);
                        console.log('订单过期');
                        location.reload();
                    } else if (x === 'Paid') {
                        clearInterval(checkTimer);
                        clearInterval(timer);
                        console.log('已支付');
                        setTimeout(() => {
                            if (RedirectUrl) {
                                location = RedirectUrl;
                            } else {
                                alert("已支付");
                            }
                        }, 0);
                    }
                });
        }

        $(() => {
            updateRemainingTime();
            timer = setInterval(updateRemainingTime, 1000);
            startCheck();
        });
    </script>
    }
}
```
