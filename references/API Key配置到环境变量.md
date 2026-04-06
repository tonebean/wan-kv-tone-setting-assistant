# API Key 配置到环境变量

这份文件只保留这个 skill 真需要的最短配置方式。

## 目标

让调用时从环境变量 `DASHSCOPE_API_KEY` 读取密钥，而不是把真实 Key 写进：
- skill 文件
- 示例代码
- 提交材料
- GitHub 仓库

## Agent 应该怎么引导用户

默认优先引导用户做“当前会话临时配置”，测试通过后再决定是否写入长期配置文件。

### macOS / Linux 临时配置

```bash
export DASHSCOPE_API_KEY="YOUR_DASHSCOPE_API_KEY"
echo $DASHSCOPE_API_KEY
```

如果第二行能打印出值，说明当前终端会话已经可用。

### macOS 长期配置

如果用户确认之后还会长期使用，再追加到自己的 shell 配置文件。

对 zsh：

```bash
echo 'export DASHSCOPE_API_KEY="YOUR_DASHSCOPE_API_KEY"' >> ~/.zshrc
source ~/.zshrc
```

对 bash：

```bash
echo 'export DASHSCOPE_API_KEY="YOUR_DASHSCOPE_API_KEY"' >> ~/.bash_profile
source ~/.bash_profile
```

### Windows

让用户按自己习惯方式配置系统环境变量，变量名固定为：

```text
DASHSCOPE_API_KEY
```

配置后重新打开终端再测试。

## Agent 需要额外提醒的点

1. 不要让用户把真实 Key 直接贴进仓库文件  
   如果只是为了本地测试，优先临时配置。

2. 如果用户发来了真实 Key，也不要把它回写进 skill 文档  
   只在当前会话使用，文件里始终保留占位符。

3. 如果配置后仍然失败，先检查是不是地域和地址不匹配  
   很多“像是密钥错了”的问题，实际是地域混用了。
