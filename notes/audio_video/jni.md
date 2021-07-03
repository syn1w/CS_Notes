JNI 是 Java Native Interface，使得在 JVM 中的 Java 程序可以和 native 应用或库互相调用。

使用场景：为了提升程序性能，把一些计算密集型的任务使用 C/C++ 语言实现；重用已有的 C/C++ 库

Android Studio 编译原生库的默认构建工具是 CMake，一些 legacy 的项目可能使用 ndk-build，所以新的 native 库，尽量使用 CMake 来进行构建



# 零、开发流程

编写声明了 `native` 方法的 Java 类，将 Java 类编译为 `.class` 字节码文件，使用 `javah -jni` 命令生成 `.h` 文件，编写 JNI C/C++ 实现代码，将 C/C++ 代码链接为动态库。

使用 Android Studio，Gradle + CMake 可以将上述步骤自动化。



# 一、基础内容

## 1. 数据类型

Java 中有两类数据类型：基本数据类型和引用数据类型。

Java 基本数据类型和 JNI/C/C++ 类型映射：

|  Java   |  native  |          C/C++          | Type signature |
| :-----: | :------: | :---------------------: | :------------: |
| boolean | jboolean |  unsigned char/uint8_t  |       Z        |
|  byte   |  jbyte   |   signed char/int8_t    |       B        |
|  char   |  jchar   | unsigned short/uint16_t |       C        |
|  short  |  jshort  |      short/int16_t      |       S        |
|   int   |   jint   |       int/int32_t       |       I        |
|  long   |  jlong   |    long long/int64_t    |       J        |
|  float  |  jfloat  |          float          |       F        |
| double  | jdouble  |         double          |       D        |
|  void   |          |          void           |       V        |



Java 引用类型和 JNI 类型映射：

|          Java          |        native        |     Type signature     |
| :--------------------: | :------------------: | :--------------------: |
|    java.lang.Class     |        jclass        |   Ljava/lang/Class;    |
|  java.lang.Throwable   |      jthrowable      | Ljava/lang/Throwable;  |
|    java.lang.String    |       jstring        |   Ljava/lang/String;   |
| java.lang.Object/Other |       jobject        |   L+fullClassName+;    |
|   java.lang.Object[]   |     jobjectArray     | `[X`  (X 元素类型签名) |
|    `builtin_type`[]    | j`builtin_type`Array |          ...           |
|      Other arrays      |        jarray        |          ...           |

Java 方法签名为 (params signature) return type signature

比如 `long foo(int n, String s, int[] arr);` 签名为 `(ILjava/lang/String;[I)J`； `void foo();` 签名为 `()V`





# 附录 参考资料

https://developer.android.com/ndk/guides?hl=zh-cn

https://developer.android.com/training/articles/perf-jni?hl=zh-cn

https://en.wikipedia.org/wiki/Java_Native_Interface

《The Java Native Interface: Programmer's Guide and Specification》