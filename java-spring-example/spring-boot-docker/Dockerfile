# syntax=docker/dockerfile:1
FROM cantara/alpine-openjdk-jdk8
COPY /target/spring-boot-docker.jar .
ENTRYPOINT ["java","-jar","spring-boot-docker.jar"]