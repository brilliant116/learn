
#### 1.安装hexo

安装node, npm

```
sudo pacman -S nodejs npm
```

在安装的过程中，遇到了EACCES权限错误。参考npm官网[解决方法](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)

我在arch wiki上也找到了类似的[解决方法](https://wiki.archlinux.org/index.php/Node.js_)。修改`~/.profile`，加入以下内容:

```
PATH="$HOME/.node_modules/bin:$PATH"
export npm_config_prefix=~/.node_modules
```

然后就可以用了。

```
npm install -g hexo-cli
```

然后就可以初始化文件夹，在想要的位置创建这个文件夹。

```
hexo new folder_name
cd folder_name
npm install
```

#### 2.将博客部署到github

首先就是新建个人仓库，就是创建一个名为username.github.io的仓库。
再生成SSH key添加到github上，
在命令行

```
git config --global user.name "username"
git config --global user.email "useremail"
```

可以如下命令检查是否正确

```
git config user.name
git config user.email
```

这个时候它会告诉你已经生成了.ssh的文件夹，在你的电脑中找到这个文件夹。
而后在GitHub的setting中，找到SSH keys的设置选项，点击New SSH key，把你的id_rsa.pub里面的信息复制进去。
检查是否正确

```
ssh -T git@github.com
```

然后打开配置文件_config.yml进行修改。

```
deploy:
  type: git
  repo: https://github.com/githubName/githubName.github.io.git
  branch: master
```

这个时候需要先安装deploy-git ，也就是部署的命令,这样你才能用命令部署到GitHub。

```
npm install hexo-deployer-git --save
```

然后

```
hexo clean
hexo generate
hexo deploy
```

或者用一行命令

```
hexo clean && hexo g -D
```

以后，你再次将本地的博客放到Github，都需要上述三条命令。

就博客在 <https://username.github.io> 网站

#### 写文字

```
hexo new post "hello world"
```

```
hexo generate
hexo g
```

```
hexo server
hexo s
```

```
hexo deploy
hexo d
```
