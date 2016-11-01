# xld-petclinic-docker
A sample Java application that shows how to package war in a tomcat docker image and deploy them with XLDeploy

## Usage
* set the `docker` environment

```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.123:2376"
export DOCKER_CERT_PATH="/Users/vpartington/.docker/machine/machines/docker-machine-virtualbox-1"
export DOCKER_MACHINE_NAME="docker-machine-virtualbox-1"
# Run this command to configure your shell:
# eval "$(docker-machine env docker-machine-virtualbox-1)"
```

* run `mvn clean package`

To integrate with *XL Deploy*,
* start XL Deploy version 5.0 with the xld-docker-plugin defined here: https://github.com/vpartington/xld-docker-plugin
* run `mvn clean install`. This command `push`the images in the registry
  using a timestamp for version and `import` the XL Deploy DAR file in XL Deploy

![deployment with xld-docker-plugin](docker_deployment.png)


The XL Deploy manifest file for the application:

``` 
<udm.DeploymentPackage version="3.1-20160921-160321" application="PetDocker">
  <orchestrator>
    <value>parallel-by-deployment-group</value>
  </orchestrator>
  <deployables>
    <dockerx.Image name="ha-proxy">
      <tags />
      <image>eeacms/haproxy:1.6</image>
      <labels>
        <value>zone=front</value>
      </labels>
      <network>petnetwork</network>
      <dependencies>
        <value>petclinic</value>
      </dependencies>
      <volumesFrom />
      <registryHost>{{PROJECT_REGISTRY_HOST}}</registryHost>
      <ports>
        <dockerx.PortSpec name="ha-proxy/admin">
          <hostPort>1936</hostPort>
          <containerPort>1936</containerPort>
        </dockerx.PortSpec>
        <dockerx.PortSpec name="ha-proxy/web">
          <hostPort>80</hostPort>
          <containerPort>5000</containerPort>
        </dockerx.PortSpec>
      </ports>
      <links />
      <volumes />
      <variables>
        <dockerx.EnvironmentVariableSpec name="ha-proxy/BACKENDS">
          <value>petclinic:8080</value>
        </dockerx.EnvironmentVariableSpec>
        <dockerx.EnvironmentVariableSpec name="ha-proxy/constraint">
          <value>zone==front</value>
          <separator>:</separator>
        </dockerx.EnvironmentVariableSpec>
      </variables>
    </dockerx.Image>
    <smoketest.HttpRequestTest name="smoke test - ha">
      <url>http://{{FRONT_HOST_ADDRESS}}/petclinic/</url>
      <expectedResponseText>{{title}}</expectedResponseText>
      <headers />
    </smoketest.HttpRequestTest>
    <dockerx.Image name="petclinic-backend">
      <image>vpartington/petclinic-backend:1.1-20162109140249</image>
      <labels>
        <value>zone=back</value>
      </labels>
      <network>petnetwork</network>
      <dependencies />
      <volumesFrom />
      <registryHost>{{PROJECT_REGISTRY_HOST}}</registryHost>
      <ports />
      <links />
      <volumes />
      <variables>
        <dockerx.EnvironmentVariableSpec name="petclinic-backend/constraint">
          <value>zone==back</value>
          <separator>:</separator>
        </dockerx.EnvironmentVariableSpec>
      </variables>
    </dockerx.Image>
    <smoketest.HttpRequestTest name="smoke test">
      <tags />
      <url>http://{{BACK_HOST_ADDRESS}}:{{HOST_PORT}}/petclinic/</url>
      <expectedResponseText>{{title}}</expectedResponseText>
      <headers />
    </smoketest.HttpRequestTest>
    <sql.SqlScripts name="sql" file="sql/sql">
      <scanPlaceholders>true</scanPlaceholders>
    </sql.SqlScripts>
    <dockerx.Folder name="petclinic.config" file="petclinic.config/config">
      <volumeName>petclinic-config</volumeName>
      <containerName>petclinic</containerName>
      <containerPath>/application/properties</containerPath>
    </dockerx.Folder>
    <dockerx.NetworkSpec name="petnetwork">
      <driver>{{petnetwork}}</driver>
    </dockerx.NetworkSpec>
    <dockerx.Image name="petclinic">
      <image>vpartington/petclinic:3.1-20162109140249</image>
      <labels>
        <value>zone=back</value>
      </labels>
      <network>petnetwork</network>
      <dependencies>
        <value>petclinic-backend</value>
      </dependencies>
      <registryHost>{{PROJECT_REGISTRY_HOST}}</registryHost>
      <ports>
        <dockerx.PortSpec name="petclinic/exposed-port">
          <hostPort>{{HOST_PORT}}</hostPort>
          <containerPort>8080</containerPort>
        </dockerx.PortSpec>
      </ports>
      <links />
      <volumes>
        <dockerx.VolumeSpec name="petclinic/petclinic-config">
          <source>petclinic-config</source>
          <destination>/application/properties</destination>
        </dockerx.VolumeSpec>
      </volumes>
      <variables>
        <dockerx.EnvironmentVariableSpec name="petclinic/constraint">
          <value>zone==back</value>
          <separator>:</separator>
        </dockerx.EnvironmentVariableSpec>
        <dockerx.EnvironmentVariableSpec name="petclinic/loglevel">
          <value>{{LOGLEVEL}}</value>
        </dockerx.EnvironmentVariableSpec>
      </variables>
    </dockerx.Image>
  </deployables>
  <applicationDependencies />
</udm.DeploymentPackage>

```

Use the following dictionary to configure your deployed application (fake values!)
![configure petdocker](petdocker_dictionary.png)


TODO
* Add you docker-hub entry in your maven setting.xml file.
```

    <server>
      <id>docker-hub</id>
      <username>vpartington</username>
      <password>#########</password>
      <configuration>
        <email>vpartington@XXXXXXX</email>
      </configuration>
    </server>

```