
# pyo3
ttps://pyo3.rs/v0.28.0/#usage

如果只是使用rust编写python库，使用pyo3就行，不过需要手动把编译好的文件复制到python包目录，有一点麻烦。

主要的库就是pyo3。

# maturin
https://www.maturin.rs/installation.html

仅仅是一个工具，方便编写python库，是对pyo3的包装。

安装：
```
pip install maturin
```

创建一个工程，里面已经内置一个函数，照着编写就行。
```
maturin new maturin_test
```

# pyo3初体验

## hello word

maturin new的工程，里面已经内置定义了一个函数，可以直接打包使用。

Cargo.toml:
```toml
[package]
name = "maturin_test"
version = "0.1.0"
edition = "2024"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
[lib]
name = "maturin_test"
crate-type = ["cdylib"]

[dependencies]
pyo3 = "0.28.1"
```

pyproject.toml:
```toml
[build-system]
requires = ["maturin>=1.12,<2.0"]
build-backend = "maturin"

[project]
name = "maturin_test"
requires-python = ">=3.8"
classifiers = [
    "Programming Language :: Rust",
    "Programming Language :: Python :: Implementation :: CPython",
    "Programming Language :: Python :: Implementation :: PyPy",
]
dynamic = ["version"]
```

lib.rs:
```rust
use pyo3::prelude::*;


/// A Python module implemented in Rust.
#[pymodule]
mod maturin_test {
    use pyo3::prelude::*;

    /// Formats the sum of two numbers as string.
    #[pyfunction]
    fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
        Ok((a + b).to_string())
    }

}
```

直接安装rust python库：
```
# 开发模式：构建并直接安装到当前虚拟环境
(base) D:\rustProject\maturin_test> maturin develop
```

在python中使用：
```py
In [1]: import maturin_test

In [2]: maturin_test
Out[2]: <module 'maturin_test' from 'D:\\apps\\miniconda3\\Lib\\site-packages\\maturin_test\\__init__.py'>

In [3]: maturin_test.sum_as_string
Out[3]: <function maturin_test.maturin_test.sum_as_string(a, b)>

In [4]: maturin_test.sum_as_string?
Signature: maturin_test.sum_as_string(a, b)
Docstring: Formats the sum of two numbers as string.
Type:      builtin_function_or_method

In [5]: maturin_test.sum_as_string(1, 2)
Out[5]: '3'
```

## arrow对象传递测试
使用rust编写主要是为了性能，使用arrow可以在不同的语言进程之间实现零拷贝，性能非常高。

直接把python的arrow对象(很多情况是c语言或者rust语言编写的库返回的对象)传入rust，打印测试一下。

Cargo.toml:
```toml
[package]
name = "maturin_test"
version = "0.1.0"
edition = "2024"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
[lib]
name = "maturin_test"
crate-type = ["cdylib"]

[dependencies]
pyo3 = "0.28.1"
pyo3-arrow = "0.16.0"
arrow = { version = "57.3.0", features = ["prettyprint"] }
```

lib.rs:
```rust
use pyo3::prelude::*;


/// A Python module implemented in Rust.
#[pymodule]
mod maturin_test {
    use pyo3::prelude::*;
    use pyo3_arrow::error::PyArrowResult;
    use pyo3_arrow::PyRecordBatchReader;
    use arrow::record_batch::RecordBatchReader;
    use arrow::util::pretty::print_batches;

    /// Formats the sum of two numbers as string.
    #[pyfunction]
    fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
        Ok((a + b).to_string())
    }

    /// Print the content of a pyarrow.RecordBatchReader (stream of batches)
    #[pyfunction]
    pub fn print_record_batch_reader<'py>(py: Python<'py>, reader: PyRecordBatchReader) -> PyArrowResult<()> {
        // 释放 GIL，让其他 Python 线程有机会并发（尤其是大表遍历时）
        py.detach(|| {
            // 零拷贝得到 Rust 的 RecordBatchReader
            let reader = reader.into_reader()?;
            print_record_batch_reader_internal(reader)
        })?;

        Ok(())
    }

    fn print_record_batch_reader_internal(mut reader: Box<dyn RecordBatchReader + Send>) -> PyArrowResult<()> {
        // 可选：先打印 schema（不消费数据）
        let schema = reader.schema();
        println!("Schema: {schema:?}");

        let mut batch_index = 0;
        let mut total_rows: usize = 0;

        // 逐 batch 迭代并打印
        while let Some(next_batch) = reader.next() {
            let batch = next_batch?;  // 处理 ArrowError -> PyErr

            batch_index += 1;
            total_rows += batch.num_rows();

            println!("\n=== Batch #{batch_index}  ({batch_rows} rows) ===",
                     batch_rows = batch.num_rows()
            );

            // 使用 arrow 自带的漂亮打印（带对齐、类型等）
            print_batches(&[batch])?
        }

        println!("\nFinished. Processed {batch_index} batches, total {total_rows} rows.");

        Ok(())
    }

}

```

