# DevOps - Gjør alt i et fett 
### Av Martin Olaussen


* [ ] Lag et spring initalizer prosjekt med kotlin og spring web 
* [ ] Lag en Dockerfile som skal plasseres i root folderen 


````
FROM maven:3.6-jdk-11 as builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn package


FROM adoptopenjdk/openjdk11:alpine-slim
COPY --from=builder /app/target/*.jar /app/application.jar
ENTRYPOINT ["java", "-jar", "/app/application.jar"]
````


* [ ] Lager et veldig basic REST api
* [ ] Lager en .github/workflow mappe med en .yml fil f.eks ci.yml (continious integration)


````
#Continious Integration

name: Java CI with Maven
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'
          cache: maven
      - name: Build with Maven
        run: mvn -B package --file pom.xml
````

## NB! Husk at master/main må endres basert på hva det heter i github

* [ ] Ta med status badge på CI workflowen når den passerer
* [ ] 



