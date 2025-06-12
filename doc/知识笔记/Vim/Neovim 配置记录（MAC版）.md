#### 前置安装

- brew 安装 `/bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"](https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)")`
- `brew install git`
- `brew install node`
- python: [下载地址](https://www.python.org/downloads/macos/)
- `brew install cmake`
- `brew install make`
- `brew install cppcheck`
- `brew install neovim`
- `brew install neovide`
>[!Tip] 如出现无法打开，可以在隐私设置里面允许打开
- `brew install ripgrep`
- `brew install gnu-sed`
- `brew install fd`
- `brew install lua-language-server`
- `brew install fzf`
- `brew install sad`
- `brew install deno`
- `brew install luacheck`
- `brew install rust-analyzer`
- `brew install rustfmt`
- `brew install trash`
- ~~安装nerd fonts `brew tap homebrew/cask-fonts``brew install font-hack-nerd-font`~~
>[!Tip] 安装方式已失效，建议手动安装

#### 问题记录
1. Windows上的Alt组合快捷键在Mac上会失效，neovim和neovide都会失效，处理方式如下：
	1. 针对neovim -> 因为是在终端启动，所以需要在终端 -> 设置 -> 描述文件 -> 键盘 -> 钩上将option用作meta建
	2. 针对neovide，处理方式参照Mac处理Option会输入特殊字符这篇小记,另外需要增加配置`vim.g.neovide_input_macos_alt_is_meta = true`
> [!Tip] 同样失效，新版本是neovide_input_macos_option_key_is_meta="Both"
2. lua-json5 包build可能会报错，这时候需要在~/.cargo/config.toml文件中添加如下配置,然后重新运行lua-json5包里的install.sh重新打包(如果.cargo文件夹下没有config.toml文件，需要自己创建)
>[!Tip] 配置如下：
>```toml
>[target.x86_64-apple-darwin]
rustflags = ["-C", "link-arg=-undefined", "-C", "link-arg=dynamic_lookup"]
>[target.aarch64-apple-darwin]
>rustflags = ["-C", "link-arg=-undefined", "-C", "link-arg=dynamic_lookup"]
>```
3. 针对mac下neovim加载lua-json5失败的问题，第二步只能解决build失败，加载需要设置package.cpath，配置代码如下：
>[!Tip] lua代码如下：
>```lua
>if not utils.is_windows then
  package.cpath = package.cpath .. ';/Users/wangxuepei/.local/share/nvim/lazy/lua-json5/lua/json5.dylib'
end
>```
4. debug单js文件成功！！但是不知道为什么。。。回去之后测试，修改内容如下：
	1. 还是使用nvim-dap-vscode-js插件
	2. vscode-js-debug插件删除重新下载，配置如下：
>[!Tip] lua配置如下：
>```lua
>return {
  'microsoft/vscode-js-debug',
  module = true,
  version = '1.x',
  opt = true,
  build = 'npm install --legacy-peer-deps && npx gulp vsDebugServerBundle && mv dist out',
}
>```

5. 特殊情况，发现eslint_d找配置找不到，它跑去.pnpm文件夹里面去找了，这时需要执行`eslint_d restart`重启eslint_d服务。具体原因可能是我先使用pnpm安装的项目依赖，然后启动了neovim，但是发现启动项目时依赖丢失，所以重新使用`npm i`安装了依赖，这时候再打开neovim之后，eslint_d服务就回去node_modules/.pnpm文件夹下去找配置，很显然现在已经没有.pnpm文件夹了，需要直接在node_modules下面找,目前重启eslind_d可以解决