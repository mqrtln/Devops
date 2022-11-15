# DevOps - Gjør alt i et fett 
#### Av Martin Olaussen



# Start 
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


# Continous integration
* [ ] Lag en veldig basic REST api
* [ ] Lag en .github/workflow mappe med en .yml fil f.eks ci.yml (continious integration)
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


# Docker 

* [ ] Lag en di_terraform.yml under workflows mappen med koden
````
name: Publish Docker image

on:
  push:
    branches:
      - master

jobs:
  build_docker_image:
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

// Sett dette inn hvis du skal ha terraform

terraform:
    name: "Terraform"
    needs: build_docker_image
    runs-on: ubuntu-latest
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_REGION: eu-west-1
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1

      - name: Terraform Format
        id: fmt
        run: terraform fmt -check

      - name: Terraform Init
        id: init
        run: terraform init

      - name: Terraform Validate
        id: validate
        run: terraform validate -no-color

      - name: Terraform Plan
        id: plan
        if: github.event_name == 'pull_request'
        run: terraform plan -no-color
        continue-on-error: true

      - name: Terraform Plan Status
        if: steps.plan.outcome == 'failure'
        run: exit 1

      - name: Terraform Apply
        if: github.ref == 'refs/heads/master' && github.event_name == 'push'
        run: terraform apply -var="prefix=<aws-username>" -var="image=244530008913.dkr.ecr.eu-west-1.amazonaws.com/<ECS-navnet>:latest" -auto-approve

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

# Terraform
 * [ ] legg til terraform koden i dockerimage.yml (den er allerede der i dette eksempelet)
 * [ ] lag en provider.tf fil i rotmappen 
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "3.56.0"
    }
  }
  backend "s3" {
    bucket = "pgr301-2021-terraform-state"
    key    = "<ditt-aws-navn>/<hva-som-helst>.state"
    region = "eu-north-1"
  }
}
```

* [ ] lag en s3_bucket.tf fil i rotmappen

```
resource "aws_s3_bucket" "jenka-botte" {
  bucket = "<ny-bucket-navn>"
}
```

* [ ] lag en variables.tf fil i rotmappen


```
variable "prefix" {
  type = string
}

variable "image" {
  type = string
}
```

* [ ] lag en apprunner.tf fil i rotmappen
```
resource "aws_apprunner_service" "service" {
  service_name = var.prefix

  source_configuration {

    authentication_configuration {
      access_role_arn = "arn:aws:iam::244530008913:role/service-role/AppRunnerECRAccessRole"
    }

    image_repository {
      image_configuration {
        port = "8080"
      }
      image_identifier      = var.image
      image_repository_type = "ECR"
    }
    auto_deployments_enabled = true
  }
}
```
