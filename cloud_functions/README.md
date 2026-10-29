# 华为云 FunctionGraph 部署

此目录保存定制运行时的启动脚本和部署说明。签到入口在 `src/index.py`，与本地和 GitHub Actions 共用签到、推送和脱敏逻辑。编译产物生成在 `dist/cloud_functions/`，ZIP 为 `dist/cloud_functions.zip`，不提交到 Git。

## 获取部署包

在本仓库启用 GitHub Actions，运行 `Checks`。全部检查通过后，在该次运行的 Artifacts 下载 `functiongraph-linux-x86_64`。解开 GitHub 下载的外层压缩包，获得内部的 `cloud_functions.zip`。

该构建使用固定版本的 PyPA manylinux 镜像，目标为 Linux x86_64、glibc 2.28 基线。目标云函数必须选择兼容的 x86_64 环境；构建信息保存在包内 `BUILD_INFO.json`。打包和离线启动检查不能替代在具体华为云环境中的测试调用。

也可以在与目标架构、glibc 兼容的 Linux 环境手动构建：

```sh
python3 -m venv .venv
. .venv/bin/activate
python -m pip install -r requirements-build.txt
sh .build/build_cloud_function.sh
python .build/test_cloud_package.py
```

构建脚本可从任意当前目录启动。输出路径已存在时会停止，避免覆盖填写过的账号文件；再次构建请指定新目录，例如 `sh .build/build_cloud_function.sh --output dist/cloud_functions-new`。自行构建时，glibc 基线取决于实际构建环境。

部署 ZIP 的根目录包含：

```text
bootstrap                           # 可执行启动脚本
index/index                         # 可执行签到程序
index/_internal/...                 # Python 和依赖
INPUT_HYPERGRYPH_TOKEN.txt           # 构建时为空
BUILD_INFO.json
```

## 配置云函数

1. 在 FunctionGraph 创建**事件函数**，选择支持自定义 `bootstrap` 的**定制运行时**，上传 `cloud_functions.zip`。不要把源码目录的 ZIP 直接当作部署包，也不要选择 HTTP 函数模式。
2. 配置环境变量 `TOKEN`，支持换行或英文逗号分隔多个账号，也支持官网返回的完整账号 JSON 或 JSON 数组。推荐使用环境变量，不将凭据放入公开仓库和共享部署包。
3. 可选配置 `SC3_SENDKEY`、`PUSHPLUS_TOKEN`、`QMSG_TOKEN` 开启对应通知；Server酱³ 与 Turbo 的密钥会自动选择各自入口。配置 `TZ=Asia/Shanghai`。
4. 若使用文件，取消 `TOKEN` 环境变量，在部署代码根目录的 `INPUT_HYPERGRYPH_TOKEN.txt` 中逐行填入账号；也可用 `TOKEN_FILE` 指定绝对路径。程序优先使用 `TOKEN`，再使用文件。文件路径基于 `RUNTIME_CODE_ROOT`，不依赖进程当前目录。
5. 根据账号数量配置执行超时，再执行一次测试调用，检查签到汇总和通知。外部请求的连接/读取超时为 5/20 秒；多个账号、角色和推送渠道会增加总耗时。
6. 测试成功后添加定时触发器。Cron 语法和时区按华为云控制台当前选项配置；GitHub Actions 的定时设置不会应用到云函数。

默认任一账号/角色失败都会通过运行时 `/invocation/error/<request-id>` 上报；正常结果走 `/invocation/response/<request-id>`。已成功角色的结果会保留，其余账号继续处理。`EXIT_WHEN_PUSH_FAIL=on` 可将推送请求失败也视为云函数失败；默认只记录推送失败。平台若配置了失败重试，可能再次执行签到，请按需要配置。

日志输出到 stderr，stdout 仅输出一个 JSON 结果；凭据、请求中的 Token 和推送密钥会脱敏。普通后台部署的 `EXIT_WHEN_FAIL=off` 不会让云函数把签到失败上报为成功。

接口和启动要求依据：[华为云定制运行时官方文档](https://support.huaweicloud.com/intl/zh-cn/devg-functiongraph/functiongraph_01_0406.html)。此仓库的自动检查验证了部署包结构、可执行权限、离线启动和模拟运行时协议；实际账号签到、推送送达以及具体云端环境仍需部署后测试。
