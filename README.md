# C++ 语块阅读器 · 教学文件

一个 **三周从零** 的 C++ 大作业实战教程：用 C++ 做一个「英文语块切分 + 阅读计时」程序，
前端用 Python 展示，中间用 JSON 打通。

> 每天一课，做完把结果反馈给老师/助教，确认后再进下一课。

---

## 最终你会做出什么

```
输入：一段英文
   ↓
C++ 引擎：分词 → 词性 → 语块切分（DP）→ 计时
   ↓
JSON
   ↓
Python 界面：语块高亮、逐块计时、WPM 报告
```

---

## 目录

| 文件 | 内容 | 状态 |
|---|---|---|
| [tutorial/00-roadmap.md](tutorial/00-roadmap.md) | 三周作战地图、每日任务、检查点 | ✅ |
| [tutorial/01-hello.md](tutorial/01-hello.md) | Day 1：装 VS、写第一个程序、看懂编译错误 | ✅ |
| [tutorial/02-read-file.md](tutorial/02-read-file.md) | Day 2：读文件、统计行数/字符数 | ✅ |
| [tutorial/03-tokenize.md](tutorial/03-tokenize.md) | Day 3：分词、Token 结构体 | ✅ |

---

## 进度

- [x] **Day 1** — Hello World、`C2059` / `LNK2019` 错误识别
- [x] **Day 2** — `ifstream` + `getline`，能统计行数与字符数
- [ ] Day 3 — 分词
- [ ] Day 4 — 结构体 Token
- [ ] Day 5 — 输出 JSON
- [ ] Day 6 — Python 界面
- [ ] Day 7 — 复盘提交

---

## 怎么用这份教程

1. 按顺序读，**不要跳课**
2. 每课都有「验收标准」，全部打勾才往下走
3. 卡住超过 30 分钟，把**完整报错**发出来
4. 每天结束在代码仓库里 commit 一次

---

## 三条铁律

| # | 规则 |
|---|---|
| 1 | 每一步都运行一次，不要写完 200 行才编译 |
| 2 | 报错先看**第一条**，后面的多半是连锁反应 |
| 3 | 卡住 30 分钟就问，别硬耗一晚上 |

---

## 环境

- Windows 10/11
- Visual Studio 2026 Community（工作负载：使用 C++ 的桌面开发）
- Python 3.8+（只用标准库 tkinter）
- Git 2.x
