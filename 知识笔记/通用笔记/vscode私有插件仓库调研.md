---
tags:
  - 知识笔记
---
#### 本次调研记录
1. 基于openvscode-server开始，启动环境
	1. 安装依赖
	2. 运行watch-webd，启动代码监听
	3. [[vscode私有插件仓库调研#^ad40ed|新增的脚本]]
	4. 启动web-server,这时候可以通过[http://localhost:9888/](http://localhost:9888/)访问本地vscode项目，这时候问题就在于这里的扩展菜单加载的是vscode官网的插件仓库，所以需要开始构建私有仓库，并连接到这里的vscode实例中![[Pasted image 20241231152016.png]]
2. 紧接着就需要搭建一个私有的本地extension服务，然后修改vscode源码，将vscode访问扩展的地址改为本地地址；这里经过搜索发现有一个openvsx的开源项目支持构建私有化的本地extension服务，那么下载openvsx的源码，试着部署到本地Docker，测试本地服务是否可行
	1. 直接通过docker-compose --profile openvsx up -d启动即可，第一次启动需要等一段时间。启动之后查看一下容器，所以访问3000端口，即可看到服务的ui页面（这边注册了六个任务，前两个分别是后端服务需要使用的数据库和ES服务，kibana是es的可视化服务，cli是命令行工具，webui是私有仓库的前端页面服务，server是对应的私有仓库的后端服务）![[Pasted image 20241231152057.png]]![[Pasted image 20241231152104.png]]
	2. 这时候服务启动就成功了，这里可以发现，当前仓库中一个插件都没有。通过页面的network发现，它是从8080的后端服务中加载的![[Pasted image 20241231152123.png]]
	3. 那么进入openvsx/server,源码中发现有一个test-extensions.gradle,里面定义了一个downloadTestExtensions的gradle命令，所以开始尝试将这个命令集成到Dockerfile中，测试一下能不能添加插件
	4. 看了一下命令干的事情，就是下载几个插件，然后通过cli命令发布到8080，那么可以直接跳过这一步，先本地开发一个测试扩展，然后发布到8080进行测试
	5. 开始开发测试插件
		1. 安装yo和generator-code包（这是官方推荐的开发vscode插件的三方包）
		2. 创建vscode web extension完成，yo构建插件项目时，需要注意选择web extension
		3. 打包之后准备发布到本地的私有扩展仓库
	6. 准备私有仓库发布
		1. 先查看cli下的publish.ts源码中明显可以看见需要通过pat传递一个token![[Pasted image 20241231152223.png]]
		2. 查看官方文档需要先去openvsx注册拿token，**（按道理本地发布是不希望有这个token的，且就算正式发布到自己的私有扩展的线上环境，这个token也不希望来自openvsx，但是这个token校验不仅仅来自于node，后端服务也有校验，如果真的需要接自己的鉴权服务，需要查看java源码进行删除或者修改，这边先跳过，直接使用官方的token，后续如果真的需要再去看这块怎么处理）**
		3. 下面开始去拿token，先去openvsx使用github登陆，再进入setting，最后进入profile中关联eclipse账号，没有需要创建![[Pasted image 20241231152244.png]]
		4. 再创建eclipse账号，注意创建时，github名称一定要填对，否则授权会出错
		5. 关联成功之后即可generate自己的发布token,[[vscode私有插件仓库调研#^a256b1|eclipse token记录]]
		6. 准备开始发布到私有扩展仓库
		7. 启动openvsx本地私有扩展仓库服务
		8. 发布失败。。。问题在于之前的token是发布到openvsx官方服务的，而本地需要控制callback_url,并且在github 的OAuth中创建项目指向本地服务，然后在服务中设置callback_url，相关callback_url的sh如下![[Pasted image 20241231152425.png]]
		9. 但是，本地能不能跳过呢，因为我还是希望本地能够跳过验证。所以回头再看一下之前的test-extensions.gradle发现它并不需要设置这些，所以分析一下源码。最终走到cli文件夹下有相关源码，原来是可以通过super_token绕过鉴权，同时也发现可以通过环境变量OVSX_REGISTRY_URL设置具体的服务地址我门现在就在本地所以可以不设置，**如果后续真的需要部署私服，需要在docker-compose.yml中配置相应的环境变量**![[Pasted image 20241231152439.png]]
		10. 所以先行测试cli中的load_test_extension是够可用，执行之后发现可用，所以分析一下这个文件发现他是从官网(代码里的publicReg)下载插件再发布到本地的8080(代码里的localReg)。既然可行那就自己写一个发布方法,使用super_token发布，并且创建namespace **somnus9527(注意这里的namespace并不是前端概念中的namespace，而是必须和配置的publisher一致才行，否则发布会失败)**,[[vscode私有插件仓库调研#^d503ed|自定义发布脚本]]
		11. 发布成功之后就可以发现我们的vscode扩展私服中就已经存在了一个插件了![[Pasted image 20241231152505.png]]
		12. 现在私服已经搭建完成，插件也已经发布成功，后续开始将本地的vscode服务的扩展关联到本地的私服上。
		13. 首先需要在openvscode-server的源码中修改product.json的扩展服务相关配置，将服务挂到本地启动的openvsx服务上,[[vscode私有插件仓库调研#^b4f6ae|product.json配置]]
		14. 重新编译之后，重启服务，会出现csp报错，发现源码中有大量的csp相关代码，难以修改，但是如果真的建立私服一定会以https的形式暴露，所以这块也并不需要去处理，所以简单的安装一个Disable Content-Security-Policy插件禁用csp检查，那么启动之后，进入插件搜索我们之前在私服中注册的插件athena-test-extension，就可以发现能够搜索到了![[Pasted image 20241231152531.png]]![[Pasted image 20241231152537.png]]
		15. 最后测试一下插件可用性，先进行安装![[Pasted image 20241231152550.png]]
		16. 安装完成之后，测试插件提供的命令是否可用,下面的图可以看到命令已经可用了![[Pasted image 20241231152604.png]]
		17. 执行效果![[Pasted image 20241231152615.png]]


>[!Tip] 命令
>- "web": "./scripts/code-web.sh"
>- "web-server": "./scripts/code-server.sh"

^ad40ed

>[!Tip] eclipse token
>`be64081e-76d4-47d7-a4de-d82b61b4e069`

^a256b1

>[!Tip] 自定义发布脚本
>```javascript
>  const { Registry } = require('../lib/registry');
>  const accessToken = 'super_token';
>  function parseEnv() {
> 	 const args = process.argv.slice(2);
> 	 const options = {};
> 	 args.forEach(arg => {
> 	   const [key, value] = arg.split('=');
> 	   if (key && value) {
> 	    options[key.replace('--', '')] = value;
> 	   }
> 	 });
> 	 return options;
>  }
>  function log(message, data) {
> 	console.log(`[Somnus Log] ${message} ${data ? `-- ${data}` : ''}`);
>  }
> async function createNamespace(namespace, localReg) {
> log('开始创建命名空间...');
> try {
>    const nsResult = await localReg.createNamespace(namespace, accessToken);
>    log('创建成功', nsResult.success);
>  } catch (error) {
>    if (error.message.startsWith('Namespace already exists')) {
>      log(`无需创建,namespace = ${namespace} 已存在`, error);
>    } else {
>     log('创建失败，请重试！');
>      process.exit(1);
>    }
>  }
>}
>  async function publish() {
>  log('开始发布本地vscode扩展...');
>  const localReg = new Registry({registryUrl: process.env.OVSX_REGISTRY_URL ?? 'http://localhost:8080'});
>  const options = parseEnv();
>  if (!options.path) {
>    log('必须提供上传扩展的绝对路径！');
>    process.exit(1);
>  }
>  if (!(options.ignoreNamespace && options.ignoreNamespace === 'ignore')) {
>    if (!options.namespace) {
>      log('必须提供上传扩展的命名空间！');
>      process.exit(1);
>    }
>  }
>  if (options.ignoreNamespace !== 'ignore') {
>    await createNamespace(options.namespace, localReg);
>  }
>  try {
>    const published = await localReg.publish(options.path, accessToken);
>    if (published.namespace && published.name) {
>      log(`\u2713 上传成功 ${published.namespace}.${published.name}@${published.version}`);
>    }
>  } catch (error) {
>    if (!error.message.endsWith('is already published.')) {
>      console.error(``);
>      log('\u274c 上传失败', error);
>    }
>  }
> }
> publish();
>```

^d503ed
>[!Tip] product.json配置
>```
>"extensionsGallery": {
>		"serviceUrl": "http://localhost:8080/vscode/gallery",
>		"itemUrl": "http://localhost:8080/vscode/item",
>		"resourceUrlTemplate": "http://localhost:8080/vscode/unpkg/{publisher}/{name}/{version}/{path}",
>		"controlUrl": "",
>		"recommendationsUrl": "",
>		"nlsBaseUrl": "",
>		"publisherUrl": ""
>	},
>	"linkProtectionTrustedDomains": [
>		"http://localhost:8080"
>	]
>```

^b4f6ae

#### 调研总结
1. 整体来看，扩展私服建立可行，vscode私服也可行。假如我们期望如下几种场景，需要做的事情如下：
	1. 场景：即提供自己的vscode浏览器端环境，又提供插件私服，那么需要做的事情如下：
		1. 编译一份openvscode-server，修改部分代码或者配置，并部署服务
		2. 编译一份openvsx，修改部分配置且增加一份专属的发布代码，并部署服务
	2. 场景：只提供vscode浏览器端环境，不提供插件私服，那么需要做的事情如下：
		1. 编译一份openvscode-server，修改部分代码或者配置，并部署服务
		2. 插件写完发布到open-vsx.org官方服务，使用统一的namespace （这里不发布到vscode的官方市场，是由于微软的限制，非微软官方软件，不允许直连vscode官方插件市场）
	3. 场景：都不提供，那么只需要写插件发布到vscode的官方市场，使用方直接使用vscode.dev的官方服务即可