【Maven工程war包依赖基础war包的解决方案及坑点】


一、业务场景
传统的SSM项目一般都为war包部署，多模块的项目一般都是将模块打包成jar包依赖进web工程中，但是对于作为基础项目或者分模块的web项目来说，打成war包对静态资源的访问就不太方便；这里介绍一下通过Maven WAR Plugin的<overlays>解决这个问题。这对于没有上微服务的项目来说应该是个不错的解决方案，将通用的常规化的功能抽到base.war中，而其他类似项目依赖base.war作为基础项目的一个模块并对其做小部分定制化的修改即可，大大提高了同类型项目的交付周期。

二、overlay简介
Overlays(覆盖)主要用于跨多Web项目间共享公共资源。它能够在目标WAR本身覆盖除了原生WAR构件以外的所有文件，并在WEB-INF/lib目录下收集原生WAR项目的依赖。

<overlay>元素包含有下列子元素：
id - overlay id。如果你不提供的话，WAR插件将自动生成一个。
groupId - 配置你想要覆盖的groupId。
artifactId – 配置你想要覆盖的构件的artifactId。
type – 配置你想要覆盖的构件类型。默认值是：war。
classifier – 如果有多个构件匹配当前的groupId/artifactId，那么你需要配置构件的classifier以明确覆盖(classifier：该元素用来帮助定义构建输出的一些附属构件)。
includes - 要包含的文件。默认情况下，所有文件都能被包含。
excludes – 要排除的文件。默认情况下，在META – INF目录是被排除在外的。
targetPath - 在webapp结构的目标相对路径，当然这只在覆盖类型为war时才有效。默认情况下，覆盖的内容都追加在webapp的根节点下。
skip – 当设置为true时，跳过本次覆盖。默认值是：false。

三、实现方式
基础项目：base.war，具体开发项目：a.war
如果在a项目中没有对base项目种具体方法的引用，则只需要在a项目中添加下面的build节点。

<build>
    <finalName>a</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <overlays>
                    <overlay>
                        <groupId>ff</groupId>
                        <artifactId>base</artifactId>
                    </overlay>
                </overlays>
            </configuration>
        </plugin>
    </plugins>
</build>
这样便可以实现将base项目中同包同名的文件进行覆盖，但是这也仅仅局限于静态资源，如果我们需要引用Java的某各类，那么在打包的时候会发现出现找不到java类的报错。
问题原因：
由于开发中我们只需要覆盖部分文件，方法调用依然是调用base中定义的方法，但根据Java规范，classpath不能指定WAR文件。这就意味着在开发工具中调试时，项目可以正常运行；但是在编译时，a项目无法访问base项目中定义的类，所以在a项目中，我们不能像常规类组件那样扩展或使用base定义的类。要解决这一问题，我们必须在base中重新设置maven-war-plugin的一项缺省配置，将classes打包成jar包引入到a中，这样才能使得a编译打包成功。相关pom配置。
在base项目中添加maven-war-plugin插件：

<build>
    <finalName>base</finalName>
    <plugins>
        <plugin>
            <artifactId>maven-war-plugin</artifactId>
            <configuration>
                <!-- 把class打包jar作为附件 -->
                <attachClasses>true</attachClasses>
            </configuration>
        </plugin>
</plugins>
</build>
在a项目中引入上面打包的jar文件：

<dependencies>
    <dependency>
        <groupId>ff</groupId>
        <artifactId>base</artifactId>
        <version>1.0.0</version>
        <type>jar</type>
        <classifier>classes</classifier>
        <scope>provided</scope>
    </dependency>
</dependencies>
这里还要注意下scope不能随意修改，这个jar我们只是编译需要，不需要放到最终打包的war文件中，如果放入最终的war包中，启动时Mybatis会报错（具体原因是Mapper文件的namespace重名了，因为项目中有一份，jar包中还有一份）。

<scope>可以使用5个值：

compile，缺省值，适用于所有阶段，会随着项目一起发布。
provided，类似compile，期望JDK、容器或使用者会提供这个依赖。如servlet.jar。
runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段。
test，只在测试时使用，用于编译和运行测试代码。不会随项目发布。
system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。
依赖范围控制哪些依赖在哪些classpath 中可用，哪些依赖包含在一个应用中。让我们详细看一下每一种范围：

compile （编译范围）

compile是默认的范围；如果没有提供一个范围，那该依赖的范围就是编译范围。编译范围依赖在所有的classpath 中可用，同时它们也会被打包。

provided （已提供范围）

provided 依赖只有在当JDK 或者一个容器已提供该依赖之后才使用。例如， 如果你开发了一个web 应用，你可能在编译 classpath 中需要可用的Servlet API 来编译一个servlet，但是你不会想要在打包好的WAR 中包含这个Servlet API；这个Servlet API JAR 由你的应用服务器或者servlet 容器提供。已提供范围的依赖在编译classpath （不是运行时）可用。它们不是传递性的，也不会被打包。

runtime （运行时范围）

runtime 依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC
驱动实现。

test （测试范围）

test范围依赖 在一般的编译和运行时都不需要，它们只有在测试编译和测试运行阶段可用。

system （系统范围）

system范围依赖与provided 类似，但是你必须显式的提供一个对于本地系统中JAR 文件的路径。这么做是为了允许基于本地对象编译，而这些对象是系统类库的一部分。这样的构件应该是一直可用的，Maven 也不会在仓库中去寻找它。如果你将一个依赖范围设置成系统范围，你必须同时提供一个 systemPath 元素。注意该范围是不推荐使用的（你应该一直尽量去从公共或定制的 Maven 仓库中引用依赖）。


【链接】：https://www.jianshu.com/p/cde7795f37c5
