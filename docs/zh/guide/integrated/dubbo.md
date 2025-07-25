# Dubbo   <Badge type="tip" text="^1.8.7" />
## 简介

`smart-doc`从`1.8.7`版本开始支持`Dubbo API`文档的生成，下面介绍如何利用`smart-doc`工具来生成`Dubbo`的`RPC`内部接口文档。
## Dubbo文档集成
`smart-doc`本着使用简单的原则开发了`maven`插件和`gradle`，通过插件来降低`smart-doc`的集成难度和去除依赖侵入性。
您可以根据自己使用的依赖构建管理工具来选择相关的插件，下面以使用`smart-doc-maven-plugin`插件集成`smart-doc`生成`dubbo`为例。
当然集成`smart-doc`来生成`Dubbo RPC`接口文档你有两种可选方式：

- 使用`smart-doc`扫描`dubbo api`模块
- 使用`smart-doc`扫描`dubbo provider`模块

下面来看下集成方式。
### 添加插件
在你的`dubbo api`或者或者是`dubbo provider`模块中添加`smart-doc-maven-plugin`。当然你只需要选中一种方式即可
```xml
<plugin>
    <groupId>com.ly.smart-doc</groupId>
    <artifactId>smart-doc-maven-plugin</artifactId>
    <version>[最新版本]</version>
    <configuration>
        <!--指定生成文档的使用的配置文件,配置文件放在自己的项目中-->
        <configFile>./src/main/resources/smart-doc.json</configFile>
        <!--指定项目名称-->
        <projectName>测试</projectName>
    </configuration>
    <executions>
        <execution>
            <!--如果不需要在执行编译时启动smart-doc，则将phase注释掉-->
            <phase>compile</phase>
            <goals>
                <!--根据需求选择配置下面的任一goal-->
                <goal>rpc-html</goal>
                <goal>rpc-adoc</goal>
                <goal>rpc-markdown</goal>
                <goal>rpc-word</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
### 添加smart-doc所需配置文件
在`Dubbo API`或者或者是`dubbo provider`模块`reources`中添加`smart-doc.json`配置文件

```json
{
  "isStrict": false, //是否开启严格模式
  "allInOne": true,  //是否将文档合并到一个文件中，一般推荐为true
  "outPath": "D://md2", //指定文档的输出路径
  "projectName": "smart-doc",//配置自己的项目名称
  "rpcApiDependencies":[{ // 项目开放的dubbo api接口模块依赖，配置后输出到文档方便使用者集成
      "artifactId":"SpringBoot2-Dubbo-Api",
      "groupId":"com.demo",
      "version":"1.0.0"
  }],
  "rpcConsumerConfig":"src/main/resources/consumer-example.conf"//文档中添加dubbo consumer集成配置，用于方便集成方可以快速集成
}
```
关于`smart-doc`如果你生成文档需要更详细的配置请常看官方其它文档 [配置项](/zh/guide/advanced/config)

**rpcConsumerConfig：**

如果下你想让`dubbo consumer`集成更加快速，你可以将集成配置示例`consumer-example.conf`中，
`smart-doc`会将该示例直接输出到文档中。

```yaml
dubbo:
  registry:
    protocol: zookeeper
    address:  ${zookeeper.adrress}
    id: my-registry
  scan:
    base-packages: com.iflytek.demo.dubbo
  application:
    name: dubbo-consumer
```
## Dubbo接口扫描
上面提到了`smart-doc`支持单独去扫描`dubbo api`或者`dubbo provider`。在
扫描原理是主要通过识别`smart-doc`官方自定义`@dubbo`注释`tag`或`Dubbo`官方的`@service`注解。

### 扫描Dubbo api
`dubbo api`通常都是很简洁的`Dubbo`接口定义，如果你需要让`smart-doc`扫描到`Dubbo`接口，那么需要加上`@dubbo`注释`tag`。示例如下：

```java
/**
 * 用户操作
 *
 * @author yu 2019/4/22.
 * @author zhangsan 2019/4/22.
 * @version 1.0.0
 * @dubbo
 */
public interface UserService {

    /**
     * 查询所有用户
     *
     * @return
     */
    List<User> listOfUser();

    /**
     * 根据用户id查询
     *
     * @param userId
     * @return
     */
    User getById(String userId);
}
```
针对一些特殊的用户，可能需要指定`Dubbo`服务接口的版本`@version`、服务名称`@service`、接口协议`@protocol`。在`smart-doc`中也可以比较方便的指定。
示例如下：
```java
/**
 * Dubbo Interface
 * @author yusun
 * @dubbo
 * @service test
 * @protocol dubbo
 * @version 2.0.0
 */
public interface DubboInterface {

  /**
   * Say Hello
   * @param word
   * @return
   */
  String sayHello(String word);
}
```
- `@version`指定`Dubbo`服务的接口版本
- `@service`指定`Dubbo`服务名称
- `@protocol`指定`Dubbo`服务协议类型
#### 扫描Dubbo provider
如果想通过`dubbo provider`生成`RPC`接口文档的情况，你不需要加任何的其他注释`tag`，`smart-doc`自动扫描`@service`注解完成。

```java
/**
 * @author yu 2019/4/22.
 */
@Service
public class UserServiceImpl implements UserService {

    private static Map<String,User> userMap = new HashMap<>();

    static {
        userMap.put("1",new User()
                .setUid(UUIDUtil.getUuid32())
                .setName("zhangsan")
                .setAddress("四川成都")
        );
    }
    
    /**
     * 获取用户
     * @param userId
     * @return
     */
    @Override
    public User getById(String userId) {
        return userMap.get(userId);
    }

    /**
     * 获取用户
     * @return
     */
    @Override
    public List<User> listOfUser() {
        return userMap.values().stream().collect(Collectors.toList());
    }
}
```
## 生成操作
直接通过`mvn`命令运行插件的文档生成命令或者在`IDEA`中直接单击插件的可视化命令即可。

![maven-smart-doc](/assets/20200705230512435.png)

运行`rpc-html`等就能生成`Dubbo RPC`文档
