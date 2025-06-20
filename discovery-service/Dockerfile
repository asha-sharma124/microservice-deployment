FROM openjdk:8-jre-slim
VOLUME /tmp
ENV environment=docker
ADD target/*.jar app.jar
COPY wait-for-it.sh wait-for-it.sh
RUN chmod +x wait-for-it.sh

#ENV JAVA_OPTS="-XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode -XX:+PrintGCDetails -XX:+PrintGCTimeStamps"
ENV JAVA_OPTS="-XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode"
ENTRYPOINT java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -Dspring.profiles.active=$environment -jar /app.jar
EXPOSE 8761
