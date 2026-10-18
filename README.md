# kstack

Kieran 的个人 skill 集，面向前后端全栈开发，用来替代 mattpocock/skills、emil 和 pstack。

- 设计与取舍：[docs/design.md](docs/design.md)
- skill 源文件：`skills/<name>/SKILL.md`，按需读取的参考资料放在 `skills/<name>/references/`
- skill 正文用英文写，文档用中文写

## 本地安装

开发阶段把每个 skill 软链接到 `~/.agents/skills/`：

```bash
for d in skills/*/; do ln -sfn "$PWD/$d" ~/.agents/skills/"$(basename "$d")"; done
```
