```
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="off" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  <Service name="Catalina">
    <Executor
            name="tomcatThreadPool"
            namePrefix="catalina-exec-"
            maxThreads="500"
            minSpareThreads="30"
            maxIdleTime="60000"
            prestartminSpareThreads = "true"
            maxQueueSize = "100"
    />
    <Connector 
       executor="tomcatThreadPool"
       port="8080" 
       protocol="org.apache.coyote.http11.Http11Nio2Protocol" 
       connectionTimeout="20000" 
       maxConnections="10000" 
       redirectPort="8443" 
       enableLookups="false" 
       acceptCount="100" 
       maxPostSize="10485760"
       maxHttpHeaderSize="8192" 
       compression="on" 
       disableUploadTimeout="true" 
       compressionMinSize="2048" 
       acceptorThreadCount="2" 
       compressableMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript" 
       URIEncoding="utf-8"
    />
    <Engine name="Catalina" defaultHost="localhost">
      <Host name="localhost"  appBase="webapps"
            unpackWARs="false" autoDeploy="false">
      </Host>
    </Engine>
  </Service>
</Server>
```

---

```markdown
##hello
* 111111
```



