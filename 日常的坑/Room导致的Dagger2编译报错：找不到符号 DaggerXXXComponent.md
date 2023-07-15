Dagger2编译一直报错：

```
错误: 找不到符号
import com.bjwgh.motorcycle.exam.di.component.DaggerMineComponent;
```

几乎每个Component类都找不到，然后选择 `Run with --stacktrace`，报错是找不到 Mac/aarch64 的库：

```
Caused by: java.lang.Exception: No native library is found for os.name=Mac and os.arch=aarch64. path=/org/sqlite/native/Mac/aarch64
	at org.sqlite.SQLiteJDBCLoader.loadSQLiteNativeLibrary(SQLiteJDBCLoader.java:333)
	at org.sqlite.SQLiteJDBCLoader.initialize(SQLiteJDBCLoader.java:64)
	at androidx.room.verifier.DatabaseVerifier.<clinit>(DatabaseVerifier.kt:68)
	... 151 more
```

解决方案：

**升级room版本到2.4.0-alpha03或以上**