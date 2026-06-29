# talih-ota

查询 TALIH OTA 更新、归档历史版本，并整理可下载的 ROM 包链接。

## 支持产品

默认产品路径为 `xpad2`。当前支持的 TALIH 设备 profile 系列：

```text
talih-pd
talih-pd2
talih-pd2a
talih-pd3p
talih-pd3s
talih-pd3u
```

运行 `talih-ota devices` 可以查看本机配置里的实际设备列表。

## 安装 Release

macOS Apple Silicon：

```bash
curl -L https://github.com/yoyicue/talih-ota/releases/latest/download/talih-ota-aarch64-apple-darwin.tar.gz -o talih-ota.tar.gz
tar -xzf talih-ota.tar.gz
sudo install -m 0755 talih-ota /usr/local/bin/talih-ota
talih-ota --version
```

macOS Intel：

```bash
curl -L https://github.com/yoyicue/talih-ota/releases/latest/download/talih-ota-x86_64-apple-darwin.tar.gz -o talih-ota.tar.gz
tar -xzf talih-ota.tar.gz
sudo install -m 0755 talih-ota /usr/local/bin/talih-ota
talih-ota --version
```

Linux x86_64：

```bash
curl -L https://github.com/yoyicue/talih-ota/releases/latest/download/talih-ota-x86_64-unknown-linux-gnu.tar.gz -o talih-ota.tar.gz
tar -xzf talih-ota.tar.gz
sudo install -m 0755 talih-ota /usr/local/bin/talih-ota
talih-ota --version
```

Linux ARM64：

```bash
curl -L https://github.com/yoyicue/talih-ota/releases/latest/download/talih-ota-aarch64-unknown-linux-gnu.tar.gz -o talih-ota.tar.gz
tar -xzf talih-ota.tar.gz
sudo install -m 0755 talih-ota /usr/local/bin/talih-ota
talih-ota --version
```

Windows x86_64 PowerShell：

```powershell
Invoke-WebRequest https://github.com/yoyicue/talih-ota/releases/latest/download/talih-ota-x86_64-pc-windows-gnullvm.zip -OutFile talih-ota.zip
Expand-Archive .\talih-ota.zip -DestinationPath .
.\talih-ota.exe --version
```

Windows ARM64 PowerShell：

```powershell
Invoke-WebRequest https://github.com/yoyicue/talih-ota/releases/latest/download/talih-ota-aarch64-pc-windows-gnullvm.zip -OutFile talih-ota.zip
Expand-Archive .\talih-ota.zip -DestinationPath .
.\talih-ota.exe --version
```

## 快速使用

如果本地配置里已经设置了 `default_device`，常用命令不需要再写 `--device`。

默认配置文件：

```text
macOS / Linux: ~/.config/talih-ota/config.toml
Windows:       .\talih-ota.toml
```

Release 包不包含本地配置。第一次使用前创建配置文件：

```bash
mkdir -p ~/.config/talih-ota
$EDITOR ~/.config/talih-ota/config.toml
```

Windows PowerShell：

```powershell
notepad .\talih-ota.toml
```

Windows 的 `.\talih-ota.toml` 指运行命令时所在目录。配置放在其他位置时，使用 `--config <path>` 指定。

最小配置格式：

```toml
tal_secret_key = "<TAL_SECRET_KEY>"
tal_genie_sk = "<TAL_GENIE_SK>"
default_device = "talih-pd2"

[devices.talih-pd2]
sn = "<device-sn>"
product = "xpad2"
default_local_version = "V260423"
history_local_version = "V260523"
```

`default_local_version` 用于 `current`；`history_local_version` 用于 `history`。如果不想写设备 profile，也可以临时使用 `talih-ota --device-sn <sn> current --product xpad2`。

### 1. 看当前可用设备

```bash
talih-ota devices
```

### 2. 下载当前可升级 ROM

```bash
talih-ota current -o current_ota.json >/dev/null &&
jq -r '.package_url // empty' current_ota.json |
while IFS= read -r url; do
  [ -n "$url" ] && curl -L "$url" -o latest_ota.zip
done
```

如果没有生成 `latest_ota.zip`，表示这次查询没有返回可下载包。

### 3. 下载历史 ROM

```bash
talih-ota history -o ota_archive.json
talih-ota links -i ota_archive.json -o SUMMARY.md >/dev/null
```

从历史包里挑一个版本下载：

```bash
jq -r '.unique_packages[] | "\(.to_version // "?") \(.package_url)"' ota_archive.json
```

或者批量下载所有历史包：

```bash
mkdir -p roms
jq -r '.unique_packages[].package_url' ota_archive.json |
while IFS= read -r url; do
  name="$(basename "${url%%\?*}")"
  curl -L "$url" -o "roms/$name"
done
```

## 常用补充

临时切换设备：

```bash
talih-ota --device <name> current
talih-ota --device <name> history -o ota_archive.json
```

不用设备 profile，直接用 SN：

```bash
talih-ota --device-sn <sn> current --product xpad2
```

指定本地版本探测：

```bash
talih-ota current V260423
```

只快速归档普通升级包：

```bash
talih-ota history --no-init -o ota_archive.json
```

更多参数：

```bash
talih-ota --help
talih-ota <command> --help
```
