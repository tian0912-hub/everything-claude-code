---
description: Configure Cangjie environment and version. Use before other cangjie commands when multiple versions are installed.
---

# Cangjie Environment Setup

Configure the Cangjie development environment when multiple versions are installed.

## Quick Setup

在项目根目录创建 `.cangjie-env` 文件：

```bash
# 项目根目录执行
echo 'CANGJIE_ENV_SCRIPT=/Users/tianbin/root/bin/i-cangjie' > .cangjie-env
```

或者设置环境变量：

```bash
export CANGJIE_ENV_SCRIPT=/Users/tianbin/root/bin/i-cangjie
```

## Configuration Priority

命令按以下优先级查找环境配置：

1. **环境变量** `CANGJIE_ENV_SCRIPT`
2. **项目配置文件** `.cangjie-env` (项目根目录)
3. **默认路径** `/Users/tianbin/root/bin/i-cangjie`

## Usage

```bash
# 查看当前配置
/cangjie-env

# 验证环境是否正常
/cangjie-env --verify
```

## How to Configure

### 方法 1: 项目配置文件 (推荐)

在项目根目录创建 `.cangjie-env`：

```bash
# .cangjie-env
CANGJIE_ENV_SCRIPT=/Users/tianbin/root/bin/i-cangjie
```

这样每个项目可以使用不同版本的 Cangjie。

### 方法 2: 环境变量

在 `~/.zshrc` 或 `~/.bashrc` 中添加：

```bash
export CANGJIE_ENV_SCRIPT=/Users/tianbin/root/bin/i-cangjie
```

### 方法 3: 直接在 shell 中设置

```bash
# 每次会话前执行
. /Users/tianbin/root/bin/i-cangjie
```

## Example Session

````text
User: /cangjie-env

Agent:
# Cangjie Environment

## Configuration Check

Reading from (in order):
1. Environment variable CANGJIE_ENV_SCRIPT: not set
2. Project file .cangjie-env: found
   `CANGJIE_ENV_SCRIPT=/Users/tianbin/root/bin/i-cangjie`

## Sourcing Environment

```bash
$ . /Users/tianbin/root/bin/i-cangjie
Cangjie environment configured (version: 0.53.4)
```

## Verification

```bash
$ cjpm --version
cjpm 0.53.4
```

## Current Status

| Setting | Value |
|---------|-------|
| Config Source | .cangjie-env |
| Script Path | /Users/tianbin/root/bin/i-cangjie |
| Cangjie Version | 0.53.4 |

✅ Environment ready!
````

## Creating .cangjie-env File

```bash
# 在项目根目录执行
cat > .cangjie-env << 'EOF'
# Cangjie environment configuration
# Path to version selector script
CANGJIE_ENV_SCRIPT=/Users/tianbin/root/bin/i-cangjie
EOF
```

## How Other Commands Use This

All `/cangjie-*` commands will:

1. Read `CANGJIE_ENV_SCRIPT` from environment or `.cangjie-env` file
2. Source the script before running cjpm commands

```bash
# 自动执行的逻辑
if [ -f ".cangjie-env" ]; then
    source .cangjie-env
fi
. "${CANGJIE_ENV_SCRIPT:-/Users/tianbin/root/bin/i-cangjie}"
cjpm build
```

## Troubleshooting

### Script Not Found

```
❌ Cangjie environment script not found

Create .cangjie-env file in project root:
  echo 'CANGJIE_ENV_SCRIPT=/path/to/i-cangjie' > .cangjie-env

Or set environment variable:
  export CANGJIE_ENV_SCRIPT=/path/to/i-cangjie
```

### cjpm Not Available

```
❌ cjpm command not available after sourcing

Check your i-cangjie script adds cjpm to PATH
```

## Related Commands

- `/cangjie-build` - Build Cangjie projects
- `/cangjie-test` - Run Cangjie tests  
- `/cangjie-review` - Review Cangjie code