--- 
layout: post
title: Authentication--Brute-force(暴力破解用户登录认证)
subtitle:
date: 2020-09-18
author: D
header-img:
catalog: true
tags: [authentication,brute-force]
---
# 1.暴力破解
### 1.1 通过不同的响应枚举用户名(Username enumeration via different responses)
比如：
1.输入错误的用户名，返回`Invalid username`错误信息.可以利用`Invalid username`枚举出正确的用户名.
2.输入错误的密码，返回`Incorrect password`错误信息.可以利用`Incorrect password`枚举出正确的密码.
可以利用 **Burp Suite** 里面的组件 **Intruder** 或者 扩展 **Turbo Intruder** 来实现用户名和密码的枚举.
直到发现真正有效的用户名，以及跟用户名相匹配的密码.
执行过程如下:
With Burp running, investigate the login page and submit an invalid username and password. 
- 1.运行 Burp，调查登录页面并提交无效的用户名和密码。
- 2.在 Burp 的 <kbd>Proxy</kbd> 标签下的 <kbd>HTTP history</kbd> 标签找到 `POST /login` 请求，并发送到 **Burp Intruder**.
- 3.在 Intruder中，转到 <kbd>Positions</kbd> 选项卡,确保选择攻击类型`Sniper`.
- 4.单击<kbd>Clear §</kbd>按钮，以删除任何自动分配的有效负载位置。 选择POST请求里面的 `username`，然后单击 <kbd>Add §</kbd> 按钮，以将有效负载位置添加到此参数。 
      此位置将由两个§符号表示，例如：`username =§invalid-username§`
- 5.暂时先将密码保留为任何静态值。比如 `password=1`
- 6.在<kbd>Payloads</kbd>选项卡, 选择 payload 类型为 `Simple list`.
- 7.把候选的用户名单粘贴到<kbd>Payload options</kbd>里面，然后点击<kbd>Start attack</kbd>，攻击将会打开一个新的窗口.
- 8.攻击完成后，在“Results”选项卡上，检查“Length”列。 单击列标题以对结果进行排序。 请注意，其中一个条目与其他条目不同。
- 9.检查此响应。 请注意，其他响应包含消息“Invalid username”，但是此响应显示“Incorrect password.”。 记下该用户名。                                                                    
- 10.关闭攻击，然后返回 <kbd>Positions</kbd> 标签。 再次单击 <kbd>Clear §</kbd>按钮，然后将username参数更改为刚确定的用户名。 
      将有效负载位置添加到`password`参数。`username=刚确定的用户名＆password=§invalid-password§`
- 11.在“Payloads”选项卡上，清除用户名列表，并将其替换为候选密码列表。 然后点击<kbd>Start attack</kbd>按钮.
- 12.攻击完成后，请查看“Status”列。 请注意，每个请求都返回了200状态代码，直到最终一个请求返回`302`。这表明登录尝试已成功。 记下密码.
- 13.返回浏览器，使用您标识的用户名和密码登录.

### 1.2 通过细微的不同响应枚举用户名(Username enumeration via subtly different responses)
例子:
- 1.在Burp运行的情况下，提交无效的用户名和密码。 将`POST / login` 请求发送到 Burp Intruder，并将有效负载位置添加到 `username` 参数。
- 2.在“Payloads”选项卡上，确保选择了有效载荷类型“Simple list”，并添加候选用户名列表。
- 3.在 <kbd>Options</kbd> 选项卡上的 "Grep - Extract" 下，单击 <kbd>Add</kbd>。 在对话框中，向下滚动响应，直到找到错误消息`Invalid username or password.`。
      使用鼠标突出显示消息的文本内容。 其他设置将自动调整。 单击<kbd>OK</kbd>，然后开始攻击。
- 4.攻击完成后，请注意，还有**一列**包含您提取的错误消息。 使用此列对结果进行**排序**，以发现其中之有细微的不同
- 5.仔细查看此响应，请注意该响应在错误消息中包含一个错字-而不是`.`，而是尾随空格。即一般是`Invalid suername or password.`而它却是`Invalid username or password ` 记下该用户名。
- 6.关闭攻击窗口回到<kbd>Positions</kbd>选项卡. 插入刚确定的用户名,而且在password的参数添加payload即:`username=确定的用户名&password=§invalid-password§`
- 7.在<kbd>Payloads</kbd>标签, 清除候选的用户名，添加候选的密码.然后点击<kbd>Start attack</kbd>. 
- 8.攻击完成后，请注意，其中一个请求返回了302响应。 记下该密码。
- 9.返回浏览器，使用破解出来的用户名和密码登录。

