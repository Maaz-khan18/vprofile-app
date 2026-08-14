# VProfile App
##
VProfile is a Java Spring MVC web application built with Maven and packaged as a WAR for deployment on Tomcat. The project includes Spring MVC, Spring Security, JPA, RabbitMQ, Elasticsearch, and MySQL integrations, and it is configured for CI/CD with GitHub Actions, Docker, ECR, and Helm-based deployment.

## Tech Stack

- Java 21 (CI pipeline target)
- Maven
- Spring Framework 6
- Spring MVC
- Spring Security
- Spring Data JPA / Hibernate
- MySQL
- RabbitMQ
- Elasticsearch
- Tomcat 10
- Docker
- Amazon ECR
- Helm
- SonarQube (self-hosted)

## Project Structure

```text
.
├── .github/
│   └── workflows/
│       └── ci.yml
├── Docker-files/
│   ├── app/
│   │   ├── Dockerfile
│   │   └── multistage/
│   │       └── Dockerfile
│   ├── db/
│   │   ├── db_backup.sql
│   │   └── Dockerfile
│   └── web/
│       ├── Dockerfile
│       └── nginvproapp.conf
├── src/
│   ├── main/
│   │   ├── java/
│   │   ├── resources/
│   │   └── webapp/
│   └── test/
│       └── java/
├── pom.xml
├── sonar-project.properties
├── Dockerfile
└── README.md
```

## Prerequisites

Before running this project locally, make sure the following are installed:

- JDK 21 or compatible Java version
- Maven 3.9+
- Docker
- Git
- Optional: Tomcat for manual WAR deployment

## Build and Run Locally

### 1. Clone the repository

```bash
git clone <repository-url>
cd Vprofile-app
```

### 2. Build the project

```bash
mvn clean package
```

This produces a WAR artifact in the target directory.

### 3. Run tests

```bash
mvn test
```

### 4. Run the app locally

You can deploy the WAR to Tomcat or run it using a servlet container configured in your local environment.

```bash
mvn jetty:run
```

## Docker Build

This project includes a multi-stage Docker build at:

- Docker-files/app/multistage/Dockerfile

To build locally:

```bash
docker build -f Docker-files/app/multistage/Dockerfile -t vprofile-app:local .
```

To run the container:

```bash
docker run -p 8080:8080 vprofile-app:local
```

## CI/CD Pipeline

The repository includes a GitHub Actions workflow in [.github/workflows/ci.yml](.github/workflows/ci.yml).

### Pipeline flow

1. Feature branch push
   - No pipeline runs

2. Pull request to main
   - Maven build
   - Unit tests
   - Checkstyle
   - SonarQube scan
   - SonarQube quality gate check
   - Merge is blocked if the quality gate fails

3. Push to main
   - Build Docker image
   - Push image to Amazon ECR with:
     - commit SHA tag
     - latest tag
   - Update the Helm values file in the GitOps repository

## SonarQube Configuration

The project uses the existing configuration file:

- sonar-project.properties

This file contains the project key, source folder, test output paths, and report paths for coverage and Checkstyle.

## Required GitHub Configuration

### Repository Variables

- AWS_REGION
- ECR_REPOSITORY
- HELM_REPO_NAME
- SONAR_HOST_URL

### Repository Secrets

- SONAR_TOKEN
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- HELM_REPO_USER
- GITOPS_PAT
- SLACK_WEBHOOK (optional for notifications)

## Helm Update Behavior

The pipeline updates the Helm values file in a separate GitOps repository. The expected structure is:

```yaml
app:
  image:
  tag:
  replicas: 1
  containerPort: 8080
  servicePort: 8080
```

The workflow uses yq to update:

- app.image
- app.tag

## Notes

- The application is packaged as a WAR and is intended for deployment into Tomcat or a compatible servlet container.
- The Docker build uses a multi-stage process to compile the app and copy the generated WAR into a Tomcat runtime image.
- The CI pipeline is designed to run SonarQube and validation checks on pull requests while restricting Docker/ECR and Helm updates to pushes to main.

## License

This project is provided as-is for educational and deployment demonstration purposes unless otherwise specified by the repository owner.
