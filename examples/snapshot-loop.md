# 三步循环演练 | Snapshot-loop drill, including a deliberate STALE_REF

把「看 → 按 ref 做 → 再看验证」练成肌肉记忆，顺带亲手触发一次 STALE_REF，学会正确反应：

1. **装好并过权限关**：`npm install -g agent-desktop` 后先跑 `agent-desktop status`；macOS 用户跑 `agent-desktop permissions --guide` 把无障碍权限授好，Linux 用户确认自己在图形会话里且装了 `at-spi2-core`。
2. **读内置文档**：把 `agent-desktop skills get desktop` 和 `agent-desktop skills get platform` 的输出完整看一遍——这是与你手头版本严格对应的说明书，比任何二手教程都准。
3. **走一遍干净循环**：挑一个无害应用（如 Finder 或文本编辑器）：`snapshot --app <应用名> -i` 找到目标元素，记下它完整的 `@快照id:e序号`，执行一次 `click`，立刻再 `snapshot` 一次，确认界面状态真的变了。
4. **故意制造 STALE_REF**：不关应用，手动在应用里切换页面或开个弹窗，再拿第 3 步的旧 ref 执行一次动作——观察返回的 `STALE_REF`（或 `AMBIGUOUS_TARGET`）错误信封，看清 `code`、`disposition`、`suggestion` 三个字段长什么样。
5. **按规矩恢复**：重拍快照、换新 ref、重做动作并验证成功。记住结论：STALE_REF 不是故障，是这套工具的安全设计——脚本里遇到它就重拍，不要重试旧 ref，更不要改用坐标硬点。
