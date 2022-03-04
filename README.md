### 项目介绍
利用ESLint+Husky+lint-staged进行提交前lint的示例代码

# Lint最佳实践
业界比较常用的方案是Husky配合lint-staged在代码提交前进行Lint，防止将不规范的代码提交到远端。

# 配置步骤
1、修改package.json
```
{
  "scripts": {
    "prepare": "husky install"
  }
}
```
说明：prepare：在打包和发布包之前运行，在本地 npm install 时运行，不带任何参数。这是在 prepublish 之后运行，但在 prepublishOnly 之前运行。
​

2、安装husky和lint-staged
```
yarn add husky lint-staged -d
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/269389/1646376914328-afb03c67-62b8-4fbc-bef0-e0be5e6ab6b9.png#clientId=ua563b106-8c30-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=73&id=u29cd17e4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=146&originWidth=484&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14938&status=done&style=none&taskId=u27336145-2237-46b3-a418-a7c40584c28&title=&width=242)
安装完会看到日志里出现 husky install，正式prepare里配置的内容，随着安装运行了。
我们看项目目录会发现根目录下多了以下文件：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/269389/1646376995072-423630b1-54d3-4ece-a246-a90e87a24b30.png#clientId=ua563b106-8c30-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=66&id=u808240d8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=132&originWidth=352&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8602&status=done&style=none&taskId=ubb9f150a-c259-4126-8898-6d0adfcba95&title=&width=176)
以上两步骤如果顺序错了也没关系，手动运行一下prepare，让husky install执行即可
```
npm run prepare
```


3、创建hook
```
npx husky add .husky/pre-commit "npx lint-staged"
```
4、根目录创建**.lintstagedrc.json**
```
{
  "*.{js,jsx,ts,tsx}": ["eslint  --fix"]
}
```
5、然后编写代码，提交试试
![image.png](https://cdn.nlark.com/yuque/0/2022/png/269389/1646378116935-f6402953-391c-4aab-9a12-d70c985d5a61.png#clientId=ua563b106-8c30-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=327&id=u9040e759&margin=%5Bobject%20Object%5D&name=image.png&originHeight=654&originWidth=1014&originalType=binary&ratio=1&rotation=0&showTitle=false&size=96108&status=done&style=none&taskId=u92108d8d-0b0a-4f04-9b1b-3f692bde7b8&title=&width=507)
6、如果由于某些原因，想强行提交，可以再后面添加参数--no-verify跳过
```
 git ci -m 'test husky' --no-verify
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/269389/1646378199960-b50c7878-8f9c-4613-97e3-d065f5ac6e2d.png#clientId=ua563b106-8c30-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=75&id=uf46c9dd8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=150&originWidth=960&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31660&status=done&style=none&taskId=u605a937f-54fd-4e99-bf2a-03e07bf554c&title=&width=480)


# husky原理解析
## 旧版husky
### git hooks介绍
Husky是如何在代码提交时触发代码校验的？在研究它的原理之前，需要介绍另外一个概念：git hooks，官方文档的描述是：
> 和其它版本控制系统一样，Git 能在特定的重要动作发生时触发自定义脚本。 有两组这样的钩子：客户端的和服务器端的。 客户端钩子由诸如提交和合并这样的操作所调用，而服务器端钩子作用于诸如接收被推送的提交这样的联网操作。 你可以随心所欲地运用这些钩子。

目前git支持17个hooks(这里存疑)，都以单独的脚本形式存储在.git/hooks文件夹下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/269389/1646368510075-4185935d-10b5-494e-95dd-dd6759343299.png#clientId=ua563b106-8c30-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=124&id=u503fad27&margin=%5Bobject%20Object%5D&name=image.png&originHeight=248&originWidth=954&originalType=binary&ratio=1&rotation=0&showTitle=false&size=88928&status=done&style=none&taskId=ua9111cf8-d185-4898-9188-d0d65055cae&title=&width=477)
以一次commit为例，会先后触发pre-commit、prepare-commit-msg、commit-msg和post-commit等hooks。我们可以利用这些hooks做一些有趣的事。比如：我们可以利用pre-commit进行代码校验，利用commit-msg进行commit message的校验，只要你懂得shell语法，当然你也可以使用Perl、Ruby或者Python。
另外，并不需要我们创建所有hook脚本，只需要按需创建即可。
### Husky和git hooks的关系
Husky[官方的描述](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fhusky)是：
> Git hooks made easy（让git hooks变得简单）

