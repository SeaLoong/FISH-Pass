English | [简体中文](README.md)

# 🐟 FISH!Pass

> Proxy tool for **FISH!** (VRChat Fish World) — intercept and override encrypted role data via [Fiddler Classic](https://www.telerik.com/fiddler/fiddler-classic)

### How It Works

Fish World fetches encrypted player role data from remote URLs on startup:

| Source                   | URL                                                       |
| ------------------------ | --------------------------------------------------------- |
| **Trusted** (primary)    | `https://gamerexde.github.io/trickforge-public/roles.txt` |
| **Untrusted** (fallback) | `https://api.trickforgestudios.com/api/v1/roles/vrc/all`  |

The data is AES-256-CBC encrypted (custom key derivation from Udon IL bytecode). **FISH!Pass** lets you generate your own encrypted `roles.txt` with custom player entries, and use Fiddler Classic's AutoResponder to intercept requests and serve your local data — the game loads your data instead.

For technical details on the encryption protocol, see [Fish-Udon-Raw-Ilcode](https://github.com/SeaLoong/Fish-Udon-Raw-Ilcode).

---

### Quick Start

#### 1. Generate `roles.txt`

Use the **online tool**: **[FISH!Pass Generator](https://sealoong.github.io/FISH-Pass/)** (or open `index.html` locally)

1. Enter VRChat display names (one per line)
2. Select role flags — **P** (Patron), **S** (Supporter), **N** (Nitro) — all selected by default
3. Click **Generate Encrypted Data**
4. Click **Download roles.txt** to save the file

> All encryption runs in your browser. No data is sent to any server.

#### 2. Install & Launch Fiddler Classic

1. Download and install [Fiddler Classic](https://www.telerik.com/fiddler/fiddler-classic) (free, Windows only)
2. Launch Fiddler Classic

#### 3. Enable HTTPS Decryption

Both intercepted URLs are **HTTPS**, so Fiddler needs to decrypt HTTPS traffic:

1. Go to **Tools → Options**
2. Switch to the **HTTPS** tab
3. Check **Capture HTTPS CONNECTs**
4. Check **Decrypt HTTPS traffic**
5. Click **Yes** in the prompt to install Fiddler's root certificate into the system trust store
6. Click **OK** to save

> Fiddler auto-generates and installs a self-signed CA certificate. To remove it later: **Tools → Options → HTTPS → Actions → Remove Interception Certificates**.

#### 4. Configure AutoResponder Rules

AutoResponder is Fiddler's core feature for matching request URLs and returning custom responses:

1. Switch to the **AutoResponder** tab in the right panel
2. Check **Enable rules**
3. Check **Unmatched requests passthrough**
4. Click **Add Rule** and add the following two rules:

**Rule 1 — Intercept the trusted URL:**

| Field  | Value                                                                 |
| ------ | --------------------------------------------------------------------- |
| Match  | `EXACT:https://gamerexde.github.io/trickforge-public/roles.txt`       |
| Action | Full path to your generated `roles.txt` (e.g. `C:\path\to\roles.txt`) |

**Rule 2 — Intercept the untrusted URL:**

| Field  | Value                                                          |
| ------ | -------------------------------------------------------------- |
| Match  | `EXACT:https://api.trickforgestudios.com/api/v1/roles/vrc/all` |
| Action | Same `roles.txt` file path                                     |

> **Tip**: In the Action dropdown, select **Find a file...** and browse to your `roles.txt`.

5. Click **Save** to save the rules

#### 5. Gateway / Upstream Proxy (Optional)

If you're already using a proxy (e.g., Clash, v2ray), configure Fiddler's gateway to route traffic through it:

1. Go to **Tools → Options**
2. Switch to the **Gateway** tab
3. Select **Manual Proxy Configuration**
4. Enter your upstream proxy address, e.g.: `http=127.0.0.1:7890;https=127.0.0.1:7890`
5. Click **OK** to save

If you don't use an upstream proxy, keep the default **Use System Proxy (recommended)**.

#### 6. Run

1. Make sure Fiddler Classic is running — the status bar at the bottom-left should show **Capturing** (if not, press `F12` or click **File → Capture Traffic**)
2. Verify AutoResponder rules are enabled (**Enable rules** is checked)
3. Launch VRChat and enter Fish World — the game will load your custom role data

> You can observe intercepted requests in Fiddler's session list on the left — they will show a local response icon (grey arrow).

#### 7. Stop

Close Fiddler Classic — system proxy settings are automatically restored.

> You can also press `F12` to stop capturing without closing Fiddler, or uncheck **Enable rules** in AutoResponder to stop interception.

---

### Data Format

The decrypted `roles.txt` contains:

```json
{
  "v": "1",
  "players": {
    "PlayerName1": "p,s,n",
    "PlayerName2": "p",
    "PlayerName3": ""
  }
}
```

| Flag      | Meaning          |
| --------- | ---------------- |
| `p`       | Patron           |
| `s`       | Supporter        |
| `n`       | Nitro            |
| _(empty)_ | No special roles |

---

### Related Projects

| Project                                                                  | Description                                                              |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| [Fiddler Classic](https://www.telerik.com/fiddler/fiddler-classic)       | Free HTTP/HTTPS debugging proxy tool for Windows                         |
| [Fish-Udon-Raw-Ilcode](https://github.com/SeaLoong/Fish-Udon-Raw-Ilcode) | Decompiled Udon IL bytecode & reverse-engineering analysis of Fish World |

---

### License

[MIT](LICENSE)
