1. 从Net3开始支持使用[kotlin-serialization](https://github.com/Kotlin/kotlinx.serialization)(以下简称ks)

1. 更多使用教程请阅读: [Kotlin最强解析库 - kotlin-serialization](https://juejin.cn/post/6963676982651387935)

<br>
## kotlin-serialization 特点

- kotlin官方发行
- 可配置性强
- 支持动态解析
- 自定义序列化器
- 支持ProtoBuf/CBOR/JSON等其他数据结构序列化
- 非空覆盖(即返回的Json字段为null则使用数据类默认值)
- 启用宽松模式, 允许Json和数据类字段匹配不一致
- 相对其他解析库他解决泛型擦除机制, 支持任何泛型, 可直接返回Map/List/Pair...

> ks的数据模型类都要求使用注解`@Serializable`(除非自定义解析过程), 父类和子类都需要使用 <br>
> 一般开发中都是使用[插件生成数据模型](model-generate.md), 所以这并不会增加工作量. 即使手写也只是一个注解, 但是可以带来默认值支持和更安全的数据解析

## 依赖


项目 build.gradle


```kotlin
classpath "org.jetbrains.kotlin:kotlin-serialization:$kotlin_version"
// 和Kotlin插件同一个版本号即可
```

module build.gradle

```kotlin
apply plugin: "kotlin-kapt"
apply plugin: 'kotlinx-serialization'
implementation "org.jetbrains.kotlinx:kotlinx-serialization-json:1.3.0"
```

## 配置转换器

这里使用Demo中的[SerializationConvert](https://github.com/liangjingkanji/Net/blob/master/sample/src/main/java/com/drake/net/sample/converter/SerializationConverter.kt)作演示.
如果你业务有特殊需要可以复制Demo中的转换器代码稍加修改

=== "全局配置"
    ```kotlin
    NetConfig.initialize("https://github.com/liangjingkanji/Net/", this) {
        setConverter(SerializationConvert())
        // ... 其他配置
    }
    ```
=== "单例配置"
    ```kotlin
    val userList = Get<List<UserModel>>("list") {
        converter = SerializationConvert() // 单例转换器, 此时会忽略全局转换器
    }.await()
    ```

## 使用

```kotlin
scopeNetLife {
    // 这里后端直接返回的Json数组
    val userList = Get<List<UserModel>>("list") {
        converter = SerializationConvert()
    }.await()

    tvFragment.text = userList[0].name
}
```

```kotlin
@Serializable
data class UserModel(var name: String, var age: Int, var height: Int)
```

> 具体解析返回的JSON中的某个字段请在转换器里面自定, 其注意如果存在父类, 父类和子类都需要使用`@Serializable`注解修饰 <br>
如果想详细了解KS, 请阅读文章: [Kotlin最强解析库 - kotlin-serialization](https://juejin.cn/post/6963676982651387935)

## 非空覆盖

当Json中存在字段值为null或者Json和数据类字段不匹配, 为了避免解析失败或者想要Json字段为null时使用数据类字段默认值则可以开启`非空覆盖`功能,
这时将使用默认值, 如果没有写默认值依然将导致解析失败

Json配置
```kotlin
val jsonDecoder = Json {
    ignoreUnknownKeys = true // JSON和数据模型字段可以不匹配
    coerceInputValues = true // 如果JSON字段是Null则使用默认值
}
```

数据类使用默认值
```kotlin
@Serializable
data class Data(var name:String = "", var age:Int = 0)
```

手动写默认值太麻烦, 推荐使用插件生成默认值

<img src="https://i.loli.net/2021/11/19/YahlbxO9dWf1PN5.png" width="600"/>

插件具体配置使用请查看: [数据模型生成插件](model-generate.md)

