### 简介
> 一个完整的接口自动化测试平台需要支持接口的自动执行,自动生成测试报告,以及持续集成。Jmeter支持接口的
测试,Ant支持自动构建,而 Jenkins支持持续集成,所以三者组合在一起可以构成个功能完善的接口自动化测试平台。

### 环境准备
#### 环境依赖
-  JDK环境置
-  Jmeter安装
-  Ant安装环境变量配置
-  Jenkins安装

#### Ant简介
Apache Ant:是个将软件编译、测试、部署等步联系在一起加以自动化的工具,大多用于Java环境中的软件开发下载安装

下载地址:https://ant.apache.org/bindownload.cgi  
下载之后解压到任意文件路径,我这里是放到C盘根目录
。

环境变量配置：

-  ANT_HOME:  C:\apache-ant-1.10.5  

-  Path:  %ANT_HOME%\bin  

-  Classpath: %ANT_HOME%\lib

#### Jenkins简介
Jenkins是个开源软件项目,是基于Java开发的一种持集成工具,用于监控持续重复的工作,旨在提供一个开放易用的软件平台,使软件的持续集成变成可能

####  依赖文件配置
-  首先在 Jmeter 目录下面新建一个文件夹loadtest，文件夹名称不要使用下划线,空格等字符，并将脚本放置到该文件夹中。

- 将 Jmeter_extras文件中的ant-Jmeter-1.1.1.jar放到Ant中的lib文件夹中

- 将 Jmeter_extras文件中的 jmeter-results-detall-report_21.xsl, build.xml、 collapse.png、 expand.png放到ant目录中的bin目录下面。
####  build.xml配置
在Ant的bin目录中打开bui1d.xml文件,将文件内容修改为：
```
<?xml version="1.0" encoding="UTF-8"?>  
<project name="ant-jmeter-test" default="all" basedir=".">  
    <tstamp>  
        <format property="time" pattern="yyyyMMddhhmm" />  
    </tstamp>  
    <!-- 需要改成自己本地的 Jmeter 目录-->  
    <property name="jmeter.home" value="C:\Users\Administrator\Downloads\apache-jmeter-5.0\apache-jmeter-5.0" />  
    <!-- jmeter生成jtl格式的结果报告的路径-->  
    <property name="jmeter.result.jtl.dir" value="C:\Users\Administrator\Downloads\apache-jmeter-5.0\apache-jmeter-5.0\loadtest\jtl" />  
    <!-- jmeter生成html格式的结果报告的路径-->  
<property name="jmeter.result.html.dir" value="C:\Users\Administrator\Downloads\apache-jmeter-5.0\apache-jmeter-5.0\loadtest\html" />  
    <property name="ReportName" value="TestReport" />  
    <property name="jmeter.result.jtlName" value="${jmeter.result.jtl.dir}/${ReportName}${time}.jtl" />  
    <property name="jmeter.result.htmlName" value="${jmeter.result.html.dir}/${ReportName}${time}.html" />  
      
    <target name="all">  
        <antcall target="test" />  
        <antcall target="report" />  
    </target>  
      
    <target name="test">  
        <taskdef name="jmeter" classname="org.programmerplanet.ant.taskdefs.jmeter.JMeterTask" />  
        <jmeter jmeterhome="${jmeter.home}" resultlog="${jmeter.result.jtlName}">  

            <testplans dir="C:\Users\Administrator\Downloads\apache-jmeter-5.0\apache-jmeter-5.0\loadtest" includes="*.jmx" />  
        </jmeter>  
    </target>  
          
    <target name="report">  
        <xslt in="${jmeter.result.jtlName}"  
              out="${jmeter.result.htmlName}"  
              style="${jmeter.home}/extras/jmeter-results-detail-report_30.xsl" />  
          
        <!-- 因为上面生成报告的时候，不会将相关的图片也一起拷贝至目标目录，所以，需要手动拷贝 -->  
        <copy todir="${jmeter.result.html.dir}">  
            <fileset dir="${jmeter.home}/extras">  
                <include name="collapse.png" />  
                <include name="expand.png" />  
            </fileset>  
        </copy>  
    </target>  
</project>  
```
####  Ant 构建
执行以下命令进行构建
> ant -buildfile C:\apache-ant-1.10.5\bin\build.xml

![ant命令.jpg](https://upload-images.jianshu.io/upload_images/12273007-3ad08276092a1717.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

进入loadtest目录，可以看到生成如下文件:
![生成的文件.jpg](https://upload-images.jianshu.io/upload_images/12273007-e5d61ac18247a5bf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 报告优化
Jmeter默认生成的报告不是很详细,因比我们需要进行优化。

这里我们使用新的报告模板:
jmeter-results-detail-report_30.xsl

将模板放置到jmeter的extras目录下

![30xls文件.jpg](https://upload-images.jianshu.io/upload_images/12273007-e3ffa55b85c97ec8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

默认的服告模板是 Jmeter-results-detail-report_21
，打开build.xml将21改为
<condition property="style_version" value=_30">

最后执行即可生成最新的服告:样式如下.
可以清晰看到每个请求发送,响应内容

![修改后的报告.jpg](https://upload-images.jianshu.io/upload_images/12273007-71a96b2da121bfd8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####  集成到 Jenkins
在Jenkins中新建个任务：interface_test_wysh

在构建选项中选择 Invoke Ants，然后在 Build File输入 build. xml 文件路径，注意不要输入到Targets 里面去
了,需要点击高级选项后才可以显示 Build File

![jenkins配置.jpg](https://upload-images.jianshu.io/upload_images/12273007-7f17c33859cd53eb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

执行之后可以看到控制台输出和cmd的控制台输出是一样的

![控制台输出.jpg](https://upload-images.jianshu.io/upload_images/12273007-a052f8dd686031b6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

项目中涉及到的脚本及文件已上传至我的github中









