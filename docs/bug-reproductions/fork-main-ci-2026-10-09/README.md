# main `3062b9d8d0` 上的 CI 记录

`onlyfeng/orca` 以跟进 `stablyai/orca` 为主。

- 不把本地行为改动合并进用来跟踪 upstream 的 `main`。侧枝或这份记录就够。
- 测试可以在侧枝上调整。不要为了让 fork 的 CI 变绿去改运行行为。
- 不主动向 `stablyai/orca` 提 pull request。

侧枝是 `cursor/fix-main-ci-failures-a1ad`。上面只留了两处测试调整。

## 测试调整

空闲换区会拒绝连上不足 3 分钟的控制连接。`tests/e2e/relay-region-correction.unit.test.ts` 把时钟冻在连接瞬间，所以本该完成的迁移一直返回 busy，其中一次会等到超时。Node 24 和 Node 26 的 [Scheduled x86 unit compatibility](https://github.com/onlyfeng/orca/actions/runs/37814959208) 都是这个结果。侧枝里的测试先把时钟拨过 3 分钟，并续上控制租约。本地这组 16 个测试通过。

`find` 的顺序跟文件系统走。同一条 x86 任务里，中继目录列举的两个名字是 `bbb` 然后 `aaa`，断言写死了相反顺序。侧枝里的测试只比较名字。本地 `ssh-remote-commands` 测试通过。

## 只记录、不改代码

Node 26.11.1 的 `node:sqlite` 会把显式 `undefined` 写成 NULL 并落库。Node 26.0.0 和 Node 24 会抛错。失败测试是 `src/main/sqlite/sync-database-portability.test.ts`，在上面的 x86 任务里。包装层可以在写入前拒绝 `undefined`，但那是运行行为，留给 upstream。

[Skill update round trip](https://github.com/onlyfeng/orca/actions/runs/37712906871) 的 13 个任务都停在 `git ls-tree v1.4.178-rc.2`。这个 fork 没有发布标签，标签在 `stablyai/orca`。快照登记里的 git tree 能对上历史技能内容。改脚本可以绕过标签，但会让以后同步 upstream 时这一文件分叉，所以没有留在侧枝上。

## 没有收成一处改动

- 夜间 E2E：14 个分片里 9 个失败，失败的界面测试互不相关。https://github.com/onlyfeng/orca/actions/runs/37849746785
- Terminal IME：Wayland 韩文末位数字有一条断言失败。https://github.com/onlyfeng/orca/actions/runs/37811616037
- Terminal Perf：打字延迟超过 25ms 预算。https://github.com/onlyfeng/orca/actions/runs/37803259367
- 调色板性能预算：只在 Node 26 的 x86 分片上超时。
- Performance contracts：被取消。
