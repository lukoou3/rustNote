

## rust与java高效共享内存
jni调用是有开销的，如果把java的字节数组传入rust native处理然后再返回字节数组，java字节数组和rust的`Vec<u8>`转换会有复制开销，而且方法内存存在jni调用，最高效的方式是java侧使用堆外内存，java和rust通过地址访问共享内存。不过两边都需要自己手动管理释放内存。

### java生产数据rust读取

使用Unsafe直接申请堆外内存，返回地址，写入数据后把地址和大小传入rust，最后需要再java端释放内存。

### rust生产数据java读取
利用的into_raw_parts和from_raw_parts方法实现：
```rust
pub fn into_raw_parts(self) -> (*mut T, usize, usize);

pub unsafe fn from_raw_parts(ptr: *mut T, length: usize, capacity: usize) -> Self;
```

rust测伪代码，jni最新版本api有点变化，下面主要是思路。


```rust
/// 定义传递载体
#[repr(C)]
pub struct VecParts {
    pub ptr: *mut u8,
    pub len: usize,
    pub cap: usize,
}


/// 分配并导出 (into_raw_parts)
#[no_mangle]
pub extern "system" fn Java_NativeLib_allocateSharedVec(
    _env: JNIEnv,
    _class: JClass,
    size: jint,
) -> jlong {
    let mut v = vec![0u8; size as usize];
    // 填充测试数据
    v[0] = 88;

    // 核心：直接拆解 Vec
    // 该方法会消耗 Vec，不会触发析构函数（不会释放内存）
    let (ptr, len, cap) = v.into_raw_parts();

    let parts = Box::new(VecParts { ptr, len, cap });
    
    // 返回 VecParts 结构体的指针给 Java
    Box::into_raw(parts) as jlong
}

/// 回收并释放 (from_raw_parts)
#[no_mangle]
pub extern "system" fn Java_NativeLib_freeSharedVec(
    _env: JNIEnv,
    _class: JClass,
    parts_ptr: jlong,
) {
    if parts_ptr == 0 { return; }

    unsafe {
        // 1. 先拿回结构体所有权
        let parts = Box::from_raw(parts_ptr as *mut VecParts);

        // 2. 核心：使用原始三要素重建 Vec
        // 重建后，Vec 重新获得了这块内存的所有权
        let _v = Vec::from_raw_parts(parts.ptr, parts.len, parts.cap);

        // 3. 函数结束，_v 自动调用 drop，释放堆外内存
    }
}
```

java读取内存示例:
```java
import sun.misc.Unsafe;
import java.lang.reflect.Field;

public class NativeMemory {
    private static final Unsafe unsafe;

    static {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    // 对应 Rust 结构体的偏移量 (64位系统)
    private static final long PTR_OFFSET = 0;
    private static final long LEN_OFFSET = 8;
    private static final long CAP_OFFSET = 16;

    public native long allocateSharedVec(int size);
    public native void freeSharedVec(long handle);

    public void test() {
        // 1. 获取 Rust 返回的 VecParts 结构体指针
        long handle = allocateSharedVec(1024);

        try {
            // 2. 从结构体指针中读取字段信息
            // getLong(address) 相当于 C 语言的 *(long*)address
            long dataPtr = unsafe.getLong(handle + PTR_OFFSET);
            long length  = unsafe.getLong(handle + LEN_OFFSET);
            
            System.out.println("数据指针: " + dataPtr + ", 长度: " + length);

            // 3. 读取数据内容 (假设读取第一个字节)
            byte firstByte = unsafe.getByte(dataPtr);
            System.out.println("第一个字节的内容: " + firstByte);

            // 4. 写入数据内容
            unsafe.putByte(dataPtr + 1, (byte) 101);

        } finally {
            // 5. 释放内存
            freeSharedVec(handle);
        }
    }
}
```

这里有几个比较重要的地方说明：

* 访问VecParts结构体的指针也是为了减少jni的调用，因为java读取和rust使用内存时需要知道len和cap大小，如果不返回结构体指针的话只能再java中定义一个类然后在rust测调用jni创建实例返回，调用jni会有性能损失。  
* 释放内存时释放了VecParts结构体和Vec的内存，和生产数据对应。  
* `#[repr(C)]`保证字段顺序和对齐。


### 进一步优化

避免频繁的跨界调用：跨越 Java/Native 边界有固定开销。建议一次性在堆外内存中处理大块数据，而不是频繁调用微小的 native 方法。在 Java 侧攒够一批数据（比如 1000 条），一次性写入堆外内存，然后调用一次 Rust 进行批量处理。就比如arrow共享内存，一次批处理多行数据。

