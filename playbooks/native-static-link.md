# 原生后端的静态链接

## 什么时候用

产物要以单个二进制分发到不同发行版、又不想绑定宿主 glibc 版本时。
`moon` 没有 static 开关，但原生后端最后一步是普通 C 链接，可以从外部接管。

## 先排除的错路（都实测无效）

- `moon build --help`：没有任何 static / linker 相关选项
- `CC="cc -static"`：被忽略，产物照旧动态链接
- `moon.pkg` 里 `link = { ... }`：`Unexpected key 'link' found in moon.pkg.`
- `MOON_CC="cc -static"`：`failed to find executable `cc -static``——它把整个字符串当可执行文件路径，不解析参数

## 做法：包装脚本当 MOON_CC

```bash
mkdir -p /tmp/cc-static
printf '#!/bin/sh\nexec cc -static "$@"\n' > /tmp/cc-static/cc
chmod +x /tmp/cc-static/cc
ln -sf "$(command -v ar)" /tmp/cc-static/ar     # 必须：moon 从 cc 所在目录推导归档器
MOON_CC=/tmp/cc-static/cc moon build --release
```

- `ar` 软链不能省：只给 `cc` 会报 `failed to resolve native archiver executable`
- 编译与链接共用这个 `cc`，编译阶段带 `-static` 无害

## 验收

```bash
file _build/native/release/build/cmd/main/main.exe   # 期望 … statically linked …
ldd  _build/native/release/build/cmd/main/main.exe   # 期望「不是动态可执行文件」
```

## 代价与限制

- 体积：hello world 476KB → 1.17MB；带 C stub 的真实项目静态后 1.28MB
- 是 glibc 静态链接而非 musl：用到 NSS 的调用（`getpwnam` 之类）会失效或告警；
  只用 socket / 文件 / D-Bus 的程序不受影响
- 复测版本 moon `0.1.20260923`：真实项目产物静态链接并实跑通过（连上会话总线，名字冲突按预期退出）
