
## rdkafka引入

在`Cargo.toml`中添加如下依赖：
```toml
rdkafka = { version = "0.39.0", features = ["cmake-build", "sasl"]}
```

但是sasl在windows下编译失败，环境很难搞， 改为：
```toml
[target.'cfg(target_os = "windows")'.dependencies]
rdkafka = { version = "0.39.0", features = ["cmake-build"]}
[target.'cfg(target_os = "linux")'.dependencies]
rdkafka = { version = "0.39.0", features = ["cmake-build", "sasl"]}
```

sasl在linux中编译没有问题，但是把文件复制到其他机器上可能会报错：
```
libsasl2.so.3: cannot open shared object file: No such file or directory
```
因为默认libsasl2是动态链接的，看https://github.com/fede1024/rust-rdkafka/blob/master/rdkafka-sys/README.md#features文档描述：
The gssapi feature enables SASL GSSAPI support with Cyrus libsasl2. By default the system's libsasl2 is dynamically linked, but static linking of the version bundled with the sasl2-sys crate can be requested with the gssapi-vendored feature.

把sasl的feature改为gssapi-vendored， libsasl2会静态链接，生成的文件会变大一点：
```toml
rdkafka = { version = "0.39.0", features = ["cmake-build", "gssapi-vendored"]}
```

## windows snappy压缩问题
不知什么原因，windows中不会启用snappy, linux会自动检测环境启用snappy。
https://github.com/fede1024/rust-rdkafka/pull/586, https://github.com/fede1024/rust-rdkafka/issues/637

可以手动修改build.rs代码(有可能被修改过去)，或者自己fork下rust-rdkafka，修改build.rs，引入git上自己的库。

```rust
if !cmake_library_paths.is_empty() {
env::set_var("CMAKE_LIBRARY_PATH", cmake_library_paths.join(";"));
}

// 添加的代码
config.define("WITH_SNAPPY", "1"); // windows中启用snappy压缩
    
println!("Configuring and compiling librdkafka");
let dst = config.build();
```

```toml
# windows中不会启用snappy, https://github.com/fede1024/rust-rdkafka/pull/586, https://github.com/fede1024/rust-rdkafka/issues/637
[target.'cfg(target_os = "windows")'.dependencies]
rdkafka = { git = "https://github.com/lukoou3/rust-rdkafka.git", branch = "master", features = ["cmake-build"]}
[target.'cfg(target_os = "linux")'.dependencies]
rdkafka = { git = "https://github.com/lukoou3/rust-rdkafka.git", branch = "master", features = ["cmake-build", "gssapi-vendored"]}
```

## windows sasl环境问题
首先是依赖OpenSSL， 安装后，然后又依赖libsasl2，libsasl2的环境很难搞，所以没有搞，windows上本来就是测试，没必要使用sasl。



