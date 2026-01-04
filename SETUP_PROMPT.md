# 🐱 TradeCat 快速安装指南

> 复制下面的提示词到 AI 助手，让 AI 一步步带你完成安装

---

## 📋 安装提示词

复制以下内容到 ChatGPT / Claude：

```
你是 TradeCat 安装助手。请一步一步指导我安装，每次只给 1-2 个命令，等我确认后再继续。

## 安装目标
- 系统: Ubuntu 24.04 (WSL2)
- 数据库: TimescaleDB (PostgreSQL 16, 端口 5433)
- 项目: github.com/tukuaiai/tradecat

## 配置信息
- 数据库用户: postgres / postgres
- 数据库名: market_data
- 项目路径: ~/.projects/tradecat

## 安装步骤
1. WSL2 + Ubuntu 24.04
2. 系统依赖 (build-essential, python3-dev 等)
3. TimescaleDB 2.x
4. TA-Lib 系统库
5. 克隆项目 + ./scripts/init.sh
6. 配置 .env (Bot Token)
7. 启动服务

## 规则
- 用中文回复
- 命令用代码块
- 遇到错误帮我分析
- 重要提醒用 ⚠️

现在开始，先问我：
1. 用的是 Windows 还是已有 Linux？
2. 是否已安装 WSL2？
```

---

## 🚀 手动安装 (5分钟)

如果你熟悉 Linux，直接执行：

### 1️⃣ 安装依赖

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y build-essential python3-dev python3-pip python3-venv git curl wget

# TimescaleDB
echo "deb https://packagecloud.io/timescale/timescaledb/ubuntu/ $(lsb_release -c -s) main" | sudo tee /etc/apt/sources.list.d/timescaledb.list
wget --quiet -O - https://packagecloud.io/timescale/timescaledb/gpgkey | sudo apt-key add -
sudo apt update && sudo apt install -y timescaledb-2-postgresql-16
sudo timescaledb-tune --quiet --yes
sudo systemctl restart postgresql

# TA-Lib
cd /tmp && wget http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz
tar -xzf ta-lib-0.4.0-src.tar.gz && cd ta-lib
./configure --prefix=/usr && make -j$(nproc) && sudo make install && sudo ldconfig
```

### 2️⃣ 创建数据库

```bash
sudo -u postgres psql -c "CREATE DATABASE market_data;"
sudo -u postgres psql -d market_data -c "CREATE EXTENSION IF NOT EXISTS timescaledb;"
```

### 3️⃣ 部署项目

```bash
mkdir -p ~/.projects && cd ~/.projects
git clone https://github.com/tukuaiai/tradecat.git && cd tradecat
./scripts/init.sh
```

### 4️⃣ 配置 Bot Token

```bash
# 编辑配置
vim services/telegram-service/config/.env

# 填入你的 Bot Token
TELEGRAM_BOT_TOKEN=你的Token
```

### 5️⃣ 启动

```bash
./scripts/start.sh daemon
./scripts/start.sh status
```

---

## 📥 导入历史数据 (可选)

从 [HuggingFace](https://huggingface.co/datasets/123olp/binance-futures-ohlcv-2018-2026) 下载数据后：

```bash
cd backups/timescaledb
zstd -d candles_1m.bin.zst -c | psql -d market_data -c "COPY market_data.candles_1m FROM STDIN WITH (FORMAT binary)"
```

---

## ❓ 常见问题

| 问题 | 解决 |
|:---|:---|
| TimescaleDB 连接失败 | `sudo systemctl status postgresql` 检查状态 |
| TA-Lib 编译失败 | 先 `sudo apt install build-essential` |
| Bot 无法连接 | 配置代理 `HTTPS_PROXY=http://127.0.0.1:7890` |
| pip 安装慢 | 换源 `pip install -i https://pypi.tuna.tsinghua.edu.cn/simple` |

---

## 📞 获取帮助

- Telegram 群: [@glue_coding](https://t.me/glue_coding)
- 频道: [@tradecat_ai_channel](https://t.me/tradecat_ai_channel)
