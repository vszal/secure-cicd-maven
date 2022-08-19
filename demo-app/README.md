# Welcome to the Maven demo app!

This is a Maven quote rotation sample application courtesy of [ddobrin](https://github.com/ddobrin). It can be built locally, in docker, and run in Docker, Kubernetes, or Cloud Run. The parent directory contains skaffold.yaml and Cloud Build files to build automatically.

## Local build Pre-requisites

1. Install [Eclipse-Temurin 17](https://adoptium.net/installation/) in the local build environment
2. Install [Maven 3](https://maven.apache.org/install.html) in the local build environment

## Integration test with Maven

1. Ensure a local docker environment is running
2. From the demo-app directory run:

```bash
mvn verify
```

## Maven package

1. Ensure a local docker environment is running
2. From the demo-app directory run:

```bash
mvn package
```

## Docker build

After packaging locally, you can run docker build to create a container.

Customize command below for your project, repo, and app name:

```bash
docker build . -t us-central1-docker.pkg.dev/YOUR-PROJECT/YOUR-REPO/demo-app
```
## Docker run

Customize command below for your project, repo, and app name:
```bash
docker run us-central1-docker.pkg.dev/YOUR-PROJECT/YOUR-REPO/demo-app
```

## Docker push

Customize command below for your project, repo, and app name:
```bash
docker push us-central1-docker.pkg.dev/YOUR-PROJECT/YOUR-REPO/demo-app
```

## Continuous development with Skaffold

1. Ensure a local docker environment is running
2. Navigate to the root directory of the repo
3. Customize command below for your project and repo, and run:

```bash
skaffold dev --default-repo=us-central1-docker.pkg.dev/YOUR-PROJECT/YOUR-REPO
```
