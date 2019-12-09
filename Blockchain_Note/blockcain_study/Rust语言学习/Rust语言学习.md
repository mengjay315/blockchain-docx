# Rust语言学习

**Rust**语言官网

https://www.rust-lang.org/tools/install

## 安装Rust

> curl https://sh.rustup.rs -sSf | sh

执行上述命令安装，我把终端代理打开了，结果最后都会出现如下错误：

![1561864176736](images/1561864176736.png)

以为是代理的问题，换了代理也不行，然后关闭了终端代理后正常下载：

> gaojie@admin1103:~$ curl https://sh.rustup.rs -sSf | sh
> info: downloading installer
>
> Welcome to Rust!
>
> This will download and install the official compiler for the Rust programming 
> language, and its package manager, Cargo.
>
> It will add the cargo, rustc, rustup and other commands to Cargo's bin 
> directory, located at:
>
>   /home/gaojie/.cargo/bin
>
> This path will then be added to your PATH environment variable by modifying the
> profile file located at:
>
>   /home/gaojie/.profile
>
> You can uninstall at any time with rustup self uninstall and these changes will
> be reverted.
>
> Current installation options:
>
>    default host triple: x86_64-unknown-linux-gnu
>      default toolchain: stable
>   modify PATH variable: yes
>
> 1) Proceed with installation (default)
> 2) Customize installation
> 3) Cancel installation
> >1
>
> info: syncing channel updates for 'stable-x86_64-unknown-linux-gnu'
> info: latest update on 2019-05-23, rust version 1.35.0 (3c235d560 2019-05-20)
> info: downloading component 'rustc'
>  88.4 MiB /  88.4 MiB (100 %)   4.2 MiB/s in 13s ETA:  0s
> info: downloading component 'rust-std'
>  59.1 MiB /  59.1 MiB (100 %)   4.5 MiB/s in 14s ETA:  0s
> info: downloading component 'cargo'
> info: downloading component 'rust-docs'
>  10.4 MiB /  10.4 MiB (100 %)   3.5 MiB/s in  3s ETA:  0s
> info: installing component 'rustc'
>  88.4 MiB /  88.4 MiB (100 %)  15.4 MiB/s in  5s ETA:  0s
> info: installing component 'rust-std'
>  59.1 MiB /  59.1 MiB (100 %)  17.9 MiB/s in  3s ETA:  0s
> info: installing component 'cargo'
> info: installing component 'rust-docs'
>  10.4 MiB /  10.4 MiB (100 %)   3.4 MiB/s in  4s ETA:  0s
> info: default toolchain set to 'stable'
>
>   stable installed - rustc 1.35.0 (3c235d560 2019-05-20)
>
>
> Rust is installed now. Great!
>
> To get started you need Cargo's bin directory ($HOME/.cargo/bin) in your PATH 
> environment variable. Next time you log in this will be done automatically.
>
> To configure your current shell run sour



**vim .bashrc，把#HOME/.cargo/bin添加到PATH中**

> export PATH="$HOME/.cargo/bin:$PATH"

**source .bashrc使环境变量生效**

**验证版本：rustc --version**

> rustc 1.35.0 (3c235d560 2019-05-20)

在vscode和Goland开发工具上都装了**Rust插件**

**Getting Statred文档**

https://doc.rust-lang.org/book/ch01-00-getting-started.html









