FROM openjdk:8-jre-slim

VOLUME /tmp
ENV environment=docker
ADD target/*.jar app.jar
ENV JAVA_OPTS="-XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode"
# ENV JAVA_OPTS="-XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode -XX:+PrintGCDetails -XX:+PrintGCTimeStamps"
ENTRYPOINT java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -Dspring.profiles.active=$environment -jar /app.jar
EXPOSE 9090
