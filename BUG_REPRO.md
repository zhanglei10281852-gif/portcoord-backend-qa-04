# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

并发额度预留时所有请求都显示成功，最终预留量超过每日上限。请修复并发容量控制，确保只有仍有额度的请求能够成功。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/portcoord-backend-qa-04
- 仓库地址：https://github.com/zhanglei10281852-gif/portcoord-backend-qa-04.git
- parent SHA：628d2bf5f72ed2240eaae0d95e314d878f57abd8

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/portcoord-backend-qa-04.git bug-repro
cd bug-repro
git checkout --detach 628d2bf5f72ed2240eaae0d95e314d878f57abd8
go test ./internal/quota -run "^TestQuota_ConcurrentReserveRace$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/quota -run "^TestQuota_ConcurrentReserveRace$" -count=1
--- FAIL: TestQuota_ConcurrentReserveRace (0.01s)
    quota_test.go:134: expected exactly 10 successful, got 20
    quota_test.go:137: expected 10 rejected, got 0
FAIL
FAIL	portcoord/internal/quota	0.019s
FAIL

```

stderr：

```text
warning: internal/quota/quota_test.go has type 100755, expected 100644
warning: internal/quota/quota_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/quota -run "^TestQuota_ConcurrentReserveRace$" -count=1
--- FAIL: TestQuota_ConcurrentReserveRace (0.35s)
    quota_test.go:134: expected exactly 10 successful, got 20
    quota_test.go:137: expected 10 rejected, got 0
FAIL
FAIL	portcoord/internal/quota	0.610s
FAIL

```

stderr：

```text
warning: internal/quota/quota_test.go has type 100755, expected 100644
warning: internal/quota/quota_test.go has type 100755, expected 100644

```

## 通过条件

在题面触发条件下，公开行为必须恢复且原始异常不再出现；定向命令 go test -race ./internal/quota -run ^TestQuota_ConcurrentReserveRace$ -count=1、相关包测试、全量测试、race、vet 和 build 必须通过；不得删除或跳过测试，也不得绕过目标逻辑。
