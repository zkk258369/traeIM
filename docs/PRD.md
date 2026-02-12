# Nexus Chat - Product Requirement Document (PRD) v1.0
**Author**: Technical Architecture PM  
**Date**: 2026-02-12  
**Status**: Draft

## 1. 第一性原理审计 (First Principles Audit)

**核心命题**：为何需要另一个社交软件？Nexus Chat 的本质是什么？

*   **极速连接 (Connectivity)**: 用户需要极低延迟、无阻碍的沟通体验。
    *   *推论*: “像 Telegram 一样丝滑”意味着不仅是 UI 动画流畅，更意味着网络层协议优化（如 MTProto 风格的自定义协议或 HTTP/3 QUIC），以及本地数据库的极速读写。
*   **数据永存 (Permanence)**: “所有消息永久云端存储”解决了设备切换和历史回溯的痛点。
    *   *推论*: 存储成本将随时间线性甚至指数增长。架构必须支持冷热数据分离。
*   **社交与交易 (Social + Transaction)**: 结合聊天、朋友圈（动态墙）与支付。
    *   *推论*: 这是一个闭环生态系统。支付功能的引入意味着极高的安全合规要求。

**审计结论**: Nexus Chat 不仅仅是一个聊天工具，它试图构建一个集即时通讯、内容分享和价值交换于一体的高性能数字生活平台。核心技术壁垒在于**高性能客户端渲染**与**低成本海量云存储**的平衡。

---

## 2. 功能需求 (Functional Requirements)

### 2.1 用户认证 (Authentication)
*   **功能**: 手机号验证码一键登录。
*   **流程**: 输入手机号 -> 发送 SMS -> 验证通过 -> 自动注册/登录。
*   **细节**: 支持多端同步登录，JWT 鉴权。

### 2.2 核心聊天 (Core Chat)
*   **点对点 (P2P)**:
    *   支持文字、Emoji。
    *   支持高清图片（原图/压缩图发送）。
    *   支持语音消息（带波形图，支持暂停/播放）。
    *   **阅后即焚 (Self-Destruct)**:
        *   支持发送“阅后即焚”图片/视频。
        *   接收方点击查看后开始 5 秒倒计时。
        *   倒计时结束后，文件从本地彻底删除，并通知服务端同步物理删除。
        *   界面需防止截屏/录屏（Android 禁止截屏，iOS 监听截屏事件并通知发送方）。
*   **群聊 (Group Chat)**:
    *   支持 500 人上限大群。
    *   群管理功能（踢人、禁言、公告）。

### 2.3 社交增强 (Social+)
*   **动态墙 (Moments)**:
    *   类似朋友圈，用户发布图文/短视频。
    *   支持点赞、评论。
    *   隐私设置（仅好友可见/公开）。

### 2.4 商业化 (Commercialization)
*   **支付集成**:
    *   集成微信支付/支付宝 SDK。
*   **红包功能**:
    *   P2P 红包。
    *   群拼手气红包。
    *   资金流转需符合监管要求（即时清算或托管模式）。

---

## 3. 非功能需求 (Non-Functional Requirements)

### 3.1 交互与性能 (UX & Performance)
*   **Telegram 级丝滑**:
    *   列表滚动保持 60/120 fps。
    *   本地数据库优先策略 (Offline-first): 消息先写入本地 DB (如 SQLite/Realm) 立即上屏，后台异步发送。
    *   图片加载使用渐进式加载和内存缓存。

### 3.2 数据存储 (Data Storage)
*   **云端永存**: 所有历史消息不删除。
*   **同步策略**: 增量同步 (Sync)，换机登录拉取全量会话索引，按需拉取历史消息。

### 3.3 安全性 (Security)
*   传输层 TLS 1.3 加密。
*   支付密码/生物识别验证。

---

## 4. 技术架构概览 (Technical Architecture Overview)

*   **Client**: Flutter (跨平台，高性能渲染) 或 Native (Swift/Kotlin) 追求极致体验。建议首选 Flutter 配合 C++ 核心库以接近 Telegram 体验。
*   **Server**: Go (高并发处理 IM 消息) + gRPC。
*   **Database**: 
    *   NoSQL (MongoDB/Cassandra) 存储海量消息历史。
    *   MySQL 存储用户关系、账户资金数据（强一致性）。
    *   Redis 做消息队列和热点缓存。
*   **Object Storage**: S3 兼容存储 (AWS S3/MinIO) 存储图片、语音。

## 5. 风险预警 (待风险官评估)
*   云存储成本失控。
*   支付牌照与合规性。
*   内容安全 (鉴黄/反诈)。
