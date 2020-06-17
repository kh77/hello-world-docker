# Hello World Rest API

### Building an Image

1. Build a Jar - /target/hello-world-docker.jar
2. Setup the Prerequisites for Running the JAR - openjdk:8-jdk-alpine
3. Copy the jar
4. Run the jar

### Docker Commands - Creating Image Manually

- docker run -dit openjdk:8-jdk-alpine
- docker container exec naughty_knuth ls /tmp
- docker container cp target/hello-world-docker.jar naughty_knuth:/tmp
- docker container exec naughty_knuth ls /tmp
- docker container commit naughty_knuth kh77/hello-world-docker:manual1
- docker run kh77/hello-world-docker:manual1
- docker container ls
- docker container commit --change='CMD ["java","-jar","/tmp/hello-world-docker.jar"]' naughty_knuth kh77/hello-world-docker:manual2
- docker run -p 8080:8080 kh77/hello-world-docker:manual2


### Running the Application

Run RestfulWebServicesApplication as a Java Application.

- http://localhost:8080/hello-world

```txt
Hello World
```

- http://localhost:8080/hello-world-bean

```json
{"message":"Hello World"}
```

- http://localhost:8080/hello-world/path-variable/kh77

```json
{"message":"Hello World, practice"}
```

## Docker File

### Basic
```
FROM openjdk:8-jdk-alpine
EXPOSE 8080
ADD target/hello-world-docker.jar hello-world-docker.jar
ENTRYPOINT ["sh", "-c", "java -jar /hello-world-docker.jar"]
```

### Level 2

```
FROM openjdk:8-jdk-alpine
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","RestfulWebServicesApplication"]
```

## Plugins

### Dockerfile Maven

- From Spotify
- https://github.com/spotify/dockerfile-maven

```
<plugin>
	<groupId>com.spotify</groupId>
	<artifactId>dockerfile-maven-plugin</artifactId>
	<version>1.4.10</version>
	<executions>
		<execution>
			<id>default</id>
			<goals>
				<goal>build</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<repository>kh77/${project.name}</repository>
		<tag>${project.version}</tag>
		<skipDockerInfo>true</skipDockerInfo>
	</configuration>
</plugin>
```


docker build -t kh77/hello-world-docker:dockerfile1 .


for jib kazim
Containerizing application to Docker daemon as 01-hello-world-docker:0.0.1-SNAPSHOT...

### JIB

- https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin#quickstart

- https://github.com/GoogleContainerTools/jib/blob/master/docs/faq.md

#### "useCurrentTimestamp - true" discussion
- https://github.com/GooleContainerTools/jib/blob/master/docs/faq.md#why-is-my-image-created-48-years-ago 
- https://github.com/GoogleContainerTools/jib/issues/413 

```
<plugin>
	<groupId>com.google.cloud.tools</groupId>
	<artifactId>jib-maven-plugin</artifactId>
	<version>1.6.1</version>
	<configuration>
		<container>
			<creationTime>USE_CURRENT_TIMESTAMP</creationTime>
		</container>
	</configuration>
	<executions>
		<execution>
			<phase>package</phase>
			<goals>
				<goal>dockerBuild</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```
```
<configuration>
	<from>
		<image>openjdk:alpine</image>
	</from>
	<to>
		<image>kh77/${project.name}</image>
		<tags>
			<tag>${project.version}</tag>
			<tag>latest</tag>
		</tags>
	</to>
	<container>
		<jvmFlags>
			<jvmFlag>-Xms512m</jvmFlag>
		</jvmFlags>
		<mainClass>RestfulWebServicesApplication</mainClass>
		<ports>
			<port>8100</port>
		</ports>
	</container>
</configuration>
```

### fabric8io/docker-maven-plugin

- https://dmp.fabric8.io/
- Remove Spotify Maven and JIB Plugins. Add the plugin shown below and configure property for jar file.

Supports 
 - Dockerfile
 - Defining Dockerfile contents in POM XML. 

#### Using Dockerfile

```
<!-- To build the image - "mvn clean package" -->
<!-- Successfully tagged webservices/01-hello-world-docker -->
<!-- docker run -p 8080:8080 webservices/01-hello-world-docker -->
<plugin>
	<groupId>io.fabric8</groupId>
	<artifactId>docker-maven-plugin</artifactId>
	<version>0.26.0</version>
	<executions>
		<execution>
			<id>docker-build</id>
			<phase>package</phase>
			<goals>
				<goal>build</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

```
<properties>
...
 <jar>${project.build.directory}/${project.build.finalName}.jar</jar>
</properties>
```

#### Using XML Configuration

```
<!-- To build the image - "mvn clean package" -->
<!-- TAG - 01-hello-world-docker:latest -->
<!-- docker run -p 8080:8080 01-hello-world-docker:latest -->
<plugin>
   <groupId>io.fabric8</groupId>
   <artifactId>docker-maven-plugin</artifactId>
   <version>0.26.0</version>
   <extensions>true</extensions>
   <configuration>
      <verbose>true</verbose>
      <images>
         <image>
            <name>${project.artifactId}</name>
            <build>
               <from>java:8-jdk-alpine</from>
               <entryPoint>
                  <exec>
                     <args>java</args>
                     <args>-jar</args>
                     <args>/maven/${project.build.finalName}.jar</args>
                  </exec>
               </entryPoint>
               <assembly>
                  <descriptorRef>artifact</descriptorRef>
               </assembly>
            </build>
         </image>
      </images>
   </configuration>
   <executions>
	<execution>
		<id>docker-build</id>
		<phase>package</phase>
		<goals>
			<goal>build</goal>
		</goals>
	</execution>
   </executions>
</plugin>
 ```
 
### Maven Dependency Plugin

```
<plugin>	
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-dependency-plugin</artifactId>
	<executions>
		<execution>
			<id>unpack</id>
			<phase>package</phase>
			<goals>
				<goal>unpack</goal>
			</goals>
			<configuration>
				<artifactItems>
					<artifactItem>
						<groupId>${project.groupId}</groupId>
						<artifactId>${project.artifactId}</artifactId>
						<version>${project.version}</version>
					</artifactItem>
				</artifactItems>
			</configuration>
		</execution>
	</executions>
</plugin>
```
 

### Improve Caching of Images using Layers
 
#### CURRENT SITUATION					

			--------------- 
			    FAT JAR
			--------------- 
			      JDK
			--------------- 

####  DESIRED SITUATION
			--------------- 
			    CLASSES   
			---------------
			 DEPENDENCIES 
			---------------
			     JDK      
			---------------
 