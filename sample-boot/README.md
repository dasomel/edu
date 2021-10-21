배포1
=
## 1. kubernetes-maven-plugin 설정(pom.xml)
    <plugin>
        <groupId>org.eclipse.jkube</groupId>
        <artifactId>kubernetes-maven-plugin</artifactId>
        <version>1.0.2</version>
        <configuration>
            <images>
                <image>
                    <name>dasomel/egov-sampleboot:${timestamp}</name>
                    <build>
                        <from>fabric8/java-centos-openjdk8-jre</from>
                        <maintainer>dasomell@gmail.com</maintainer>
                        <assembly>
                            <inline>
                                <baseDirectory>/deployments</baseDirectory>
                            </inline>
                            <mode>dir</mode>
                            <targetDir>/deployments</targetDir>
                        </assembly>
                        <env>
                            <JAVA_LIB_DIR>/deployments</JAVA_LIB_DIR>
                            <CATALINA_OPTS>-Djava.security.egd=file:/dev/urandom</CATALINA_OPTS>
                        </env>
                    </build>
                </image>
            </images>
        </configuration>
        <executions>
            <execution>
                <id>jkube</id>
                <goals>
                    <goal>resource</goal>
                    <goal>build</goal>
                    <goal>push</goal>
                    <goal>deploy</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
## 2. /src/main/jkube/service.yml
    apiVersion: v1
    kind: Service
    metadata:
      name: sample-boot
    spec:
      selector:
        app: sample-boot
      ports:
        - protocol: TCP
          port: 8080
          targetPort: 8080
          nodePort: 30101
      type: NodePort
## 3. build & deploy 
    mvn clean package spring-boot:repackage k8s:build k8s:resource k8s:push k8s:apply
## 4. Confirm
   http://아이피:30101/maven/egovSampleList.do
   
배포2
=
## 1. local-ingress.yaml
##### kubectl apply -f ./k8s/local-ingress.yaml
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: opdc-ingress
    spec:
      rules:
        - host: sample.io
          http:
            paths:
              - path: /sample
                backend:
                  serviceName: egov-sample
                  servicePort: 8080
              - path: /maven
                backend:
                  serviceName: sample-boot
                  servicePort: 8080
        - host: sample2.io
          http:
            paths:
              - path: /gradle
                backend:
                  serviceName: sample-boot2
                  servicePort: 8080
## 2. Confirm
    http://127.0.0.1:30101/maven/egovSampleList.do
    http://sample.io/maven/egovSampleList.do