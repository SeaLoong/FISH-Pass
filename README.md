[English](README.en.md) | 简体中文

# 🐟 FISH!Pass

> **FISH!**（VRChat Fish World）代理工具 — 通过 [Fiddler Classic](https://www.telerik.com/fiddler/fiddler-classic) 拦截并覆盖加密角色数据

### 工作原理

Fish World 启动时会从远程 URL 获取加密的玩家角色数据：

| 来源               | URL                                                       |
| ------------------ | --------------------------------------------------------- |
| **可信**（主要）   | `https://gamerexde.github.io/trickforge-public/roles.txt` |
| **不可信**（备用） | `https://api.trickforgestudios.com/api/v1/roles/vrc/all`  |

数据采用 AES-256-CBC 加密（密钥通过 Udon IL 字节码中的自定义派生算法生成）。**FISH!Pass** 让你生成包含自定义玩家条目的加密 `roles.txt`，并通过 Fiddler Classic 的 AutoResponder 功能在本地拦截请求并返回自定义数据 —— 游戏将加载你的数据。

加密协议的技术细节请参阅 [Fish-Udon-Raw-Ilcode](https://github.com/SeaLoong/Fish-Udon-Raw-Ilcode)。

---

### 快速开始

#### 1. 生成 `roles.txt`

使用**在线工具**：**[FISH!Pass 生成器](https://sealoong.github.io/FISH-Pass/)**（或在本地打开 `index.html`）

1. 输入 VRChat 显示名称（每行一个）
2. 选择角色标志 — **P**（赞助者）、**S**（支持者）、**N**（Nitro）— 默认全选
3. 点击 **生成加密数据**
4. 点击 **下载 roles.txt** 保存文件

> 所有加密操作均在浏览器本地完成，不会向任何服务器发送数据。

#### 2. 安装并启动 Fiddler Classic

1. 下载并安装 [Fiddler Classic](https://www.telerik.com/fiddler/fiddler-classic)（免费，仅限 Windows）
2. 启动 Fiddler Classic

#### 3. 启用 HTTPS 解密

由于需要拦截的两个 URL 均为 **HTTPS**，需要开启 Fiddler 的 HTTPS 解密功能：

1. 打开菜单 **Tools → Options**
2. 切换到 **HTTPS** 选项卡
3. 勾选 **Capture HTTPS CONNECTs**
4. 勾选 **Decrypt HTTPS traffic**
5. 在弹出的对话框中点击 **Yes** 安装 Fiddler 根证书到系统信任存储
6. 点击 **OK** 保存设置

> Fiddler 会自动生成并安装自签名 CA 证书。关闭 Fiddler 后可在 **Tools → Options → HTTPS → Actions → Remove Interception Certificates** 移除证书。

#### 4. 配置 AutoResponder 规则

AutoResponder 是 Fiddler 的核心功能，用于匹配请求 URL 并返回自定义响应：

1. 切换到右侧面板的 **AutoResponder** 选项卡
2. 勾选 **Enable rules**（启用规则）
3. 勾选 **Unmatched requests passthrough**（未匹配的请求正常放行）
4. 点击 **Add Rule** 添加以下两条规则：

**规则一 — 拦截可信 URL：**

| 字段   | 值                                                                   |
| ------ | -------------------------------------------------------------------- |
| Match  | `EXACT:https://gamerexde.github.io/trickforge-public/roles.txt`      |
| Action | 选择你生成的 `roles.txt` 文件的完整路径（如 `C:\path\to\roles.txt`） |

**规则二 — 拦截不可信 URL：**

| 字段   | 值                                                             |
| ------ | -------------------------------------------------------------- |
| Match  | `EXACT:https://api.trickforgestudios.com/api/v1/roles/vrc/all` |
| Action | 选择同一个 `roles.txt` 文件路径                                |

> **提示**：在 Action 下拉框中选择 **Find a file...**，然后浏览选择你的 `roles.txt` 文件。

5. 点击 **Save** 保存规则

#### 5. 上游代理 / 网关设置（可选）

如果你本身已在使用代理（如 Clash、v2ray 等），需要配置 Fiddler 的网关让流量通过上游代理：

1. 打开菜单 **Tools → Options**
2. 切换到 **Gateway** 选项卡
3. 选择 **Manual Proxy Configuration**
4. 输入上游代理地址，例如：`http=127.0.0.1:7890;https=127.0.0.1:7890`
5. 点击 **OK** 保存

如果不使用上游代理，保持默认的 **Use System Proxy (recommended)** 即可。

#### 6. 运行

1. 确保 Fiddler Classic 正在运行 — 左下角状态栏显示 **Capturing**（如未显示，按 `F12` 或点击 **File → Capture Traffic** 开启抓包）
2. 确认 AutoResponder 规则已启用（**Enable rules** 已勾选）
3. 启动 VRChat 进入 Fish World — 游戏将加载你的自定义角色数据

> 可以在 Fiddler 左侧会话列表中观察到被拦截的请求，图标会显示为本地响应（灰色箭头图标）。

#### 7. 停止

关闭 Fiddler Classic — 系统代理设置会自动恢复。

> 也可以在不关闭 Fiddler 的情况下按 `F12` 停止抓包，或取消勾选 **AutoResponder** 中的 **Enable rules** 来停止拦截。

---

### 数据格式

解密后的 `roles.txt` 内容：

```json
{
  "v": "1",
  "players": {
    "玩家名1": "p,s,n",
    "玩家名2": "p",
    "玩家名3": ""
  }
}
```

| 标志     | 含义                |
| -------- | ------------------- |
| `p`      | 赞助者（Patron）    |
| `s`      | 支持者（Supporter） |
| `n`      | Nitro 增强          |
| _（空）_ | 无特殊角色          |

---

### 相关项目

| 项目                                                                     | 说明                                           |
| ------------------------------------------------------------------------ | ---------------------------------------------- |
| [Fiddler Classic](https://www.telerik.com/fiddler/fiddler-classic)       | 免费的 Windows HTTP/HTTPS 调试代理工具         |
| [Fish-Udon-Raw-Ilcode](https://github.com/SeaLoong/Fish-Udon-Raw-Ilcode) | Fish World 全部 Udon IL 字节码反编译及逆向分析 |

---

### 许可证

[MIT](LICENSE)
