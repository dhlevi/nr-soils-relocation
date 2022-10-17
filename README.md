![Lifecycle:Experimental](https://img.shields.io/badge/Lifecycle-Experimental-339999)

# Soil Relocation Information System

This is a mapping application providing high-volume receiving site registration to Indigenous Nations, municipalities and other interested parties.  Subscribe for notifications in your area!


### Process

1. BC users in BC submit soil relocation information using a [CHEFS form](link-goes-here)

2. [This script](./chefs_soils.py) is run on a schedule

3. Data from CHEFS forms is uploaded to [ArcGIS Online (AGOL)](https://www.arcgis.com/) and soil movement subscribers are notified

4. This information is displayed in the [frontend](link-goes-here), which is be made public soon


### Technical

The Project uses [OpenShift CronJobs](https://docs.openshift.com/container-platform/4.10/nodes/jobs/nodes-nodes-jobs.html#nodes-nodes-jobs-creating-cron_nodes-nodes-jobs)

The template is [here](https://raw.githubusercontent.com/bcgov/nr-soils-relocation/main/openshift/sris-schedule-job.yml)

The workflow file for build and deploy is [here](.github/workflows/ci-openshift-prod.yaml).

GitHub secrets are passed as environment variables through a configmap.

The CHES API Key is generated from Postman, by providing the username(client_id) and (client_secret).  That populates as environment variable(CHES_API_OAUTH_SECRET) in the pipeline.

Cronjob times are in UTC, like the OpenShift servers.

### OpenShift Deployment

NOTE: This application is the middle layer between CHEFS forms and AGOL, neither of which offer non-PROD environments. Once our application hits production and becomes publicly available this approach will be revisited.

1. Clone this repository and make changes in feature branches (e.g. feat/abc).  Submit changes in Pull Requests against the main branch.

2. Once the code is reviewed and approved, merge it to main branch.

3. Go to the Actions tab, click on (`Build And Add Job to Openshift Prod`)(https://github.com/${{ github.repository }}/actions//workflows/ci-openshift-prod.yaml) and trigger a build

   1. Provide a unique tag number.  Check the [releases/tags](https://github.com/${{ github.repository }}/tags) and increment accordingly

   2. Click on Run workflow button

4. The workflow will:

   1. Build and push a container image to the [GitHub Container Registry (GHCR.io)](https://github.com/${{ github.repository }}/pkgs/container/${{ github.event.repository.name }}), tagged with the release you provided

   2. Create the matching GitHub [release/tag](https://github.com/${{ github.repository }}/tags)

   3. Deploy code and configmap changes to OpenShift

5. A cronjob will be deployed in the PROD environment using the new image

6. Environment variables are supplied to the container through the configmap using GitHub Secrets

7. Environment varialbes must be updated only through GitHub Secrets, our _source of truth_

8. The cronjob is scheduled to run daily at 8 AM UTC (1AM PDT)

9. Cronjobs can be triggered manually, usually for testing, with the One-Time Job steps below

#### One-Time Job

Sometimes waiting for a scheduled cronjob is not practical.  These steps explain how to run a one-time job.

1. Visit the [OpenShift console](https://console.apps.silver.devops.gov.bc.ca/k8s/ns/f0431b-prod/cronjobs/sris-cron-job/yaml)

2. Copy the spec.spec.containers section of the cronjob template, including the container's TAG

3. Go to Jobs menu in the same namespace and click Create Job, located on the right hand side

4. Populate the job, and below, and click Create
   
   1. Name: `sris-1-of-job-$name`, using your own name for $name
   
   2. Containers: using the code copied in step 2, above
    
5. Verify that a new container is running your new job

6. Finally, delete this and any other completed jobs
