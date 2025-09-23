FROM amazoncorretto:17
LABEL authors="leonardogloria"
WORKDIR /app
COPY *.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]