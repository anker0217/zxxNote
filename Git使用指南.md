---
date: 2025-11-17
tags:
  - Git
---
Git通过仓库(repository)来管理版本，简称repo，而仓库又分为远程repo和本地repo。远程repo通常建立在github、gitee等网站上，而本地repo通过git init命令在本地创建。
远程repo在创建时，以github为例，不要选择自动创建readme.md，不然之后把本地仓库push上去时，会因为远端分支和本地分支不一致发生冲突，要使用push -f

在本地repo链接到远端repo时，可以使用http链接和ssh链接，http需要输入github账号密码（大概吧，没具体用过），这里通过一个通用的方法，创建ssh链接

## 检查是否存在 SSH 密钥

在生成新的 SSH 密钥之前，应该检查本地计算机上是否存在密钥。

1. 打开Git Bash。
    
2. 输入`ls -al ~/.ssh`以查看是否存在现有的 SSH 密钥。
    
    ```shell
    $ ls -al ~/.ssh
    # Lists the files in your .ssh directory, if they exist
    ```
    
3. 检查目录列表，看看你是否已经拥有一个公钥。默认情况下，GitHub 支持的公钥文件名如下。
    
    - _id_rsa.pub_
        
    - _id_ecdsa.pub_
        
    - _id_ed25519.pub_
        
    
    提示
    
    _如果收到“~/.ssh_不存在 ”的错误提示，则说明在默认位置没有 SSH 密钥对

## 生成新的 SSH 密钥

可以在本地计算机上生成新的 SSH 密钥。生成密钥后，您可以将公钥添加到您在 GitHub.com 上的帐户，以启用通过 SSH 进行 Git 操作的身份验证。

1. 打开Git Bash。
    
2. 请粘贴以下文本，并将示例中使用的电子邮件地址替换为您的 GitHub 电子邮件地址。
    
    ```shell
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ```
    
    如果您使用的是不支持 Ed25519 算法的旧系统，请使用：
    
    ```shell
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```
    
    当系统提示“输入保存密钥的文件”时，您可以按**Enter 键**接受默认文件位置。请注意，如果您之前创建过 SSH 密钥，ssh-keygen 可能会要求您重写另一个密钥，在这种情况下，我们建议您创建一个自定义名称的 SSH 密钥。为此，请输入默认文件位置，并将 id_ALGORITHM 替换为您的自定义密钥名称。
    
```powershell
> Enter file in which to save the key (/c/Users/YOU/.ssh/id_ALGORITHM):[Press enter]
```
    
3. 在提示符处，输入安全密码短语。
    
    ```shell
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```
	 

## 使用 SSH 密钥密码短语

在上一步设置了ssh密钥密码短语可以有效的保护电脑安全，但这也意味着每次使用ssh密钥时都需要输入一遍，如在git push/pull时，比较麻烦。但除了麻烦还会有其他问题，在git bash中这肯定没问题，但在某些插件中，在vscode中还好，在obsidian的git插件中，并未提供输入密码的选项，会直接导致同步失败。

1. 添加或更改密码短语

	可以通过输入以下命令来更改现有私钥的密码短语，而无需重新生成密钥对：

```shell
$ ssh-keygen -p -f ~/.ssh/id_ed25519
> Enter old passphrase: [Type old passphrase]
> Key has comment 'your_email@example.com'
> Enter new passphrase (empty for no passphrase): [Type new passphrase]
> Enter same passphrase again: [Repeat the new passphrase]
> Your identification has been saved with the new passphrase.
```

如果您的密钥已经设置了密码，则在更改为新密码之前，系统会提示您输入旧密码。

2. `ssh-agent`在 Windows 版 Git 上自动启动

可以`ssh-agent`设置在打开 bash 或 Git shell 时自动运行。复制以下几行代码并粘贴到`~/.profile`或 Git shell 的`~/.bashrc`中：

```bash
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2=agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add
fi

unset env
```

如果ssh私钥未存储在默认位置（例如 `/usr/local/bin` `~/.ssh/id_rsa`），则需要告知 SSH 身份验证代理其存储位置。要将密钥添加到 ssh-agent，==将`ssh-add`改为`ssh-add ~/path/to/my_key`==。(待测试，我并没有具体试过)

如果想`ssh-agent`在一段时间后忘记密钥，可以修改命令为`ssh-add -t <seconds>`。

3. 现在，当你第一次运行 Git Bash 时，系统会提示你输入密码：

```shell
> Initializing new SSH agent...
> succeeded
> Enter passphrase for /c/Users/YOU/.ssh/id_rsa:
> Identity added: /c/Users/YOU/.ssh/id_rsa (/c/Users/YOU/.ssh/id_rsa)
> Welcome to Git (version 1.6.0.2-preview20080923)
>
> Run 'git help git' to display the help index.
> Run 'git help <command>' to display help for specific commands.
```

该`ssh-agent`进程将持续运行，直到您注销、关闭计算机或终止该进程为止。

## 将SSH公钥添加到github中

这部比较简单，如果都是默认路径，打开`~/.ssh/id_rsa.pub`，复制其中的内容，在github setting的ssh key管理中新建ssh密钥，可以根据设备取个名字，然后将复制的公钥粘贴到对应位置即可。