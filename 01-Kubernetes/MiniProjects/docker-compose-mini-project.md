# Building a Microservices Application with Python and Java

This guide walks you through creating a microservices application with a Python service (using Flask) and a Java service (using Spring Boot). Both services will be deployed using Docker Compose.

---

## Application Overview

The application consists of the following services:

1. **Python Service**: A lightweight Flask application providing a greeting API.
2. **Java Service**: A Spring Boot application with another greeting API.

Docker will containerize both services, and Docker Compose will manage and deploy them.

---

## Project Structure

Below is the directory layout for the project:

```
microservices-app/
├── python-service/
│   ├── app.py
│   ├── Dockerfile
│   ├── requirements.txt
├── java-service/
│   ├── src/main/java/com/example/demo/DemoApplication.java
│   ├── src/main/resources/application.properties
│   ├── Dockerfile
│   ├── pom.xml
├── docker-compose.yml
```

## Python Service Details

### Python Microservice

#### `python-service/app.py`

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/greet', methods=['GET'])
def greet():
    return jsonify(message="Hello from Python service!")

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### `python-service/requirements.txt`

```
Flask
```

#### `python-service/Dockerfile`

```dockerfile
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

### Java Microservice

#### `java-service/src/main/java/com/example/demo/DemoApplication.java`

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @RestController
    class GreetingController {

        @GetMapping("/greet")
        public String greet() {
            return "Hello from Java service!";
        }
    }
}
```

#### `java-service/src/main/resources/application.properties`

```properties
server.port=8080
```

#### `java-service/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.6</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

#### `java-service/Dockerfile`

```dockerfile
FROM openjdk:11-jre-slim

WORKDIR /app

COPY target/demo-0.0.1-SNAPSHOT.jar demo.jar

CMD ["java", "-jar", "demo.jar"]
```

### Docker Compose File

#### `docker-compose.yml`

```yaml
version: "3.8"

services:
  python-service:
    build: ./python-service
    ports:
      - "5000:5000"
    networks:
      - app-network

  java-service:
    build: ./java-service
    ports:
      - "8080:8080"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

Deployment Instructions

1. **Build the Java Application**: First, navigate to the `java-service` directory and build the Spring Boot application:

   ```bash
   cd java-service
   ./mvnw package
   ```

   This will create a JAR file in the `target` directory.

2. **Start Docker Compose**: Navigate back to the root directory of the project (`microservices-app`) and run Docker Compose:

   ```bash
   cd ..
   docker-compose up --build
   ```

3. **Testing the Services**:
   - Python service: Open a browser and go to `http://localhost:5000/greet` to see the greeting message from the Python service.
   - Java service: Open a browser and go to `http://localhost:8080/greet` to see the greeting message from the Java service.

Final Thoughts

This tutorial demonstrates a simple yet effective way to create and manage microservices using Flask and Spring Boot. By leveraging Docker Compose, you can easily deploy and orchestrate multiple containerized services, providing a streamlined development and deployment workflow.

Let me know if you'd like additional edits or adjustments!
