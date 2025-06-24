下面是一个示例 GitHub Actions Workflow 脚本，演示如何自动删除最近一次提交（即重置到上一个提交）并强制推送到远程仓库。你可以根据需求修改重写历史的命令。

```yaml
name: Auto Delete Last Commit

on:
  workflow_dispatch:  # 手动触发，也可以改为 schedule 定时触发
  # schedule:
  #   - cron: '0 0 * * *'  # 每天午夜执行

jobs:
  delete-last-commit:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository with full history
        uses: actions/checkout@v3
        with:
          fetch-depth: 0  # 获取完整历史，方便重写

      - name: Configure Git user
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Delete last commit and force push
        run: |
          git reset --hard HEAD~1
          git push origin HEAD --force
        env:
          # 需要设置有推送权限的令牌，默认 GITHUB_TOKEN 有权限
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

说明：
- 该工作流默认手动触发（workflow_dispatch），你也可以取消注释 schedule 部分实现定时触发。
- 使用 `git reset --hard HEAD~1` 删除最近一次提交，你可以改成其他 git 命令实现更复杂的历史重写。
- `actions/checkout@v3` 的 `fetch-depth: 0` 确保拉取完整历史。
- 使用默认的 `${{ secrets.GITHUB_TOKEN }}` 进行身份验证，确保该令牌有推送权限。
- 强制推送会覆盖远程历史，慎用！

如果你需要删除特定提交或批量清理历史，可以告诉我，我帮你写更复杂的脚本。