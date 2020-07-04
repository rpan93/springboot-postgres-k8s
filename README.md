# springboot-postgres-k8s
springboot-postgres-k8s

1. mvn clean install -DskipTests=true
2. docker build -t springboot-postgres-k8s:1.0 .
3. cd src/main/resources
4. kubectl apply -f postgres-credentials.yml
5. kubectl get secrets
6. kubectl apply -f postgres-configmap.yml
7. kubectl get configmaps
8. kubectl apply -f postgres-deployment.yml
9. kubectl get deployments
10. kubectl get services
11. kubectl get pods
12. kubectl apply -f deployment.yml
13. kubectl get deployments
14. kubectl get pods
15. kubectl logs -f <springboot-... pod>
16. kubectl exec -it <postgress... pod> bash
17. psql -h postgres -U postgres <password: postgres>
18. check database: \l
19. \c employeedb
20. \dt
21. select * from employee;


Springboot2.3 and java 14
buildpacks
https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/html/#build-image

mvn spring-boot:build-image

docker run -it -p8080:8080 demo:0.0.1-SNAPSHOT

dive demo:0.0.1-SNAPSHOT

to repackage docker with layers:
https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/html/#repackage-layers

copy and paste this to pom
## <configuration>
	<layers>
			<enabled>true</enabled>
	</layers>
## </configuration>

mvn spring-boot:build-image

it will enable the layers

run application in jar mode:
java -jar target/xxxxx.jar

showing layers:
java -Djarmode=layertools -jar target/xxxx.jar 

java -Djarmode=layertools -jar target/xxxx.jar list 

java -Djarmode=layertools -jar target/xxxx.jar extract --destination target/tmp

tree target/tmp

Write a docker file:
https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#writing-the-dockerfile

# multi-stage docker file:

FROM adoptopenjdk:14-jre-hotspot as builder
WORKDIR application
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM adoptopenjdk:14-jre-hotspot
WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]


docker build .
## see the contents
dive <hash> 

look at the jar:

cd target
unzip <xxx>.jar
  
  cd BOOT-INF
  you will see the indexes 
  classpath.idx
  layers.idx
  classes
  lib
  
  cat classpath.idx
  
  you will see the order the jars will load
  
  cat layers.idx
  you will see the order of the layers will be loaded
  
  # customize the layers you can:
  
  <configuration>
		<layers>
				<enabled>true</enabled>
##			<configuration>${project.basedir}/src/layers.xml</configuration>
		</layers>
	</configuration>
  
  underneath src directory, create file layers.xml
  
  <layers xmlns="http://www.springframework.org/schema/boot/layers"
					  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
					  xsi:schemaLocation="http://www.springframework.org/schema/boot/layers
                      https://www.springframework.org/schema/boot/layers/layers-2.3.xsd">
	<application>
		<into layer="spring-boot-loader">
			<include>org/springframework/boot/loader/**</include>
		</into>
		<into layer="application" />
	</application>
  
	<dependencies>
		<into layer="snapshot-dependencies">
			<include>*:*:*SNAPSHOT</include>
		</into>
    <into layer="company-dependencies">
       <inlcude>com.fasterxml.jackson.core:*</include>
     </into>
		<into layer="dependencies" />
	</dependencies>
  
	<layerOrder>
		<layer>dependencies</layer>
		<layer>spring-boot-loader</layer>
		<layer>snapshot-dependencies</layer>
    <layer>company-dependencies</layer>
		<layer>application</layer>
    </layerOrder>
</layers>

mvn spring-boot:build-image
dive demo:0.0.1-SNAPSHOT

you will see the extra company layer


# graceful shutdown
make the endpoint slow
mvn spring-boot:run

in application.properties add:
server.shutdown=graceful
spring.lifecycle.timeout-per-shutdown-phase=20s

spring.main.cloud-platform=kubernetes
management.endpoints.web.exposure.include=*

it will wait till the request finishes.

http://localhost:8080/actuator/health  you will see 2 groups are available

liveness
readiness

in the controller,

inject:
private final ApplicationEventPublisher publisher;

@RequestMapping("/down")
public String down(){
  AvailabilityChangeEvent.publish(publisher, this, ReadinessState.REFUSING_TRAFFIC);
  return "down";
 }
 
 @RequestMapping("/up")
 public String up () {
 AvailabilityChangeEvent.publish(publisher, this, ReadinessState.ACCEPTING_TRAFFIC);
  return "up";
 }
 
 
http://localhost:8080/actuator/configprops



## Spring boot TDD
https://www.youtube.com/watch?v=s9vt6UJiHg4

https://github.com/sannidhi/tdd-with-springboot
