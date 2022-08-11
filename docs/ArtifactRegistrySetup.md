# Create a Repository in Artifact Registry

A repository can be created easily from the gcloud CLI:
```shell
# generic
# gcloud artifacts repositories create modern-java-container --repository-format=docker --location=<region> --description="Docker repository for multi-layer lab images"

# ex
gcloud artifacts repositories create modern-java-container \ 
    --repository-format=docker \ 
    --location=us-central1 \ 
    --description="Docker repository for multi-layer images"
```


## Validate that the repo has been created
```shell
gcloud artifacts repositories list | grep demo-app

REPOSITORY               FORMAT  DESCRIPTION                               LOCATION     LABELS  ENCRYPTION          CREATE_TIME          UPDATE_TIME
...
demo-app                 DOCKER  Demo Application Images                   us-central1          Google-managed key  
...
```

## Authentication
Before you can push or pull images, configure Docker to use the Google Cloud CLI to authenticate requests to Artifact Registry.

To set up authentication to Docker repositories in the region us-central1, run the following command:
```shell
gcloud auth configure-docker us-central1-docker.pkg.dev
```

