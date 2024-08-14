# 网络工具


## ssh

ssh 免密登录步骤：

在 `.ssh` 目录下输入 `ssh-keygen` 指令，依提示建立好公钥和私钥。公钥是 `.pub` 后缀名的文件。将公钥里的内容拷贝到远程服务器 `.ssh` 目录下的 `authorized_keys` 文件，就可以免密登录了。如果是 vscode，再添加 `IdentityFile` 的配置，是私钥对应的文件。

在 mobaxterm 中，则直接配置 `Use Private Key` 即可。如果要配置跳板机，在添加时的 Network settings 中配置。


要连接到 github 也很简单，将上面生成的公钥在 github 的 setting 中添加。github 的服务器就有了我们的公钥，我们就可以通过 ssh 协议来克隆项目了。