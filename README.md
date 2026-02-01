# Spring Boot REST API with System-Scope Dependencies

A complete Spring Boot sample application demonstrating the use of system-scope dependencies (commons-io and commons-lang3) with Maven, Docker, and OpenShift deployment.

Problem: Enterprises often face situations where:

Certain libraries are not available in public Maven repositories
Internal/proprietary JARs need to be bundled
isolated environments restrict internet access
Legal/compliance requirements mandate specific library versions

Solution: Shows how to integrate external JARs without Maven Central dependency

## 📋 Table of Contents
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [API Endpoints](#api-endpoints)
- [Build and Verification](#build-and-verification)
- [Docker Deployment](#docker-deployment)
- [OpenShift Deployment](#openshift-deployment)
- [Troubleshooting](#troubleshooting)

## ✨ Features

- **Spring Boot 3.2.2** with Java 17
- **System-scope dependencies** using local JAR files
- **Spring Boot Maven Plugin** with `includeSystemScope=true`
- **Complete REST API** demonstrating commons-io and commons-lang3 usage
- **Docker/Podman containerization** with OpenShift compatibility
- **End-to-end deployment** to OpenShift Container Platform

## 🔧 Prerequisites

### Software Requirements
```bash
# Verify all prerequisites
java -version     # Java 17 or higher
mvn -version      # Maven 3.8+
podman --version  # or docker
oc version        # OpenShift CLI
```

### Installation (RHEL/CentOS)
```bash
# Install required packages
yum install maven podman java-21-openjdk -y

# Install OpenShift CLI
wget https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux-s390x/oc.tar.gz -P /tmp && \
    tar -xvf /tmp/oc.tar.gz -C /tmp && \
    mv /tmp/oc /usr/local/bin/ && \
    rm -rf /tmp/oc.tar.gz && \
    chmod +x /usr/local/bin/oc
```

## 🚀 Quick Start

### 1. Setup Project Structure
```bash
# Create project directory
mkdir -p /root/myapp
cd /root/myapp

# Create libs directory
mkdir -p libs
```

### 2. Download Dependencies
```bash
# Download commons-io
curl -L -o libs/commons-io-2.16.1.jar \
https://repo1.maven.org/maven2/commons-io/commons-io/2.16.1/commons-io-2.16.1.jar

# Download commons-lang3
curl -L -o libs/commons-lang3-3.14.0.jar \
https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.14.0/commons-lang3-3.14.0.jar

# Verify downloads
ls -lh libs/
```

### 3. Create Maven Configuration
Create `pom.xml` with system-scope dependencies:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>myapp</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.2</version>
    <relativePath/>
  </parent>

  <properties>
    <java.version>17</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.16.1</version>
      <scope>system</scope>
      <systemPath>${project.basedir}/libs/commons-io-2.16.1.jar</systemPath>
    </dependency>

    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.14.0</version>
      <scope>system</scope>
      <systemPath>${project.basedir}/libs/commons-lang3-3.14.0.jar</systemPath>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <includeSystemScope>true</includeSystemScope>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

### 4. Create Application Code
```bash
# Create source directory structure
mkdir -p src/main/java/com/example/demo
mkdir -p src/main/java/com/example/demo/api
mkdir -p src/main/resources
```

**DemoApplication.java:**
```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}
```

**UtilController.java:**
```java
package com.example.demo.api;

import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.RandomStringUtils;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.ByteArrayInputStream;
import java.nio.charset.StandardCharsets;
import java.util.LinkedHashMap;
import java.util.Map;

@RestController
public class UtilController {

  @GetMapping(value = "/api/util", produces = MediaType.APPLICATION_JSON_VALUE)
  public Map<String, Object> util() throws Exception {
    // commons-lang3 usage
    String raw = "   hello ocp   ";
    String trimmed = StringUtils.trim(raw);
    String token = RandomStringUtils.randomAlphanumeric(10);

    // commons-io usage
    String payload = "commons-io stream read OK";
    ByteArrayInputStream in = new ByteArrayInputStream(payload.getBytes(StandardCharsets.UTF_8));
    String readBack = IOUtils.toString(in, StandardCharsets.UTF_8);

    Map<String, Object> resp = new LinkedHashMap<>();
    resp.put("trimmed", trimmed);
    resp.put("token", token);
    resp.put("readBack", readBack);
    resp.put("status", "OK");
    return resp;
  }
}
```

**application.properties:**
```properties
server.port=8080
```

## 📁 Project Structure
```
myapp/
├── libs/
│   ├── commons-io-2.16.1.jar
│   └── commons-lang3-3.14.0.jar
├── pom.xml
├── Dockerfile
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── example/
        │           └── demo/
        │               ├── DemoApplication.java
        │               └── api/
        │                   └── UtilController.java
        └── resources/
            └── application.properties
```

## 🔌 API Endpoints

### GET /api/util
**Description**: Demonstrates usage of both commons-io and commons-lang3 libraries

**Response Example**:
```json
{
  "trimmed": "hello ocp",
  "token": "A1b2C3d4E5",
  "readBack": "commons-io stream read OK",
  "status": "OK"
}
```

## 🔨 Build and Verification

### 1. Build the Application
```bash
cd /root/myapp
mvn clean package -DskipTests
```

### 2. Verify System-Scope Dependencies
```bash
# Verify it's a Spring Boot fat jar
jar tf target/*.jar | grep -i "BOOT-INF" | head

# Verify both libraries are included
jar tf target/*.jar | grep -i "commons-io\|commons-lang3"

# Expected output:
# BOOT-INF/lib/commons-io-2.16.1.jar
# BOOT-INF/lib/commons-lang3-3.14.0.jar
```

### 3. Run Locally
```bash
java -jar target/myapp-1.0.0.jar
```

### 4. Test the API
```bash
curl http://localhost:8080/api/util
```

## 🐳 Docker Deployment

### 1. Create Dockerfile
```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY target/*.jar /app/app.jar
RUN chmod -R g=u /app
ENV JAVA_OPTS="-Djava.io.tmpdir=/tmp"
EXPOSE 8080
ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar /app/app.jar"]
```

### 2. Build and Run Container
```bash
# Build image
podman build -t myapp:1.0 .

# Run container
podman run --rm -it -p 8080:8080 localhost/myapp:1.0

# Test in another terminal
curl http://localhost:8080/api/util
```

## ☸️ OpenShift Deployment

### 1. Login and Create Project
```bash
oc login <cluster-api-url>
oc new-project my-springboot
```

### 2. Tag and Push to Internal Registry
```bash
# Tag image for OpenShift registry
podman tag myapp:1.0 image-registry.openshift-image-registry.svc:5000/my-springboot/myapp:1.0

# Login to registry
podman login -u $(oc whoami) -p $(oc whoami -t) image-registry.openshift-image-registry.svc:5000

# Push image
podman push image-registry.openshift-image-registry.svc:5000/my-springboot/myapp:1.0
```

### 3. Deploy Application
```bash
# Create deployment
oc new-app --name=myapp image-registry.openshift-image-registry.svc:5000/my-springboot/myapp:1.0

# Expose service
oc expose svc/myapp

# Get route URL
oc get route myapp
```

### 4. Test Deployment
```bash
# Get route hostname
ROUTE_HOST=$(oc get route myapp -o jsonpath='{.spec.host}')

# Test API
curl http://${ROUTE_HOST}/api/util
```

## 🐛 Troubleshooting

### Common Issues

1. **System-scope dependencies not included in JAR**
   - Verify `includeSystemScope` is set to `true` in Spring Boot Maven Plugin
   - Check JAR contents with `jar tf target/*.jar | grep commons`

2. **Docker build fails**
   - Ensure JAR file exists in `target/` directory
   - Verify Dockerfile path is correct relative to JAR location

3. **OpenShift push fails**
   - Check registry authentication: `oc whoami -t`
   - Verify project exists: `oc get projects`
   - Check image tag format matches internal registry

4. **Application doesn't start**
   - Check logs: `oc logs deployment/myapp`
   - Verify Java version compatibility
   - Check resource limits in OpenShift

### Verification Commands
```bash
# Check if dependencies are properly packaged
jar tf target/myapp-1.0.0.jar | grep -E "(commons-io|commons-lang3)" | head -5

# Verify Spring Boot structure
unzip -l target/myapp-1.0.0.jar | grep BOOT-INF/lib/spring

# Test container locally
podman run --rm -it -p 8080:8080 localhost/myapp:1.0 curl localhost:8080/api/util
```

## 📝 Notes

- **System Scope Dependencies**: These are not resolved from Maven repositories but from local filesystem. Ensure JARs are available in the `libs` directory.
- **OpenShift Compatibility**: The Dockerfile uses `chmod -R g=u /app` for arbitrary user ID support required by OpenShift security context constraints.
- **Build Process**: The application must be built before creating the Docker image since the Dockerfile copies the pre-built JAR.
- **Security**: Consider using dependency management instead of system scope for production applications where possible.

## 📄 License

This project is for demonstration purposes. The included libraries (commons-io, commons-lang3) are Apache 2.0 licensed.

---