### 1.3 通过响应时间枚举用户名(Username enumeration via response timing)
例子：
- 1.在Burp运行的情况下，提交无效的用户名和密码，然后将`POST / login`请求发送到 Burp Repeater。试用不同的用户名和密码。请注意，如果您进行过多无效的登录尝试，您的IP将被阻止。
- 2.确定支持`X-Forwarded-For`标头，这使您可以欺骗IP地址并绕过基于IP的暴力保护。
- 3.继续尝试使用用户名和密码。 请特别注意响应时间。 请注意，当用户名无效时，响应时间大致相同。 但是，当您输入有效的用户名（您自己的）时，响应时间会根据您输入的密码的长度而增加。
- 4.将此请求发送到Burp Intruder，然后将攻击类型选择为"Pitchfork"。 清除默认的有效负载位置，然后添加`X-Forwarded-For`标头。
- 5.为`X-Forwarded-For`标头和`username`参数添加有效负载位置。 将密码设置为一个非常长的字符串（应该输入大约100个字符）。
- 6.在<kbd>Payloads</kbd>选项卡上，选择有效载荷集1.选择"Numbers"有效载荷类型。 输入范围1-100，并将步长设置为1。将最大分数位数设置为0。这将用于伪造您的IP达到欺骗服务器的目的。
- 7.选择有效负载集2并添加用户名列表，开始攻击。
- 8.攻击完成后，在对话框顶部，单击 "Columns"，然后选择"Response received"和"Response completed"选项。 现在，这两列显示在结果表中。
- 9.请注意，其中一个响应时间明显长于其他响应时间。 重复几次此请求以确保它持续花费更长的时间，然后记下该用户名。
- 10.为相同的请求创建新的Burp Intruder攻击。 再次添加`X-Forwarded-For`标头，并向其添加有效负载位置。 插入刚刚标识的用户名，并将有效负载位置添加到`password`参数。
- 11.在<kbd>Payloads</kbd>选项卡上，将有效载荷组1中的数字列表添加到有效载荷组2中，并将密码列表添加到有效载荷组2中。发起攻击。
- 12.攻击完成后，找到状态为302的响应。 记下该密码。
- 13.返回浏览器，重新加载登录页面，以便您拥有有效的CSRF令牌。 然后，使用您记下的用户名和密码登录。
(该方法效果不是很明显，而且时间差的特征不稳定，至少在中国对外国网站发起攻击是如此)

### 1.4 暴力破解保护，IP阻塞.(Broken brute-force protection, IP block)
例子：
- 1.在Burp运行的情况下，调查登录页面。 如果您连续提交3个错误的登录，请观察您的IP被阻止。 但是，您可以通过在达到限制**之前**登录自己的帐户来重置计数器。
- 2.输入非法的用户名和密码,然后发送 `POST /login`请求到 Burp Intruder.创建 **pitchfork** 攻击,在`username`和`password`位置都添加 payload.
- 3.在<kbd>Payloads</kbd>标签,选择有效载荷设置为1. 添加在您的用户名和`carlos`之间交替的有效载荷列表。 确保您的用户名是第一位，并且`carlos`至少重复了100次。
      比如您有效的用户名和密码为:wiener:peter  要被攻击的用户名为:carlos 它的密码未知.要防止IP阻塞，那么就要交替登录有效用户和被攻击的用户。尝试登录顺序为:
```
wiener:peter
carlos:尝试密码1
wiener:peter
carlos:尝试密码2
...
wiener:peter
carlos:尝试密码n
...
```
- 4.编辑候选密码列表，并在每个密码之前添加您自己的密码。 确保您的密码与另一个列表中的用户名对齐。 将此列表添加到有效负载集2中并开始攻击。
- 5.将此列表添加到有效负载集2中并开始攻击。
- 6.攻击完成后，请按用户名和响应代码对结果进行排序。 成功登录到Carlos帐户的请求将收到302响应。

### 1.5 通过帐户锁定枚举用户名(Username enumeration via account lock)
例子:
- 1.在Burp运行时，调查登录页面并提交无效的用户名和密码。 将`POST / login`请求发送到Burp Intruder。
- 2.选择攻击类型"Cluster bomb"。 将有效负载位置添加到`username`参数。 在请求的末尾添加一个任意的附加参数，并向其添加第二个有效负载位置。例如：
**Burp Intruder**：
```
username=§invalid-username§&password=example&count=§0§
```
**Turbo Intruder**代替Burp Intruder那么有两个参数的Turbo Intruder配置如下：
```
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=50,
                           requestsPerConnection=1,
                           pipeline=True
                           )

    for i in range(3, 8):
        engine.queue(target.req, randstr(i), learn=1)
        engine.queue(target.req, target.baseInput, learn=2)

    for firstWord in open('/home/f/username'):
        for secondWord in open('/home/f/count'):
            engine.queue(target.req, [firstWord.rstrip(), secondWord.rstrip()])


def handleResponse(req, interesting):
    if 'You have made too many incorrect login attempts' in req.response:
        table.add(req)
```
- 3.在<kbd>Payloads</kbd>选项卡上，将用户名列表添加到第一个有效载荷集，并将数字1-5添加为第二个有效载荷集。 这将导致用户名重复5次(假设:有效的用户名尝试5次密码失败就被锁住),然后开始攻击。
- 4.在结果中，请注意，其中一个用户名的响应比使用其他用户名时的响应**长**。 仔细研究响应，并注意它包含不同的错误消息：**You have made too many incorrect login attempts.**, 记下该用户名。
- 5.在`POST /login`登录请求上创建新的Burp Intruder攻击，但是这次选择"Sniper"攻击类型。 将`username`参数设置为刚标识的用户名，并将有效负载位置添加到`password`参数。
- 6.**Burp Intruder**:将密码列表添加到有效负载集中，并为错误消息创建grep提取规则。
    **Turbo Intrude**:则把`handleResponse`条件改为`if '200' in req.response`即可. 开始攻击。
- 7.**Burp Intruder**:在结果中，查看grep提取列。 请注意，有几个不同的错误消息，但是其中一个响应**不**包含任何错误消息。
    **Turbo Intruder**:它有一列`Words`长度是不一样的，点击Words排序就很容易找到密码了 记下该密码。
      (有时虽然提示已经阻塞的错误信息，但是依然可以发送登录信息).
- 8.在浏览器中，等待一分钟(有些久的15分钟左右)以重置帐户锁定，然后使用您标识的凭据登录。

### 1.6 Broken brute-force protection, multipe credentials per request
### 1.7 Brute-forcing a stay-logged-in cookie
