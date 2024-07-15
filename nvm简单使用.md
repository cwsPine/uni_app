# NVM

## 前情提要

运行项目时发现界面效果错乱，打开控制台看到，**Node Sass does not yet support your current environment:  Windows 64-bit with Unsupported runtime** 

联想到刚才下载了node 18 版本，猜测可能与node.js、node-sass依赖环境有关。根本在于 node.js 版本过高带来的冲突。



## 安装并使用nvm管理工具

- 官网： https://github.com/coreybutler/nvm-windows/releases

- 选择下载：

  - ![image-20240715104349500](C:\Users\cws\AppData\Roaming\Typora\typora-user-images\image-20240715104349500.png)

- 将下载下来的文件，解压，得到一个.exe后缀的文件

- 安装完成后，检验安装是否成功，打开命令行窗口，输入 `nvm v` ，可以查看到当前安装的版本号。

  - <img src="C:\Users\cws\AppData\Roaming\Typora\typora-user-images\image-20240715104725456.png" alt="image-20240715104725456"/>

- 一切准备就绪之后，就可以开始安装我们的node.js版本啦

  1. 输入 **nvm list available** 可以查看当前可用的node.js 版本号

     - ![image-20240715105034472](C:\Users\cws\AppData\Roaming\Typora\typora-user-images\image-20240715105034472.png)
     - 如果列表中没有找到自己想要的版本号，也可以根据右下角网址 https://nodejs.org/en/download/releases 查找适合自己的版本号进行下载

  2. 输入命令行 `nvm install node版本号`  即可安装对应版本的node.js  

     - 这里放一张node 和 sass 版本对应表

       ![image-20240715110254365](C:\Users\cws\AppData\Roaming\Typora\typora-user-images\image-20240715110254365.png)

     - 这里展示下载node v12.22.12版本，`nvm install 12.22.12`

       - ![image-20240715110931203](C:\Users\cws\AppData\Roaming\Typora\typora-user-images\image-20240715110931203.png)
       - 可以看到 node 对应的 npm 版本都下载好了

  3. 这时输入命令 `nvm ls` 可以查看我们安装的所有node版本，以及你当前所选择使用的node版本号

     - ![image-20240715111224108](C:\Users\cws\AppData\Roaming\Typora\typora-user-images\image-20240715111224108.png)

  4. 最后根据需要输入  `nvm use node版本号`， 即可切换不同node版本  。（例：**nvm use 12.22.12**）

  5. 如果想删除那个版本号的node，也可以使用 `nvm uninstall node版本号` 命令来删除对应版本。





