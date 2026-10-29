# skyland-auto-sign

明日方舟、明日方舟：终末地的森空岛签到脚本，支持本地、GitHub Actions、Docker/NAS 与华为云 FunctionGraph。

有四种方法部署
> 1.自己手动在电脑上运行（不想挂载在服务器上的用户推荐这个，因为可以配合MAA一键签到）
>
> 2.在云函数上部署（推荐，省时省力，不用装环境，几乎不用管）
>
> 3.使用Github Actions托管（相比云函数来讲非常推荐，因为有日志会存储，而且也免费）
>
> 4.使用家用NAS面板部署
>

## 快速导航

- [在自己电脑上运行](#mode1)
    - [配合MAA运行](#maa)
- [在云函数上运行](#mode2)
  - [使用手机进行操作](#mobile)
- [使用Github Actions托管](#mode3)
- [使用家用NAS](#mode4)
- [多账号支持](#multiple_account)
- [多端登录问题](#multiple_login)
- [有关新版本参数验证](#sign-headers)
- [有关新版本获得Cred必传的dId参数](#dId)

[视频用法]( https://www.bilibili.com/video/BV1DP411h7s6)

<a name="mode1"></a>

## 方法1

1.安装 Python 3.11 或以上版本。

2.克隆源代码，并在仓库根目录执行 `python -m pip install -r requirements.txt` 安装固定版本依赖。

3.命令提示符运行`python src/main.py`（结果输出如下就代表你成功了）
![img.png](assets/img_0.png)

（以上3步可以用我打好的exe文件直接运行，链接见[release](https://gitee.com/FancyCabbage/skyland-auto-sign/releases)

4.本软件有3种模式：

第一个模式和第二个模式不多说了，填入你的信息即可。

就是注意一下输入的密码是不会显示的，但实际是有的，这点需要注意一下！

如果第一种模式和第二种模式你都没法过人机验证的话，可以使用第三种模式：

登录电脑的森空岛[官网](https://www.skland.com/)
![img.png](assets/img.png)

登录完成后，访问[这个网址](https://web-api.skland.com/account/info/hg) 。然后复制这里的所有内容，粘贴进去即可。
![img_1.png](assets/img_1.png)
![img_2.png](assets/img_2.png)

最后再次启动的时候，脚本就不会再询问你是否添加账号了，而是直接帮你签到了

1.3版本在脚本路径下多了一个`添加账号.bat`,双击运行会再次出现登陆界面，按照步骤走可以再次添加一个账号
<a name="maa"></a>

### MAA支持

可以使用MAA自动执行脚本

MAA在连接设置里有运行前脚本，输入`main.py`的路径后，每次连接模拟器时会自动帮你签到。

如果你用的exe版本的，请输入`main.exe`的路径！
![img_3.png](assets/img_3.png)

第一次执行的话应该还是会出现这个东西
![img_4.png](assets/img_4.png)

设置完毕后直接应该会执行完成，以后都会这样
![img_5.png](assets/img_5.png)

TOKEN和日志应该都会被存储在MAA根路径下
![img_6.png](assets/img_6.png)

<a name="mode2"></a>

## 方法2

使用华为云 FunctionGraph 的事件函数与定制运行时。

`cloud_functions/` 已提供可直接上传的 **[预构建部署包](cloud_functions/cloud_functions.zip)**、启动脚本和部署说明，签到入口在 `src/index.py`。下载现成包无需启用 GitHub Actions，也无需自己编译。

下载 `cloud_functions/cloud_functions.zip` 后直接上传到云函数。修改源码后，维护者可在 Linux 手动重新构建，或运行 Actions 的 `Checks` 获取新产物，再更新仓库里的部署包及校验信息。

配置 `TOKEN` 环境变量后执行测试调用；多账号支持换行或英文逗号分隔。三个推送渠道与本地、Actions 共用，任一签到失败会向华为云上报错误。最后根据控制台当前 Cron 语法和时区选项配置定时触发器。

完整步骤、手动构建方式和运行环境要求见 [华为云部署说明](cloud_functions/README.md)。构建产物不包含账号凭据；具体云端兼容性和实际签到结果需通过部署后的测试调用确认。

<a name="mobile"></a>

### 使用手机操作

手机可直接下载仓库中的 [cloud_functions.zip](cloud_functions/cloud_functions.zip)，再按华为云控制台支持的上传方式部署；具体操作见上面的部署说明。

<a name="mode3"></a>

## 方法3

使用Github Action自动运行脚本

本方法需要你有一定的git使用经验

首先确认自己的github处于登录状态，然后自己新建一个仓库

将本仓库的源代码全部下载下来，上传到github

进入自己仓库的项目主页后，在上方的菜单中进入Settings > Secrets and variables > Actions

点击 New repository secret


 **创建名为`TOKEN`的环境变量（注意变量名全大写），并填入你的鹰角网络通行证，如果要管理多个账号，换行即可** 

 **如果要开启 Server酱³ APP 推送的话，就创建名为 `SC3_SENDKEY` 的环境变量；如果是第一次使用 Server酱³ 的话，需要到 [Server酱³ 官网](https://sc3.ft07.com/) 注册一个账号，再在手机上下载 Server酱³ App。**

`SC3_SENDKEY`: Server酱³ SendKey（形如 sctp12345tXXXX...）


如果是第一次使用GitHub Action的话，还需要手动打开这个功能 在你仓库上方菜单中进入Actions

点击 I understand... enable them > Enable workflow

之后就可以自动运行签到了, 想要手动测试的话，选择左侧的Auto Sign > Run workflow, 刷新页面就能看到结果了

签到失败默认返回非零退出码，工作流会标记为失败；可设置 Variable `EXIT_WHEN_FAIL=off` 关闭这一退出策略。推送失败默认记录错误，设置 `EXIT_WHEN_PUSH_FAIL=on` 后也会使工作流失败。GitHub 邮件通知仍取决于账号的 Actions 通知设置。

还支持 `PUSHPLUS_TOKEN`、`QMSG_TOKEN` 两个可选 Secret。推送检查渠道业务返回值；“请求被服务端接受”不等于通知最终送达。

<a name="mode4"></a>

## 使用NAS部署

其实和Github Action的配置方式一样，导入脚本以后， **创建一个`TOKEN`的环境变量** 即可。

 **如果要开启 Server酱³ APP 推送的话，就创建名为 `SC3_SENDKEY` 的环境变量。如果是第一次使用 Server酱³ 的话，需要到 [Server酱³ 官网](https://sc3.ft07.com/) 注册一个账号，再在手机上下载 Server酱³ App。** 

`SC3_SENDKEY`: Server酱³ SendKey（形如 sctp12345tXXXX...）。


每个面板可能创建方式不太一样，不展示了

Docker 可通过 `docker build -t skyland-auto-sign .` 构建，再使用 `docker run --rm --env-file .env skyland-auto-sign` 运行。在本地 `.env` 中设置 `TOKEN` 及所需推送密钥；该文件、账号文件、实际推送配置和日志均被构建排除，镜像中不包含它们。容器以普通用户运行。

本地推送配置也可将 `src/push/config.example.ini` 复制为 `src/push/config.ini` 后填写。实际配置文件已被 Git 和 Docker 忽略；环境变量优先于文件，旧拼写 `PUSHPLUSH_TOKEN` 会兼容并提示改名。

网络请求默认连接/读取超时为 5/20 秒，不会自动重试签到 POST。使用代理时设置 `USE_PROXY=on`，可用 `PROXY_URL` 指定地址；`off`、`false`、`0` 均会关闭这个显式代理开关。系统标准代理环境变量仍按 Requests 规则生效。HTTPS 始终校验证书，私有 CA 可通过 `REQUESTS_CA_BUNDLE` 配置。

<a name="multiple_account"></a>

## 多账号支持

普通 Token 支持换行或英文逗号分隔，解析后去重；也支持官网返回的完整账号 JSON、每行一个账号 JSON 或 JSON 数组。文件配置可用 `TOKEN_FILE` 指定路径。无账号的后台任务会明确失败，不进入交互提示。

每个角色和账号独立处理：一个角色失败不会跳过后续角色，已完成结果会保留。认证失效只报告当前账号的问题，不会删除保存其他账号的文件。明确的已签到响应按成功处理，未知非零业务码按失败处理。

## 开发检查

```sh
python -m pip install -r requirements.txt
python -m unittest discover -s test -v
python -m pip check
```

测试禁止真实游戏和推送请求，覆盖失败状态、凭据保留、多账号解析、代理证书校验、日志脱敏、推送业务错误与云函数入口。`Checks` 工作流另外验证 Linux 部署包和 Docker 镜像不会携带本地凭据。

### 使用浏览器登录多个账号获得TOKEN时要注意的问题

**不要去登出账号，否则鹰角网络通行证会失效！**

如果要添加多个账号，请删除浏览器缓存。或者使用浏览器自带的隐私浏览模式，拿到Token后，关闭隐私窗口，再登录一次即可！
![img_20.png](assets/img_20.png)
![img_21.png](assets/img_21.png)

<a name="multiple_account"></a>

## 多端登录的问题

同一账号多端登录是没问题的，但是要注意一点就是电脑在用密码登录后，手机客户端有可能会被挤掉

最后就是别手贱去点客户端里的清理会话，因为那样子会把所有的登录状态清空

## sign-header

新版本森空岛添加了参数验证，有两个方法可以绕过，推荐第一种方法

1.森空岛签名请求头算法：

接口地址（不包括网址） + 如果是GET则使用网址问号后面的参数。如果是POST则使用body的json的字符串 + 时间戳 +
请求头的四个重要参数(`platform`,`timestamp`,`dId`和`vName`).toJSON()

将此字符串做HMAC加密，算法为SHA-256，密钥为请求cred接口会返回的一个token

再将加密后的字符串做MD5即得到签名

例子：

```text
    
    拼接的字符串：
    /api/v1/game/player/binding1695184599{"platform":"","timestamp":"1695184599","dId":"","vName":""}
    将其做hmac-sha256
    6b0516d1325dea1207e03f6840cd0c15ef2f55959c3d0ed2f18d99102a9cc7f5
    再做md5
    92229bf3c18c476e77bcd70c2bd997d3
    最后就是sign的头了
    
```

注意：

请求头四个值`platform`,`timestamp`,`dId`和`vName`必传，然后注意一下字符串顺序，四个值必须按照这样的顺序拼接

其中`platform`,`dId`和`vName`里面的值随便填

`timestamp`为时间戳，以秒为单位
不过`timestamp`因为精度问题容易造成服务器响应`请勿修改设备时间`，此时把这个时间戳稍微减少几秒就行


2.使用旧版本森空岛请求头，防止验参

```text
'User-Agent': 'Skland/1.0.1 (com.hypergryph.skland; build': '100001014; Android 31; ) Okhttp/4.11.0',
'vCode': '100001014',
'vName': '1.0.1',
'dId': 'de9759a5afaa634f',
'platform': '1'
```

<a name="dId"></a>
## 有关新版本获得Cred必传的dId参数

2024.9.10 森空岛登入接口引入了数美接口，请求头必须传递dId参数，导致无法正常登陆

现已解决，具体实现看`SecuritySm.py`文件
