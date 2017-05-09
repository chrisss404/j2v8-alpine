
This image is intended for packaging Java applications, which use Java bindings for V8 ([J2V8](https://github.com/eclipsesource/J2V8)), with the well known [OpenJDK Alpine images](https://hub.docker.com/r/_/openjdk/).

### Example for Packaging the Contents of a Spring Boot Fat Jar

    FROM chrisss404/j2v8-alpine:latest-jre8
    
    # vendor files
    COPY maven/BOOT-INF/lib /app/BOOT-INF/lib
    COPY maven/org /app/org
    
    # app files
    COPY maven/META-INF /app/META-INF
    COPY maven/BOOT-INF/classes /app/BOOT-INF/classes
    
    ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-Dserver.port=80", \
                "-cp", "/app", "org.springframework.boot.loader.JarLauncher"]
    
    EXPOSE 80
