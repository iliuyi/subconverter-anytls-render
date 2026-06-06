# Render 部署说明

本文档用于把 `subconverter-anytls-render` 部署到 Render Web Service。

仓库地址：

```text
https://github.com/iliuyi/subconverter-anytls-render
```

## 1. 部署目标

通过 Render 从 GitHub 仓库自动构建 Docker 镜像并运行服务。

部署完成后，Render 会提供一个公网地址，例如：

```text
https://subconverter-anytls-render.onrender.com
```

服务接口保持 subconverter 原接口：

```text
/version
/sub
```

## 2. 本镜像新增能力

本版本新增 Surge 配置中 `anytls` 节点的解析能力。

支持输入示例：

```text
Hong Kong 01 = anytls, example.com, 15026, password=xxx, tfo=true, sni=example.com, server-cert-fingerprint-sha256=<sha256>
```

新增解析字段：

```text
password
sni
server-cert-fingerprint-sha256
fingerprint
skip-cert-verify
tls13
udp-relay
tfo
```

## 3. Render 部署方式一：使用 Blueprint

仓库根目录已经包含：

```text
render.yaml
Dockerfile
```

Render 可以通过 `render.yaml` 创建服务。

步骤：

1. 打开 Render 控制台。
2. 点击 `New +`。
3. 选择 `Blueprint`。
4. 连接 GitHub 仓库：

   ```text
   iliuyi/subconverter-anytls-render
   ```

5. Render 会读取仓库根目录的 `render.yaml`。
6. 确认服务名称：

   ```text
   subconverter-anytls-render
   ```

7. 点击创建。
8. 等待 Render 自动执行 Docker build 和部署。

`render.yaml` 当前配置：

```yaml
services:
  - type: web
    name: subconverter-anytls-render
    runtime: docker
    plan: free
    autoDeploy: true
    envVars:
      - key: PORT
        value: 25500
```

## 4. Render 部署方式二：手动创建 Web Service

如果不使用 Blueprint，也可以手动创建。

步骤：

1. 打开 Render 控制台。
2. 点击 `New +`。
3. 选择 `Web Service`。
4. 选择：

   ```text
   Build and deploy from a Git repository
   ```

5. 连接 GitHub 账号。
6. 选择仓库：

   ```text
   iliuyi/subconverter-anytls-render
   ```

7. 填写服务名称，例如：

   ```text
   subconverter-anytls-render
   ```

8. Runtime 选择：

   ```text
   Docker
   ```

9. Branch 选择：

   ```text
   master
   ```

10. Root Directory 留空。
11. Dockerfile Path 使用默认值：

    ```text
    ./Dockerfile
    ```

12. Instance Type 可先选择 Free。
13. Environment Variables 添加：

    ```text
    PORT=25500
    ```

14. 点击 `Create Web Service`。
15. 等待构建和部署完成。

## 5. 必须注意的环境变量

### PORT

Render Web Service 必须监听一个 HTTP 端口。

本项目默认端口是：

```text
25500
```

所以 Render 中建议设置：

```text
PORT=25500
```

源码也支持读取 `PORT` 环境变量。如果你不设置，Render 通常会提供默认端口 `10000`，程序也能读取并监听它。但为了和 Dockerfile 的 `EXPOSE 25500` 保持一致，建议显式设置 `PORT=25500`。

## 6. Docker 构建参数

仓库根目录的 `Dockerfile` 支持以下 build args。

### UPDATE_RULES

默认值：

```text
0
```

作用：

构建时跳过在线更新规则。

原因：

`scripts/update_rules.py` 会在线拉取规则仓库，网络不稳定时可能导致 Render 构建变慢或卡住。当前镜像已经包含仓库内置规则，部署服务不依赖构建期在线更新。

如果你确实需要构建时更新规则，可以改成：

```text
UPDATE_RULES=1
```

但不建议在 Render 免费实例构建时开启。

### APK_MIRROR

默认值：

```text
https://mirrors.aliyun.com/alpine
```

作用：

指定 Alpine apk 软件源。

如果 Render 构建时访问该镜像源很慢，可以改回 Alpine 官方源：

```text
https://dl-cdn.alpinelinux.org/alpine
```

### THREADS

默认值：

```text
4
```

作用：

控制编译并发数。

如果 Render 免费实例构建时内存不足，可以降低为：

```text
2
```

