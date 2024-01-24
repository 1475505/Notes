# 新服务器装机

1. 换源
```sh
sudo -i
bash <(curl -sSL https://linuxmirrors.cn/main.sh)
```

2. 国内连通 Github（[ineo6/hosts](https://github.com/ineo6/hosts)）

clone镜像：`hub.fgit.cf`

```sh
curl -sSL https://gitlab.com/ineo6/hosts/-/raw/master/next-hosts | sudo tee -a /etc/hosts
```

3. 安装zsh

4. 安装辅助工具
`sudo apt install screen tmux libnss3 -y`