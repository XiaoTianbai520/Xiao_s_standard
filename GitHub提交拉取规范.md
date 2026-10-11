# GitHub 提交与拉取规范

> 适用于团队协作开发，目标：减少代码冲突，保持仓库整洁。

## 1. 开始写代码前

先同步远程最新代码：

```bash
git fetch origin
git pull --rebase origin <分支名>
```

确认本地代码为最新版本后再开始开发。

## 2. 开发过程中

- 尽量只修改自己负责的代码，避免随意改动他人负责的模块。
- 如果必须修改公共代码或他人负责的代码，提前沟通。
- 遇到代码冲突时及时联系相关负责人一起确认，不要随意覆盖代码。

## 3. 提交代码前

先检查修改内容：

```bash
git status
git diff
```

确认无误后提交：

```bash
git add .
git commit -m "feat: 完成xxx功能"
```

提交信息保持简洁，说明本次修改内容。

## 4. 推送代码前

再次同步远程代码，避免直接覆盖队友的新提交：

```bash
git fetch origin
git pull --rebase origin <分支名>
```

如果出现冲突，解决并确认无误后再推送：

```bash
git push origin <分支名>
```

## 5. 推荐操作顺序

```text
开始开发
   ↓
fetch + pull --rebase
   ↓
编写代码
   ↓
status / diff 检查
   ↓
add + commit
   ↓
再次 fetch + pull --rebase
   ↓
解决可能的冲突
   ↓
push
```

**核心原则：先拉取、再开发；先同步、再推送；只改自己负责的代码，有冲突先沟通。**
