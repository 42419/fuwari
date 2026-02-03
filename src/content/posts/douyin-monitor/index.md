---
title: 抖音视频更新监控
published: 2025-02-03
description: 从零开始，完整部署抖音用户视频更新监控系统
image: "./cover.jpeg"
tags: [Linux, 抖音, 监控, 自动化]
category: 指南
draft: false
---

# 📚 抖音用户视频更新监控系统

✅ 防置顶干扰 | ✅ 钉钉群推送 | ✅ 自动清理日志 | ✅ systemd 托管

适用于 `Ubuntu/CentOS/Debian` 等主流 Linux 发行版。

## 🧰 一、准备工作

### 1. 准备事项

- 一台搭载 Linux 系统的云服务器，并且`部署好 Evil0ctal大佬的API`
  ::github{repo="Evil0ctal/Douyin_TikTok_Download_API"}
- 一个钉钉群机器人账号并获取到 Webhook 地址

```bash
# 安装依赖（Ubuntu/Debian）
sudo apt update && sudo apt install -y curl jq

# CentOS/RHEL
sudo yum install -y curl jq
```

## 📁 二、创建项目目录与脚本

```bash
# 创建项目目录
sudo mkdir -p /opt/douyin_monitor/log
sudo mkdir -p /opt/douyin_monitor/state

# 创建用户配置文件
sudo tee /opt/douyin_monitor/users.conf << 'EOF'
# 格式：sec_user_id|昵称
MS4wLjABAAAAL6z-C5MVaIclPFu9sugIIc3-NlH6v0V4uPu_XXzxxxx|xxx
MS4wLjABAAAA4MjTvxSsNOjHfi9kfyRdu0KMKxxxxxxxxxxxW0OKY|xxxxx
MS4wLjABAAAAbMuesG2mD_6W7wIdEEnsEA3x5oRtS7aNXoxxxxxxxxxxtcyBwyxTxt8gu_z|测试
EOF
```

🔁 每行一个用户：`sec_user_id|昵称`，支持注释 `#`

## 📜 三、部署主监控脚本

```bash title="douyin_monitor.sh" del={41} ins={42}
sudo tee /opt/douyin_monitor/douyin_monitor.sh << 'EOF'
#!/bin/bash
# ========================================
# 抖音多用户视频更新监听脚本（防置顶 + 钉钉群机器人推送）
# 支持 .env 安全配置 | 使用钉钉群机器人推送通知
# ========================================

# =================== 路径配置 ===================
WORK_DIR="/opt/douyin_monitor"
USERS_CONF="$WORK_DIR/users.conf"
STATE_DIR="$WORK_DIR/state"
LOG_FILE="$WORK_DIR/log/monitor.log"
ENV_FILE="$WORK_DIR/.env"

# API 配置
API_URL="http://你的服务器IP/api/douyin/web/fetch_user_post_videos"
COUNT=10

# 创建必要目录
mkdir -p "$STATE_DIR" "$WORK_DIR/log"

# =================== 加载环境变量 ===================
if [ -f "$ENV_FILE" ]; then
    source "$ENV_FILE"
else
    echo "❌ 错误：环境变量文件未找到 $ENV_FILE" >&2
    exit 1
fi

# =================== 日志函数 ===================
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# =================== 推送通知（钉钉群机器人）===================
send_notification() {
    local title="$1"
    local content="$2"
    local video_url="$3"
    local cover_url="$4"
    local webhook_url="https://oapi.dingtalk.com/robot/send?access_token=YOUR_ACCESS_TOKEN"
    # local webhook_url="https://oapi.dingtalk.com/robot/send?access_token=YOUR_ACCESS_TOKEN" 替换为你自己的群机器人webhook地址

    # 构造整体跳转 ActionCard 消息格式
    local json_data=$(cat <<EOF
{
    "msgtype": "actionCard",
    "actionCard": {
        "title": "$title",
        "text": "$content",
        "singleTitle": "点击观看",
        "singleURL": "$video_url"
    }
}
EOF
)

    # 推送通知
    curl -s -X POST "$webhook_url" \
        -H "Content-Type: application/json" \
        -d "$json_data" \
        >> "$LOG_FILE" 2>&1 &

    log "📤 推送通知至钉钉群机器人"
}

# =================== 检查用户更新 ===================
check_user() {
    local sec_user_id="$1"
    local nickname="$2"

    log "🔍 检查用户: $nickname (sec_user_id: $sec_user_id)"

    local response
    response=$(curl -s -G \
        --data-urlencode "sec_user_id=$sec_user_id" \
        --data-urlencode "count=$COUNT" \
        "$API_URL")

    # 检查接口返回
    if ! echo "$response" | jq -e '.code == 200' >/dev/null; then
        local msg=$(echo "$response" | jq -r '.msg // "未知错误"')
        log "❌ 请求失败: $msg"
        return 1
    fi

    # 提取非置顶最新视频（按时间倒序）
    local latest=$(echo "$response" | jq -r '
        .data.aweme_list[]?
        | select(.is_top != 1)
        | .create_time |= tonumber
        | .
    ' | jq -s 'sort_by(-.create_time) | .[0] // empty')

    [ -z "$latest" ] && {
        log "⚠️ 未获取到有效视频数据"
        return 1
    }

    local new_id=$(echo "$latest" | jq -r '.aweme_id')
    local title=$(echo "$latest" | jq -r '.item_title // .desc // "无标题"')
    local create_time=$(echo "$latest" | jq -r '.create_time')
    local time_str=$(date -d "@$create_time" "+%Y-%m-%d %H:%M:%S")
    local cover_url=$(echo "$latest" | jq -r '.cover_original_scale.url_list[0] // ""')
    local state_file="$STATE_DIR/${sec_user_id}.txt"
    local old_id=$(cat "$state_file" 2>/dev/null || echo "")

    if [ -z "$old_id" ]; then
        log "🆕 首次记录: $new_id"
        echo "$new_id" > "$state_file"
    elif [ "$new_id" != "$old_id" ]; then
        log "🎉 检测到新视频: $new_id"

        # 构造 ActionCard 消息内容
        local message_content="### 新视频发布

用户：$nickname

标题：$title

时间：$time_str

检测时间：$(date '+%Y-%m-%d %H:%M:%S')"

        # 如果有封面图，添加到最下面
        if [ -n "$cover_url" ]; then
            message_content="$message_content

![视频封面]($cover_url)"
        fi

        send_notification "$nickname 发布新视频" "$message_content" "https://www.douyin.com/video/$new_id" "$cover_url"
        echo "$new_id" > "$state_file"
    else
        log "✅ 无更新"
    fi
}

# =================== 主函数 ===================
main() {
    log "✅ 抖音监控服务已启动（使用钉钉群机器人推送）"

    while true; do
        while IFS='|' read -r sec_user_id nickname || [ -n "$sec_user_id" ]; do
            # 跳过空行或注释
            [[ -z "$sec_user_id" || "$sec_user_id" =~ ^[[:space:]]*# ]] && continue

            # 清理空白字符
            sec_user_id=$(echo "$sec_user_id" | xargs)
            nickname=$(echo "$nickname" | xargs)

            check_user "$sec_user_id" "$nickname"

            # 随机延迟 3~8 秒，避免请求过快
            sleep $((3 + RANDOM % 6))
        done < "$USERS_CONF"

        # 记录本轮完成并显示等待时间（15~40秒）
        local wait_time=$((15 + RANDOM % 26))
        log "💤 本轮检测完成，等待 $wait_time 秒后重新检测..."
        sleep $wait_time
    done
}

# 启动主程序
main
EOF
```

