# DevOps - Gjør alt i et fett 
#### Av Martin Olaussen



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
* [ ] Push til github


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
* [ ] Legg til branch protection rules under settings - branches - add branch protection rule og huk av 
  * [ ] "Require pull request before merging" 
  * [ ] "Require status check to pass before merging"


## Docker 

* [ ] Lag en dockerimage.yml under workflows mappen med koden
````
name: Publish Docker image

on:
  push:
    branches:
      - master

jobs:
  push_to_registry:
    name: Push Docker image to ECR
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v2

      - name: Build and push Docker image
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 244530008913.dkr.ecr.eu-west-1.amazonaws.com
          rev=$(git rev-parse --short HEAD)
          docker build . -t <Ditt image tag>
          docker tag <image tag> 244530008913.dkr.ecr.eu-west-1.amazonaws.com/<ECS-bruker>:$rev
          docker push 244530008913.dkr.ecr.eu-west-1.amazonaws.com/<ECS-bruker>:$rev

````

## NB pass på å endre master/main basert på github branchen din og sett inn dine variabler under alle < >

* [ ] gå inn i IAM i aws tjenesten din
  * [ ] Gp til users og finn brukeren din 
  * [ ] Gå under security credentials
  * [ ] Create access key
  * [ ] Gå i github - settings - secrets - workflows
    * [ ] Legg in disse hver for seg. eks
```
  AWS_ACCESS_KEY
  din access key

  AWS_SECRET_KEY
  din secret key

  Alternativt sett in Region

  AWS_REGION 
  eu-west-1
```

