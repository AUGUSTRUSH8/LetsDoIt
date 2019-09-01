### 阿里云轻量应用服务器搭建VPS

- 大厂服务器
- 香港新加披服务器
- 方便管理
- 多用途

#### Video Site

- https://www.youtube.com/watch?v=01mU_z5Grso

#### 步骤概要
- 1.购买链接：[http://t.cn/AiKBsEQb](https://www.youtube.com/redirect?q=http%3A%2F%2Ft.cn%2FAiKBsEQb&redir_token=L7pTLMlqVooFo-YA-zphg4Xsf6J8MTU2NzQxMTAzMUAxNTY3MzI0NjMx&v=01mU_z5Grso&event=video_description)
- 2.选择购买轻量服务器（优先选择香港地区的服务器，其次新加坡），系统建议选择Debian
- 3.等待系统安装完成后设置root密码
- 4.通过【使用浏览器发起安全连接（推荐）】连接服务器
- 5.输入su（这个命令是用来切换root权限），然后输入你刚设置的root密码
- 6.进入系统后安装ssr，复制下面的命令到控制台：
  wget -N --no-check-certificate [https://heikejilaila.xyz/ss/ssrmu.sh](https://www.youtube.com/redirect?q=https%3A%2F%2Fheikejilaila.xyz%2Fss%2Fssrmu.sh&redir_token=L7pTLMlqVooFo-YA-zphg4Xsf6J8MTU2NzQxMTAzMUAxNTY3MzI0NjMx&v=01mU_z5Grso&event=video_description) && chmod +x ssrmu.sh && bash ssrmu.sh
- 7.输入1按回车就开始安装了（注意看中文提示）
- 8.建议参数：
  端口可以随机填（要在安全组打开端口放行）
  加密选择：none
  协议选择：auth_chain_b
  混淆选择：http_simple
- 9.去阿里的控制台，找到安全--防火墙，单击添加规则，为了简单方便，可以把应用类型选为【全部tcp】，确定即可。
- 10.开启加速，输入命令：
  bash ssrmu.sh
  然后输入14回车，按照菜单提示选择安装锐速启用即可。