### SHA

默认值：

```text
render
```

作用：

写入版本后缀。部署后访问 `/version` 可能看到：

```text
subconverter v0.9.9-render backend
```

## 7. 部署完成后的验证

假设 Render 分配的域名是：

```text
https://subconverter-anytls-render.onrender.com
```

### 7.1 验证服务启动

```bash
curl https://subconverter-anytls-render.onrender.com/version
```

期望返回类似：

```text
subconverter v0.9.9-render backend
```

### 7.2 验证 Clash 转换

```bash
curl -G \
  --data-urlencode target=clash \
  --data-urlencode url='https://gist.githubusercontent.com/metricss/c5e1a79c85310da1debf2db2eb2223c8/raw/9e84d58b1f56ccee1c95ff7c80295ac39dbc86e6/test.conf' \
  https://subconverter-anytls-render.onrender.com/sub
```

期望输出中包含：

```yaml
type: anytls
password: ppp
sni: ixigua.com
fingerprint: fac26f65c034829da42d740d23c4a7202475a3834f0ebaecae5f934adbbfd640
```

### 7.3 验证 sing-box 转换

```bash
curl -G \
  --data-urlencode target=singbox \
  --data-urlencode url='https://gist.githubusercontent.com/metricss/c5e1a79c85310da1debf2db2eb2223c8/raw/9e84d58b1f56ccee1c95ff7c80295ac39dbc86e6/test.conf' \
  https://subconverter-anytls-render.onrender.com/sub
```

期望输出中包含：

```json
"type":"anytls"
"password":"ppp"
"server_name":"ixigua.com"
"tcp_fast_open":true
```

## 8. 本地已验证结果

在 NAS Docker 环境中已验证：

```text
target=clash   HTTP 200，输出 5 个 type: anytls 节点
target=singbox HTTP 200，输出 5 个 type:anytls 节点
```

验证输入：

```text
https://gist.githubusercontent.com/metricss/c5e1a79c85310da1debf2db2eb2223c8/raw/9e84d58b1f56ccee1c95ff7c80295ac39dbc86e6/test.conf
```

## 9. 常见问题

### 9.1 Render 构建时间很长

原因通常是 Docker build 期间需要从 GitHub 拉取依赖：

```text
quickjspp
libcron
toml11
```

Dockerfile 已经加入：

```text
timeout 180
http.lowSpeedLimit 1000
http.lowSpeedTime 30
```

这样网络卡住时会失败并重试，而不是无限等待。

### 9.2 Render 构建失败在 update_rules.py

当前默认不会运行 `update_rules.py`。

确认 `UPDATE_RULES` 没有被设置为 `1`。

建议保持：

```text
UPDATE_RULES=0
```

### 9.3 Render 显示端口检测失败

确认 Environment Variables 里有：

```text
PORT=25500
```

如果仍失败，可以改成：

```text
PORT=10000
```

因为程序会读取 `PORT` 环境变量并监听对应端口。

### 9.4 Render Free 实例休眠

Render Free Web Service 可能会在空闲后休眠。

休眠后第一次访问会变慢，这是 Render 免费实例的正常行为。

### 9.5 转换接口返回内容很大

默认配置会带较完整规则，输出可能较大。

这是正常现象，不代表转换失败。

## 10. 更新部署

后续修改代码后：

```bash
git add .
git commit -m "Update subconverter AnyTLS"
git push
```

Render 默认 `autoDeploy: true`，推送到 `master` 后会自动重新构建并部署。

## 11. 快速命令汇总

检查版本：

```bash
curl https://YOUR-SERVICE.onrender.com/version
```

Clash 转换：

```bash
curl -G \
  --data-urlencode target=clash \
  --data-urlencode url='https://gist.githubusercontent.com/metricss/c5e1a79c85310da1debf2db2eb2223c8/raw/9e84d58b1f56ccee1c95ff7c80295ac39dbc86e6/test.conf' \
  https://YOUR-SERVICE.onrender.com/sub
```

sing-box 转换：

```bash
curl -G \
  --data-urlencode target=singbox \
  --data-urlencode url='https://gist.githubusercontent.com/metricss/c5e1a79c85310da1debf2db2eb2223c8/raw/9e84d58b1f56ccee1c95ff7c80295ac39dbc86e6/test.conf' \
  https://YOUR-SERVICE.onrender.com/sub
```
