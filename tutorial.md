<walkthrough-metadata>
  <meta name="title" content="GCP CI/CD security demo and tutorial" />
  <meta name="description" content="Demostrates a security focused CI/CD pipeline for GKE with Google Cloud tools Cloud Build, Binary Authorization, Artifact Registry, Container Analysis, and Google Cloud Deploy." />
  <meta name="component_id" content="103" />
</walkthrough-metadata>

<walkthrough-disable-features toc></walkthrough-disable-features>

# GCP CI/CD security demo and tutorial
This repo demostrates a security focused CI/CD pipeline for GKE with Google Cloud tools Cloud Build, Binary Authorization, Artifact Registry, Container Analysis, and Google Cloud Deploy.

## Select a project

<walkthrough-project-setup billing="true"></walkthrough-project-setup>

Once you've selected a project, click "Start".

## Set the PROJECT_ID environment variable

Set the PROJECT_ID environment variable. This variable will be used in forthcoming steps.
```bash
export PROJECT_ID=<walkthrough-project-id/>
```

### Enable needed APIs and Create Google Cloud Deploy pipeline
The 
<walkthrough-editor-open-file filePath="bootstrap/init.sh">bootstrap/init.sh</walkthrough-editor-open-file>
script enables your APIs, customizes your 
<walkthrough-editor-open-file filePath="clouddeploy.yaml">
clouddeploy.yaml
</walkthrough-editor-open-file> 
and creates a Cloud Deploy pipeline for you. You'll still need to do some steps manually after these scripts run, though.

Run the initialization script:
```bash
. ./bootstrap/init.sh
```

### View your Google Cloud Deploy Pipeline

