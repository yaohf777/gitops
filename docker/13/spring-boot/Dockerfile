# syntax=docker/dockerfile:1
# Custom Dockerfile syntax : https://docs.docker.com/build/buildkit/frontend/

# 拉取17-jdk-jammy，并将构建阶段命名为builder。
# 17-jdk-jammy是基于Ubuntu Jammy的JDK镜像，适用于构建Java应用程序。
# 通过使用多阶段构建，我们可以在第一个阶段中使用较大的JDK镜像来编译应用程序，而在第二个阶段中使用较小的JRE镜像来运行应用程序，从而减小最终镜像的大小。
FROM eclipse-temurin:17-jdk-jammy AS builder
# 在容器内设置工作目录为/opt/app
WORKDIR /opt/app
# 将本地目录下的.mvn文件夹复制到容器内的.mvn文件夹
# The special Maven files help download dependencies
# The .mvn directory contains the Maven Wrapper, which allows you to run Maven commands without having Maven installed on the host machine.
COPY .mvn/ .mvn
# 将本地目录下的mvnw和pom.xml复制到容器内的当前工作目录
# Copy only essential Maven files required to download dependencies
COPY mvnw pom.xml ./
# 在容器内运行 ./mvnw dependency:go-offline 命令，下载应用所需的依赖包并缓存到本地
# This command tells Maven to download all the dependencies specified in the pom.xml file and cache them locally. 
# The dependencies will be cached and won't need to be downloaded again unless the pom.xml file changes.
RUN ./mvnw dependency:go-offline
# 将本地目录下的src文件夹复制到容器内的src文件夹
COPY ./src ./src
# 在容器内运行./mvnw clean install命令，编译应用并生成可执行的jar文件
RUN ./mvnw clean install


FROM eclipse-temurin:17-jre-jammy
WORKDIR /opt/app
# 声明镜像运行时监听的端口号为 8080
EXPOSE 8080
# 从之前命名为builder的构建阶段中，将生成的jar文件复制到当前阶段的/opt/app目录下
COPY --from=builder /opt/app/target/*.jar /opt/app/*.jar
# 设置容器启动时执行的命令，即运行/opt/app目录下的所有jar文件
CMD ["java", "-jar", "/opt/app/*.jar" ]