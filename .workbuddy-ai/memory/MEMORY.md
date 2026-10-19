# 项目长期记忆（boss-zhipin-scraper）

## 运行环境硬事实

- 本机存在 `http_proxy=127.0.0.1:61409`，任何 localhost HTTP 必须 `trust_env=False`，否则 502 文本被当成 JSON 报错（已由 `cdp_http_request` 统一处理，新代码不要直接用 `requests.get` 访问 127.0.0.1:9222）。
- **工具调用里启动的进程会被回收**：跑 `--setup-chrome` 或单独启动 Chrome，命令结束后 9222 端口就没了。正确姿势：同一次 Bash/PowerShell 调用内「启动 Chrome + 跑抓取」；超过 10 分钟的任务用 `run_in_background`。
- 详情抓取节奏：3 页 45 条 ≈ 25-30 分钟（每详情含 5-10s 加载 + 3-7 次滚动 + 10-25s 间隔），前台会超时。

## 本项目关键约定

- 抓取链路（列表/详情/登录探测）的任何改动必须真机验证，单测全 mock 覆盖不到浏览器焦点、定时器、SPA、风控。
- Chrome 偶发对 `Target.createTarget` 返回 `Failed to open a new tab` 并关闭 WebSocket：只能用**新连接**重试（`open_detail_page`），不要在同一连接上重试。
- `--analysis` = 终端报告 + Markdown 文件（`boss_report_*.md`），两者统计口径必须走同一个 `compute_analysis_stats()`；报告里岗位名/公司名要可点击（company_link 优先，回落 job_link，都没有就纯文本）。
- 测试里 mock `requests` 要用 `fake_requests_module()`（同时提供 Session().get 和模块级 get），否则 `cdp_http_*` 的新实现测不到。

## 用户偏好

- 用户在意「结果物要真的生成出来」：命令行跑完必须有可见产出文件（如 Markdown 报告），只往终端打印会被认为是有 bug。
- 用户的运行示例：`python scripts/boss_cdp_raw.py --keyword "前端" --city 广州 --pages 3 --analysis`。
