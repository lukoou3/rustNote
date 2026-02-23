
## ADBC
ADBC: Arrow Database Connectivity

很多数据库支持通过 Apache Arrow Flight SQL 协议进行连接，通过arrow协议读取数据性能高。

我要使用python和rust使用ADBC读取数据，发现这两个都是调用的c语言编写的动态库，c的动态库官方似乎没有提供windows版本的，python安装的adbc_driver_flightsql里面的库是linux的.so文件，需要自己编译。

## 编译windows adbc_driver_flightsql.dll
问下ai就行，使用MSYS2（ucrt64 shell）环境编译。

```sh
# 安装依赖
pacman -S mingw-w64-ucrt-x86_64-cmake mingw-w64-ucrt-x86_64-gcc mingw-w64-ucrt-x86_64-arrow mingw-w64-ucrt-x86_64-grpc mingw-w64-ucrt-x86_64-protobuf mingw-w64-ucrt-x86_64-boost mingw-w64-ucrt-x86_64-ninja
# 也是需要go语言环境
pacman -S mingw-w64-ucrt-x86_64-go
 go version
/ucrt64/lib/go
echo 'export GOROOT=/ucrt64/lib/go' >> ~/.bashrc
source ~/.bashrc
go version
export GOPROXY=https://goproxy.cn,direct
# 也是需要git环境
pacman -S git
git --version

# clone 仓库：
git clone https://github.com/apache/arrow-adbc.git
cd arrow-adbc
# 构建
mkdir build && cd build
cmake ../c -G Ninja   -DCMAKE_BUILD_TYPE=Release   -DADBC_DRIVER_FLIGHTSQL=ON   -DADBC_BUILD_SHARED=ON   -DADBC_BUILD_STATIC=OFF   -DCMAKE_INSTALL_PREFIX=install

ninja -j8

ninja install

# 查看编译后的文件
ls -l install/bin/
ls -l install/lib/
```

## python adbc读取测试
https://docs.starrocks.io/zh/docs/unloading/arrow_flight/#%E6%AD%A5%E9%AA%A4-1-%E5%AE%89%E8%A3%85%E5%BA%93

adbc_driver_flightsql库windows版本默认不行，看下`D:\apps\miniconda3\Lib\site-packages\adbc_driver_flightsql\__init__.py`的_driver_path函数：
```py
@functools.lru_cache
def _driver_path() -> str:
    import pathlib
    import sys

    import importlib_resources

    driver = "adbc_driver_flightsql"

    # Wheels bundle the shared library
    root = importlib_resources.files(driver)
    # The filename is always the same regardless of platform
    entrypoint = root.joinpath(f"lib{driver}.so")
    if entrypoint.is_file():
        return str(entrypoint)

    # Search sys.prefix + '/lib' (Unix, Conda on Unix)
    root = pathlib.Path(sys.prefix)
    for filename in (f"lib{driver}.so", f"lib{driver}.dylib"):
        entrypoint = root.joinpath("lib", filename)
        if entrypoint.is_file():
            return str(entrypoint)

    # Conda on Windows
    entrypoint = root.joinpath("bin", f"{driver}.dll")
    print(entrypoint)
    if entrypoint.is_file():
        return str(entrypoint)

    # Let the driver manager fall back to (DY)LD_LIBRARY_PATH/PATH
    # (It will insert 'lib', 'so', etc. as needed)
    return driver
```

把编译后的adbc_driver_flightsql.dll文件放入`D:/apps/miniconda3/bin/adbc_driver_flightsql.dll`，可以正常调用库代码。

```py
import adbc_driver_manager
import adbc_driver_flightsql.dbapi as flight_sql

FE_HOST = "127.0.0.1"
FE_PORT = 9408

conn = flight_sql.connect(
    uri=f"grpc://{FE_HOST}:{FE_PORT}",
    db_kwargs={
        adbc_driver_manager.DatabaseOptions.USERNAME.value: "root",
        adbc_driver_manager.DatabaseOptions.PASSWORD.value: "",
    }
)
cursor = conn.cursor()
```

## rust adbc读取测试

rust在windows中直接使用flight-sql-client、adbc-driver-flightsql库build不通过，里面会下载c语言adbc-driver-flightsqlwindows的动态库，但是下载找不到到文件，所以需要自己编译文件。

直接使用底层库，加载dll文件即可，本来flight-sql-client、adbc-driver-flightsql库封装的就不多，读取完就是arrow对象。

```
adbc_driver_manager = "0.22.0"
adbc_core = "0.22.0"
```

```rust
fn test_query() -> Result<(), Box<dyn std::error::Error>> {
    let endpoint = "grpc://localhost:9408";
    let mut driver = ManagedDriver::load_dynamic_from_filename(
        "D:/apps/miniconda3/bin/adbc_driver_flightsql.dll",
        None,
        AdbcVersion::default(),
    )?;
    let database = driver.new_database_with_opts([
        (OptionDatabase::Uri, OptionValue::from(endpoint)),
        (OptionDatabase::Username, OptionValue::from("root")),
        (OptionDatabase::Password, OptionValue::from("")),
    ])?;
    let mut connection = database.new_connection()?;
    let mut statement = connection.new_statement()?;
    statement.set_sql_query("select * from test.test_types limit 0")?;
    //statement.set_sql_query("select count(1) cnt, max(timestamp) timestamp_max from test.test_types")?;
    let output = statement.execute()?;
    let schema = output.schema();
    //let output: Result<Vec<RecordBatch>, _> = output.collect()?;
    //let output = concat_batches(&schema, &output?)?;
    println!("{:?}", schema);
    for batch in output {
        let batch = batch?;
        println!("{:?}", batch);
    }

    Ok(())
}
```