测试：
```py
In [1]: import pyarrow as pa

In [2]: n_legs = pa.array([2, 2, 4, 4, 5, 100])
   ...: animals = pa.array(["Flamingo", "Parrot", "Dog", "Horse", "Brittle stars", "Centipede"])
   ...: names = ["n_legs", "animals"]
   ...: b = pa.record_batch([n_legs, animals], names=names)

In [3]: br = pa.RecordBatchReader.from_batches(b.schema, [b])
   ...: br
Out[3]: <pyarrow.lib.RecordBatchReader at 0x19e969c9290>

In [4]: import maturin_test

In [5]: maturin_test.print_record_batch_reader?
Signature: maturin_test.print_record_batch_reader(reader)
Docstring: Print the content of a pyarrow.RecordBatchReader (stream of batches)
Type:      builtin_function_or_method

In [6]: maturin_test.print_record_batch_reader(br)
Schema: Schema { fields: [Field { name: "n_legs", data_type: Int64, nullable: true }, Field { name: "animals", data_type: Utf8, nullable: true }], metadata: {} }

=== Batch #1  (6 rows) ===
+--------+---------------+
| n_legs | animals       |
+--------+---------------+
| 2      | Flamingo      |
| 2      | Parrot        |
| 4      | Dog           |
| 4      | Horse         |
| 5      | Brittle stars |
| 100    | Centipede     |
+--------+---------------+

Finished. Processed 1 batches, total 6 rows.
```

打印duckdb返回的arrow对象：
```py
In [7]: import duckdb

In [8]: ## python客户端

In [9]: con = duckdb.connect(config = {'threads': 2, 'memory_limit': '2g', 'allow_unsigned_extensions':True})

In [10]: con.execute("load 'D:/rustProject/extension-template-rs/extension-ci-tools/scripts/duckdb_extension_rs.duckdb_
    ...: extension'")
Out[10]: <_duckdb.DuckDBPyConnection at 0x19ef87b0930>

In [11]: maturin_test.print_record_batch_reader(con.sql("select 1 a, 'x' b").arrow())
Schema: Schema { fields: [Field { name: "a", data_type: Int32, nullable: true }, Field { name: "b", data_type: Utf8, nullable: true }], metadata: {} }

=== Batch #1  (1 rows) ===
+---+---+
| a | b |
+---+---+
| 1 | x |
+---+---+

Finished. Processed 1 batches, total 1 rows.

In [12]: maturin_test.print_record_batch_reader(con.sql("select * from faker(fields = ['{"name": "id", "type": "sequenc
    ...: e"}', '{"name": "timestamp", "type": "timestamp_sequence", "timestamp_type": "datetime"}', '{"name": "ip", "ty
    ...: pe": "ipv4"}'], rows = 100)").arrow())
  Cell In[12], line 1
    maturin_test.print_record_batch_reader(con.sql("select * from faker(fields = ['{"name": "id", "type": "sequence"}', '{"name": "timestamp", "type": "timestamp_sequence", "timestamp_type": "datetime"}', '{"name": "ip", "type": "ipv4"}'], rows = 100)").arrow())
                                                   ^
SyntaxError: invalid syntax. Perhaps you forgot a comma?


In [13]: maturin_test.print_record_batch_reader(con.sql("""select * from faker(fields = ['{"name": "id", "type": "seque
    ...: nce"}', '{"name": "timestamp", "type": "timestamp_sequence", "timestamp_type": "datetime"}', '{"name": "ip", "
    ...: type": "ipv4"}'], rows = 20)""").arrow())
Schema: Schema { fields: [Field { name: "id", data_type: Int64, nullable: true }, Field { name: "timestamp", data_type: Timestamp(Microsecond, None), nullable: true }, Field { name: "ip", data_type: Utf8, nullable: true }], metadata: {} }

=== Batch #1  (20 rows) ===
+----+---------------------+---------------+
| id | timestamp           | ip            |
+----+---------------------+---------------+
| 0  | 2026-02-16T09:14:07 | 192.168.0.109 |
| 1  | 2026-02-16T09:14:08 | 192.168.0.147 |
| 2  | 2026-02-16T09:14:09 | 192.168.0.146 |
| 3  | 2026-02-16T09:14:10 | 192.168.0.218 |
| 4  | 2026-02-16T09:14:11 | 192.168.0.158 |
| 5  | 2026-02-16T09:14:12 | 192.168.0.166 |
| 6  | 2026-02-16T09:14:13 | 192.168.0.11  |
| 7  | 2026-02-16T09:14:14 | 192.168.0.120 |
| 8  | 2026-02-16T09:14:15 | 192.168.0.58  |
| 9  | 2026-02-16T09:14:16 | 192.168.0.199 |
| 10 | 2026-02-16T09:14:17 | 192.168.0.228 |
| 11 | 2026-02-16T09:14:18 | 192.168.0.36  |
| 12 | 2026-02-16T09:14:19 | 192.168.0.69  |
| 13 | 2026-02-16T09:14:20 | 192.168.0.85  |
| 14 | 2026-02-16T09:14:21 | 192.168.0.13  |
| 15 | 2026-02-16T09:14:22 | 192.168.0.245 |
| 16 | 2026-02-16T09:14:23 | 192.168.0.103 |
| 17 | 2026-02-16T09:14:24 | 192.168.0.153 |
| 18 | 2026-02-16T09:14:25 | 192.168.0.13  |
| 19 | 2026-02-16T09:14:26 | 192.168.0.173 |
+----+---------------------+---------------+

Finished. Processed 1 batches, total 20 rows.
```


