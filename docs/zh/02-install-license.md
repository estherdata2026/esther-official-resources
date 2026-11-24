# 2. 安装与授权


### 2.1 部署四步（Windows 预览版）

1. 从 [GitHub Releases](https://github.com/estherdata2026/esther-official-resources/releases) 或 [Gitee 发行版](https://gitee.com/esther2026/esther-official-resources/releases) 下载 zip 压缩包，解压到目标服务器任意目录（绿色免安装）。
2. **领取试用授权**（见 [2.2](#22-授权license)）：访问官网 [shumai-data.com](https://shumai-data.com) 注册并登录控制台，领取全模块试用券（试用时长以券面为准），在「License 管理」下载授权文件 `esther.lic`，放入程序目录。
3. 双击 `start_esther.bat`（或命令行运行）启动服务。
4. 浏览器打开 `http://<服务器IP>:8000`，使用管理员账号登录即可使用。

> 未完成授权时启动服务会直接退出，并提示获取机器指纹、联系厂商申请授权——这是预期行为，完成 [2.2](#22-授权license) 的申请流程后再启动即可。

**修改端口**：设置环境变量 `ESTHER_PORT` 后重启服务。

### 2.2 授权（License）

- **启动需要有效授权**：首次部署需申请授权文件 `esther.lic`；未检测到有效授权时服务不会启动。
- **领取试用授权**：新用户注册即领全模块试用券（试用时长以券面为准）：
  1. 访问官网 [shumai-data.com](https://shumai-data.com) 注册账号，登录控制台自动领取试用券；
  2. 命令行执行 `esther-cli.exe license fingerprint` 获取本机指纹（64 位），在控制台「License 管理」提交指纹并下载授权文件 `esther.lic`；
  3. 将 `esther.lic` 放到程序目录，运行 `start_esther.bat` 即可正常启动。
- **查看授权状态**：`esther-cli.exe license status`（授权模式 / 有效期 / 剩余天数）。
- **到期与换机**：授权到期后服务停止启动，续费后立即恢复；授权与机器指纹绑定，换服务器 / 换机器可在控制台「License 管理」自助重置绑定（每自然月 2 次）。

### 2.3 运行数据与账号

- 运行时数据自动创建在程序目录的 `data/` 下（SQLite + Kuzu 图数据库），无需额外安装数据库。
- 首次启动会自动创建管理员账号 **admin**（初始密码 **admin123**），请登录后妥善保管。

### 2.4 安全提示

- 程序为 Nuitka 原生编译产物，部分杀毒软件可能误报，请加入白名单；依赖已静态链接，无需安装 VC++ 运行库。
- `POST /api/analyze` 接口当前未启用鉴权，请将服务部署在**内网**环境中使用；`/api/v1/*` 系列接口需要登录后携带 Token 访问。

---
