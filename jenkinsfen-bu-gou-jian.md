## jenkins分步构建

> 微服务分步构建效果国

### ![](/assets/import.png)lxyd\_jenkins

> 环境脚本独立项目, 存放微服务构建依赖的文件与脚本. 方便运维维护

### maven\_build

> lxyd\_jenkins是环境脚本独立项目, 存放微服务构建依赖的文件与脚本. 方便运维维护
>
> maven构建与及copy lxyd\_jenkins目录

![](/assets/1.png)![](/assets/2.png)![](/assets/3.png)

### docker\_build

```
cat > $WORKSPACE/../${project}/Dockerfile_rest_${project}-services << EOF
FROM 10.10.185.222/tomcat/jdk1.8:1.0.0
COPY images/bin/ucloud /data/apps/ucloud
COPY images/bin/livenessProbe /data/apps/livenessProbe
COPY images/bin/setenv.sh /data/apps/tomcat8080/bin/setenv.sh
RUN curl $JENKINS_URL/job/${project}/ws/${project}-rest/target/${project}-rest.war -o /data/apps/tomcat8080/webapps/${project}-rest.war && unzip /data/apps/tomcat8080/webapps/${project}-rest.war -d /data/apps/tomcat8080/webapps/ROOT &&  rm -f /data/apps/tomcat8080/webapps/${project}-rest.war && chown www. /data/apps/tomcat8080/webapps/ROOT -R
EOF

if [ "${project}" == "websocket" ];then
cat > $WORKSPACE/../${project}/Dockerfile_rest_${project}-services << EOF
FROM 10.10.185.222/jetty/jdk1.8:0.0.0
RUN rm -f /etc/localtime && cp -a /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ENV JAVA_OPTIONS "-javaagent:/data/apps/tingyun/tingyun-agent-java.jar"
WORKDIR /data/apps/jetty8080
COPY images/tingyun /data/apps/tingyun
COPY ${project}-rest/target/${project}-rest /data/apps/jetty8080/webapps/ROOT
RUN chown -R www:www  /data/apps/jetty8080/webapps/
USER www
EOF
fi

docker build -t 10.10.185.222/rest/${project}-services:${branches}_${TAG}_${BUILD} --no-cache=true --pull=true --file=../${project}/Dockerfile_rest_${project}-services ../${project}

if [ ! -d "../${project}/${project}-app/src" ] && [ -d "../${project}/${project}-api" ]; then  
docker tag 10.10.185.222/rest/${project}-services:${branches}_${TAG}_${BUILD} 10.10.185.222/soa/${project}-services:${branches}_${TAG}_${BUILD}
fi
```

### push\_images

```
if [ "$push" == "true" ];then
docker images |/bin/grep ${branches}_${TAG}_${BUILD} |awk '{print $1":"$2}'|xargs -i docker push {}
fi
```

### deploy\_dev

```
api_server=http://10.9.84.193:8080
resources=deployment
if [ "${env}" == "lifesense-qa" ]
then
  /data/apps/kubectl --server=${api_server} --namespace ${env} get ${resources} | awk "/^${project}-services/ {print \$1}" | while read f 
  do
    type=$(echo $f|awk -F- '{print $3}')
    /data/apps/kubectl --server=${api_server} --namespace ${env} set image deployment/${project}-services-${type} ${project}-services-${type}=10.10.185.222/$type/${project}-services:${branches}_${TAG}_${BUILD}
    /data/apps/kubectl --server=${api_server} --namespace ${env} rollout status deployment/${project}-services-${type}
  done
else
  /data/apps/kubectl --server=${api_server} --namespace ${env} get ${resources} | awk "/^${project}-services/ {print \$1}" | while read f 
  do
    type=$(echo $f|awk -F- '{print $3}')
    /data/apps/kubectl --server=${api_server} --namespace ${env} set image deployment/${project}-services-${type} ${project}-services-${type}=10.10.185.222/$type/${project}-services:${branches}_${TAG}_${BUILD}
    /data/apps/kubectl --server=${api_server} --namespace ${env} rollout status deployment/${project}-services-${type}
  done
fi
```



