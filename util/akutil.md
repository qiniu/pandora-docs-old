签名是保护用户资源安全的重要策略，但是由于生成签名过于繁琐，所以用户可以借鉴此工具来帮助生成签名。

#### 1.填写信息

[工具链接](http://pandora-toolkits.qiniu.com/auth)

打开工具链接，依次填写AK、SK、Repo，然后选择本次请求的API类型；

* AK、SK是七牛账户的公钥和私钥，通过七牛控制台——个人面板中的 `秘钥管理` 页面可以找到；
* Repo是指实时工作流中的 `数据源` 节点的名称；

![](/Users/loris/liurui/pandora-docs-old/_media/akutil1.png)

#### 2.生成签名

填写完成相应内容后，点击 `生成签名` ；
然后将页面上的第五个面板的内容复制即可；

![](/Users/loris/liurui/pandora-docs-old/_media/akutil2.png)

#### 3.生成token

填写完成相应内容后（包括过期时间），点击 `生成token` ，在 `5.生成的curl命令(需要自己加data, -d)` 面板中会生成对应的token。

#### 4.token与签名的区别

token和签名都能用于API认证，区别是token可以设置有效时间，而签名是一次性使用的。

在生成的字符串特征上，token由冒号`:`分隔为三段，签名为两段。

#### 5.token的使用场景

在不想把 ak，sk 泄露出去的场景下，如手机端、lot的一些终端设备上，可以生成token。token仅对固定的API（repo）有效。

如您希望手机端也能直接上报数据到Pandora，即可根据一个特定的repo生成一个打点的token，这个token只能用于对着这个repo打点，没有任何访问其他接口的能力。