### 设置可执行权限

```bash
sudo chmod +x /opt/douyin_monitor/douyin_monitor.sh
```

## ⚙️ 四、配置 systemd 服务（开机自启）

```bash
sudo tee /etc/systemd/system/douyin-monitor.service << 'EOF'
[Unit]
Description=抖音用户视频更新监控服务
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/douyin_monitor
ExecStart=/bin/bash /opt/douyin_monitor/douyin_monitor.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

### 启用服务

```bash
# 重载 systemd
sudo systemctl daemon-reexec
sudo systemctl daemon-reload

# 启用并启动服务
sudo systemctl enable douyin-monitor.service
sudo systemctl start douyin-monitor.service
```

## 🧹 五、配置日志自动清理（每日轮转）

```bash
sudo tee /etc/logrotate.d/douyin-monitor << 'EOF'
/opt/douyin_monitor/log/monitor.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    copytruncate
    su root root
}
EOF
```

✅ 系统通过 `systemd timer` 自动运行 logrotate，无需额外设置

## 🔍 六、验证与调试

### 1. 查看服务状态

```bash
sudo systemctl status douyin-monitor.service
```

### 2. 查看实时日志

```bash
tail -f /opt/douyin_monitor/log/monitor.log
```

### 3. 手动触发一次日志轮转（测试）

```bash
sudo logrotate -f /etc/logrotate.conf
ls -la /opt/douyin_monitor/log/
```

## 📌 七、关键配置说明

| 文件                                         | 作用     | 注意事项                      |
| -------------------------------------------- | -------- | ----------------------------- |
| `/opt/douyin_monitor/users.conf`             | 用户列表 | `sec_user_id\|昵称`，每行一个 |
| `/opt/douyin_monitor/douyin_monitor.sh`      | 主脚本   | 修改后需重启服务              |
| `/etc/systemd/system/douyin-monitor.service` | 服务托管 | 修改后需 `daemon-reload`      |
| `/etc/logrotate.d/douyin-monitor`            | 日志清理 | 保留 7 天，自动压缩           |

## 🛠️ 八、常见维护命令

```bash
# 启动服务
sudo systemctl start douyin-monitor.service

# 停止服务
sudo systemctl stop douyin-monitor.service

# 重启服务
sudo systemctl restart douyin-monitor.service

# 查看状态
sudo systemctl status douyin-monitor.service

# 查看日志
tail -f /opt/douyin_monitor/log/monitor.log

# 重新加载配置（修改 service 文件后）
sudo systemctl daemon-reload
```

## ✅ 九、功能总结

| 功能             | 是否支持 | 说明                |
| ---------------- | -------- | ------------------- |
| 监听多个抖音用户 | ✅       | 支持                |
| 排除置顶视频干扰 | ✅       | 精准识别非置顶最新  |
| 钉钉群推送通知   | ✅       | 通过钉钉群机器人    |
| 自动清理日志     | ✅       | 保留 7 天，自动压缩 |
| 开机自启         | ✅       | systemd 托管        |
| 随机检测间隔     | ✅       | 避免被限流          |

## 📎 十、注意事项

- **替换 Webhook URL**：将脚本中的 `YOUR_ACCESS_TOKEN` 替换为你自己的钉钉群机器人 Access Token
- **获取 sec_user_id**：可通过抖音分享链接解析，或使用浏览器开发者工具抓包获取
- **权限问题**：所有文件归 `root` 所有，确保脚本可读写
- **API 服务**：需要部署对应的抖音 API 服务，脚本默认调用本地 `http://localhost/api/douyin/web/fetch_user_post_videos`
