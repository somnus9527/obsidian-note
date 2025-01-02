---
tags:
  - 知识笔记/前端框架
---
### 结论

_可以实现,但是代价很大,因为引用三方包如果和appicaionId挂钩,需要多重配置_

##### 前置背景

目前多环境打包采用的是多入口的方式_一个环境一个入口,对应一套配置_

##### 需求背景

要求不同环境apk在统一个设备上共存

##### 核心问题

需要apk共存,最核心的问题就是:**针对不同的环境为apk  
配置不同的applicationId,因为系统识别app是不是同一个就是根据applicationId来识别的**

##### 基本解决方式

1. 首先为flutter打包增加打包参数 `flutter build apk -t lib/entry.dart --release --dart-define=key=value --dart-define=key=value`
2. android/app/build.gradle文件获取dart变量,并定义resValue

```
def environmentValues = [ ValueKey1: "value1", ValueKey2: "value2" ]
if (project.hasProperty('dart-defines')) {
    environmentValues = environmentValues + project.property('dart-defines')
            .split(',')
            .collectEntries { entry ->
                def pair = new String(entry.decodeBase64(), 'UTF-8').split('=')
                [(pair.first()): pair.last()]
            }
}

// 接着配置config中的applicationId

xxxxxConfig {
    applicationId: environmentValues.ValueKey1
    
    // ....省略
    
    // 或许可以添加额外的resValue
    resValue "string", "key", environmentValues.ValueKey2 }"
}
```

3. 一般到第二步就完成了,但是这边遇到了一个问题,两个app安装时提示使用了同一个内容提供者,这种情况后来发现**由于三方插件在定义 android.authorities=xxxxx时,写死了applicationId而不是使用的${applicationId}xxx的方式,这种直接将三方插件改成动态的applicationId即可**

##### 额外内容
按上述操作,app可以同时安装.但是也出现一些问题

- 由于app是同一个只是环境不同,所以两个app长得一样,那么就需要改android.label为动态的,拼上之前的app_name和flutter带入的打包环境参数.但是这样就会造成label值不再是动态的,比如语言切换之后.这里附上一个获取资源文件值的方式

```
def stringsFile = file("./src/main/res/values/string.xml")
def appName = new XmlParser().parse(stringsFile).string.find { it.@name.equals 'app_name' }.text()
```

- 最恶心的点在于,三方插件很多是根据applicationId来挂钩的,一旦applicationId变更之后,很多三方插件不可用,比如高德api,这种需要配置多份(_比如高德申请两个key,根据环境选择不同的配置_)来解决,但是这样依赖代价就非常大了,远不如apk提供切换环境能力来的简单