Verify that the Google Cloud Deploy pipeline was created in the 
[Google Cloud Deploy UI](https://console.cloud.google.com/deploy/delivery-pipelines)

## Turn on automated container vulnerability analysis
Google Cloud Container Analysis can be set to automatically scan for vulnerabilities on push (see [pricing](https://cloud.google.com/container-analysis/pricing)). 

Enable Container Analysis API for automated scanning:

<walkthrough-enable-apis apis="containerscanning.googleapis.com"></walkthrough-enable-apis>

## Configure your Github.com repo

If you have not forked this repo yet, please do so now:

[Fork this repo on Github](https://github.com/vszal/gcp-secure-cicd/fork)

To keep file changes you make in Cloud Shell in sync with your repo, you can check these file changes into your new Github repo by following these [docs](https://docs.github.com/en/get-started/importing-your-projects-to-github/importing-source-code-to-github/adding-locally-hosted-code-to-github). Note that the Github CLI is available in Cloud Shell.


## Setup a Cloud Build trigger for your repo
Now that your Github repo is setup, configure Cloud Build to run each time a change is pushed to the main branch. To do this, add a Trigger in Cloud Build:
  * Navigate to [Cloud Build triggers page](https://console.cloud.google.com/cloud-build/triggers)
  * Follow the [docs](https://cloud.google.com/build/docs/automating-builds/build-repos-from-github) and create a Github App connected repo and trigger.
  * Configuration --> Type: select "Cloud Build configuration file"
  * Configuration --> Location: ensure Repository and "/cloudbuild.yaml" is selected.

## Create GKE clusters
You'll need GKE clusters to deploy to. The Google Cloud Deploy pipeline in this example refers to two clusters:
* test-sec
* prod-sec

Feel free to add more clusters, just update the clouddeploy.yaml file with the additional steps and targets. If you have/want different cluster names update cluster definitions in:
* <walkthrough-editor-open-file filePath="bootstrap/gke-init.sh">bootstrap/gke-init.sh</walkthrough-editor-open-file>
* <walkthrough-editor-open-file filePath="clouddeploy.yaml">clouddeploy.yaml</walkthrough-editor-open-file>
* <walkthrough-editor-open-file filePath="bootstrap/gke-delete.sh">bootstrap/gke-delete.sh</walkthrough-editor-open-file>

Make sure you have Binary Authorization enabled for any existing clusters you may want to use.

### Create GKE clusters

```bash
. ./bootstrap/gke-init.sh
```

Note that these clusters are created asynchronously, so check on the [GKE UI]("https://console.cloud.google.com/kubernetes/list/overview") periodically to ensure that the clusters are up before submitting your first release to Google Cloud Deploy.

## IAM and service account setup
You must give Cloud Build explicit permission to trigger a Google Cloud Deploy release.
1. Read the [docs](https://cloud.google.com/deploy/docs/integrating)
2. Navigate to [IAM](https://console.cloud.google.com/iam-admin/iam) and locate your Cloud Build service account
3. Add these two roles
  * Cloud Deploy Releaser
  * Service Account User

## Kritis Signer and attestor setup

As part of the CI/CD pipeline, the image generated will be scanned for vulnerabilities. We’ll be using Kritis Signer as a “Cloud Builder” to vet created images against a vulnerability threshold policy and fail our builds if the vulnerability threshold is exceeded.


Follow the [doc steps](https://cloud.google.com/binary-authorization/docs/creating-attestations-kritis#set_up_the_kritis_signer_custom_builder) to create a Kritis signer custom builder image:
* Navigate outside the secure-cicd-maven project area and clone the Kritis repo
* Navigate into the Kritis folder
* Create your Kritis signer cloud builder (this will push the Kritis signer image to gcr.io/YOUR_PROJECT/kritis:latest).

## Set up the Kritis Signer Binary Authorization attestor

Beyond just failing the build, we can also have Kritis Signer create a Binary Authorization attestation that the image passed vulnerability thresholds. This will be used for admission control during deployment later.	


1. Create signing keys used, steps 1 and 2 in [this doc](https://cloud.google.com/architecture/binary-auth-with-cloud-build-and-gke#creating_signing_keys):
* Create a key ring, make note of the resource location
* Create a signing signing key, make note of the resource location

2. Create the vulnerability signing note and attestor [per these docs, steps 1-6](https://cloud.google.com/architecture/binary-auth-with-cloud-build-and-gke#create_the_vulnerability_scanner_attestation)
Please name your attestor “vulnz-attestor” during step 3.
* Make note of the attestor's "note" resource location


3. Verify that your new attestor appears in the [console UI](https://console.cloud.google.com/security/binary-authorization/attestors):
If you’ve already created your Kritis signer Cloud Builder in the previous section, then you should see two attestors:
* vulnz-attestor (Your newly created attestor)
* built-by-cloud-build attestor (system generated)

## Setup Binary Authorization policy

Now that you have some attestors, you can create a Binary Authorization policy that restricts deployments to only those images that have the two attestations.

1. From the root of the repo, run:

```bash
gcloud container binauthz policy import policy/binauthz/attestor-policy.yaml
```

2. Verify that your policy is in place by visiting the [Binary Authorization UI](https://console.cloud.google.com/security/binary-authorization/policy).

For more docs, see: https://cloud.google.com/binary-authorization/docs/setting-up

## Add substitutions to the Cloud Build Trigger
During a previous step you created a Github trigger for the build. While the following substitutions can be added directly to the cloudbuild file, it’s a bit safer to add these to the trigger itself.

1. Open the Cloud Build trigger UI
* Find and edit your trigger in the UI. If you don’t see it, make sure you select the region you set up the trigger in.

2. Click the 3 dots menu → Edit

3. Scroll to the Advanced section and add these three variables:
* Variable 1: _KMS_DIGEST_ALG (value will be "SHA512" or "SHA256", depending on your key)
* Variable 2: _KMS_KEY_NAME (value will look like: projects/vsz-demo/locations/global/keyRings/RING-NAME/cryptoKeys/YOUR-KEY/cryptoKeyVersions/1)
* Variable 3: _NOTE_NAME (value will look like: projects/vsz-demo/notes/NOTE-NAME)

You can find the last two values in the vulnz-attestor in the [Binary Authorization attestor list UI](https://console.cloud.google.com/security/binary-authorization/attestors?). 

## Set up a Cloud Build private pool
[Private pools](https://cloud.google.com/build/docs/private-pools/private-pools-overview) provide added security where builds will run with custom parameters including networks (VPC peering).

1. Go to the [Cloud Build worker pool UI](https://console.cloud.google.com/cloud-build/settings/worker-pool)
2. Click Create
3. Use these parameters:
* Name: private-pool
* Region: us-central1
* Machine type: e2-medium
* Available disk size: 100 GB
* Project: (leave blank)
* Network: (leave blank)
* Assign external IPs: checked
4. Click save

## Demo Overview

[![Google Cloud Software Supply Chain Security Demo Flow](https://user-images.githubusercontent.com/76225123/170594159-cae11896-5ac1-473c-8d71-924a8d059155.png)](https://user-images.githubusercontent.com/76225123/170594159-cae11896-5ac1-473c-8d71-924a8d059155.png)

This section is WIP.
For now, see the [gist](https://gist.github.com/vszal/2bca4b844e70449022f153ed4dc87e41)

## Tear down

To remove the running GKE clusters, run:
```bash
. ./bootstrap/gke-delete.sh
```