想象一个场景，比如在一个多人协作的团队，你在.git/hooks中创建了一些hooks，你希望共享给队友，但.git/hooks文件夹并不会提交到远端，无奈只能拷贝。
Husky就是为了解决这个问题而生的，只需要简单的配置，就可以完成hook的工作，具体配置方法参见Husky[使用文档](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fhusky%23upgrading-from-014)，以package.json为例：
```
// package.json
"husky": {
    "hooks": {
      "pre-commit": "eslint"
    }
  },

```
在npm安装Husky时，Husky会在项目的.git/hooks文件夹下创建所有支持的hooks，另外还会创建 husky.local.sh和husky.sh两个文件。其实每个hook脚本的内容都一样：​
```
# pre-commit
#!/bin/sh
# husky

# Created by Husky v4.2.5 (https://github.com/typicode/husky#readme)
#   At: 2020/8/3 上午11:25:21
#   From: ...(https://github.com/typicode/husky#readme)

. "$(dirname "$0")/husky.sh"
```
我们可以看到仅仅是执行husky.sh脚本，重点在husky.sh脚本中。
husky.sh脚本的内容和解读详情可以看这里：[https://juejin.cn/post/6879955438482227207](https://juejin.cn/post/6879955438482227207),大体流程就是从配置文件中（package.json、.huskyrc、.huskyrc.json...）读取相应的 hook 配置，执行配置中的指令/脚本。
我们再总结一下：
v4版本之前的husky的工作方式是这样的：为了能够让用户设置任何类型的git hooks，husky不得不创建所有类型的git hooks
这样做的好处就是无论用户设置什么类型的git hook，husky都能确保其正常运行。但是缺点也是显而易见的，即使用户没有设置任何git hook，husky也向git中添加了所有类型的git hook。
以上是旧版本的husky原理。
​

# 新版本husky
### core.hooksPath
直到 2016 年，Git 2.9引进了**core.hooksPath**，**可以设置Git hooks脚本的目录**，这个引进也就是新版husky改进的基础：
**可以使用husky install将git hooks的目录指定为.husky/**
```
git config core.hookspath .
```
使用husky add命令向.husky/中添加hook
通过这种方式我们就可以只添加我们需要的git hook，而且所有的脚本都保存在了一个地方（.husky/目录下）因此也就不存在同步文件的问题了。
​

# 为什么husky不自动安装git hook了
新husky的一个改变就是不自动安装git hook了。之前是在husky包里面，有个install
![image.png](https://cdn.nlark.com/yuque/0/2022/png/269389/1646379885578-76109467-8694-4209-afad-16e1565fc0ac.png#clientId=ua563b106-8c30-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=258&id=u50a5c0db&margin=%5Bobject%20Object%5D&name=image.png&originHeight=516&originWidth=1478&originalType=binary&ratio=1&rotation=0&showTitle=false&size=161243&status=done&style=none&taskId=ue4168ef9-6c5f-4d40-a842-0b7c1166237&title=&width=739)
会随着这个husky安装包本身安装的时候执行。现在推荐用户自己在自己项目的package.json中配置prepare
```
{
  "prepare": "husky install"
}
```
npm4引入了prepare钩子，行为等同于prepublish
prepare: Run both BEFORE the package is packed and published, and on local **npm install** without any arguments (See below). This is run AFTER **prepublish**, but BEFORE **prepublishOnly**
prepare：在打包和发布包之前运行，在本地 npm install 时运行，不带任何参数（见下文）。这是在 prepublish 之后运行，但在 prepublishOnly 之前运行。
​

### 包管理最佳实践
多年来，包管理器不鼓励将 postinstall 用于编译以外的任何事情。通过删除自动安装，新的 husky 是一个更好的公民，也是第一个支持 Yarn 2 零安装的版本（before, it needed to be unplugged）
> 来自：[npm docs: best practices](https://docs.npmjs.com/cli/v7/using-npm/scripts#best-practices)
> 您几乎不必显式设置预安装或安装脚本。如果您这样做，请考虑是否有其他选择。安装或预安装脚本的唯一有效用途是编译

> 来自：[Yarn 2 docs: a note about postinstall](https://yarnpkg.com/advanced/lifecycle-scripts#a-note-about-postinstall)
> 安装后脚本会对您的用户产生非常实际的影响。 [...] 重点是我们认为postinstall scripts是一个可行的解决方案

### 最近有一些关于autoinstall的一些issue
随着最近包管理器的改进（npm 7、Yarn 2），用户开始对 husky 4 自动安装问题进行复杂的调试。
**它变得不那么可靠了**：为了提供更快的安装，包管理器使用缓存（这非常好）。
因此，husky 4 postinstall 脚本现在并不总是运行。
例如，如果由于某些原因 husky 4 第一次安装失败，由于缓存，重新运行 npm install 将无法正常工作
**没有输出了**: 包管理器现在隐藏postinstall的输出.
没有更多确认已安装挂钩或信息消息来帮助用户调试问题
这可能看起来不多，但在开源项目中，具有不同经验的用户可能会克隆一个项目。因此，有一些信息就变得很重要。尤其是对于像 husky 这样具有很大副作用（更改 Git 钩子）的工具。
**如果您认为 postinstall 应该只用于编译，那么这些行为（一次性 postinstall，没有输出）是有意义的。**
### 总结
Husky因为以下原因放弃了自动安装：

- 遵循包管理器的最佳实践和演变
- 更加可靠和可预测
- 为具有不同经验的人保留用户友好的消息
# 
# 参考
[https://juejin.cn/post/6844903479283040269](https://juejin.cn/post/6844903479283040269)
[Husky原理解析及在代码Lint中的应用](https://juejin.cn/post/6879955438482227207)
[Why husky doesn't autoinstall anymore](https://blog.typicode.com/husky-git-hooks-autoinstall/)
[为什么 husky 放弃了传统的 JS 配置](https://segmentfault.com/a/1190000040717780)
​

