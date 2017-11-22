# tomcat部署

参考  [https://github.com/judasn/Linux-Tutorial/blob/master/Tomcat-Install-And-Settings.md](https://github.com/judasn/Linux-Tutorial/blob/master/Tomcat-Install-And-Settings.md)

### 支持apr

```
yum -y install epel-release
yum -y clean all
yum -y install tomcat-native
```

### JVM参数优化

```
-XX:SurvivorRatio=10 -XX:MaxTenuringThreshold=15 -XX:NewRatio=2 -XX:+DisableExplicitGC"
```

### 配置优化

```
<?xml version='1.0' encoding='utf-8'?>
<Server port="-1" shutdown="">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="off" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  <Service name="Catalina">
    <Executor name="tomcatThreadPool8080"
              namePrefix="tomcatThreadPool8080-exec-"
              maxThreads="512"
              minSpareThreads="30"
              prestartminSpareThreads = "true"
              maxQueueSize = "100"  
              maxIdleTime="60000"/>
    <Connector port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol"
               connectionTimeout="60000"
               keepAliveTimeout="60000"
               enableLookups="false"
               maxKeepAliveRequests="384"
               acceptCount="4096"
               acceptorThreadCount="2"
               server="lx"
               compression="on"
               compressionMinSize="2048"
               compressableMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript" 
               URIEncoding="UTF-8"
               executor="tomcatThreadPool8080" />
    <Engine name="Catalina" defaultHost="localhost">
      <Host name="localhost"  appBase="webapps" unpackWARs="false" autoDeploy="false">
      </Host>
    </Engine>
  </Service>
</Server>
```

### 禁止TLD扫描

修改日志配置

```
#conf/logging.properties 
org.apache.catalina.core.ContainerBase.[Catalina].[localhost].level = INFO
org.apache.jasper.compiler.TldLocationsCache.level = FIN
```

收jar包列表,加入到catalina.properties配置中

```
commons-exec-*.jar,\
config-types-*.jar,\
conf-support-*.jar,\
curator-client-*.jar,\
curator-framework-*.jar,\
curator-recipes-*.jar,\
curator-test-*.jar,\
disconf-client-*.jar,\
disconf-core-*.jar,\
druid-*.jar,\
dubbo-*.jar,\
dubbo-support-*.jar,\
ehcache-core-*.jar,\
```

解决"At least one JAR was scanned for TLDs yet contained no TLDs"问题

[http://mov-webhobo.iteye.com/blog/1939655](http://mov-webhobo.iteye.com/blog/1939655)

### 容器化部署

#### jdk环境

```
FROM centos:centos7
MAINTAINER 77749770@qq.com

ARG release=1.8.0_131
ARG release_major=8u131
ARG release_minor=b11
ARG release_key=d54c1d3a095b4ff2b6607d096fa80163
ARG base_url=http://download.oracle.com/otn-pub/java/
ARG jdk_download_url=${base_url}jdk/${release_major}-${release_minor}/${release_key}/jdk-${release_major}-linux-x64.rpm
ARG jce_policy_download_url=${base_url}jce/8/jce_policy-8.zip

ENV LC_ALL=en_US.UTF-8 \
    JAVA_HOME=/usr/java/jdk${release}

RUN yum -y update && \
yum -y install curl wget tar zip unzip ca-certificates libgcc libstdc++ tzdata-java zlib

RUN wget -q -O /tmp/jdk.rpm --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" "$jdk_download_url" \
    && rpm -iv /tmp/jdk.rpm \
    && rm /tmp/jdk.rpm

RUN wget -q -O /tmp/jce_policy.zip --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" "$jce_policy_download_url" \
    && unzip -oj -d ${JAVA_HOME}/jre/lib/security /tmp/jce_policy.zip \*/\*.jar \
    && rm /tmp/jce_policy.zip \
    && echo LANG=en_US.UTF-8 > /etc/sysconfig/i18n

ENV PATH=${JAVA_HOME}/bin:$PATH
```

构建脚本 [https://github.com/caiwenhao/image/tree/master/tomcat/lifesense-v1](https://github.com/caiwenhao/image/tree/master/tomcat/lifesense-v1)

验证:

../sport/Dockerfile\_rest\_sport-services

```
FROM reg.lifesense.com/lifesense/tomcat:lifesense-v1
COPY images/bin/ucloud /data/apps/ucloud
COPY images/bin/livenessProbe /data/apps/livenessProbe
COPY images/tingyun /opt/tomcat/tingyun
COPY sport-rest/target/sport-rest /opt/tomcat/webapps/ROOT
```

```
cd /data/jenkins_home_new/workspace/docker_build
docker build -t sport:1 --no-cache=true --pull=true --file=../sport/Dockerfile_rest_sport-services ../sport
docker run -e DISCONF_ENV=online -ti sport:1 /bin/bash
```

### jvm调优参考

http://www.cnblogs.com/markjiao/p/5368682.html



