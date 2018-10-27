# google-cloud-storage-signed-urls

Lab to demonstrate how to create presigned URLs for secure access to objects in Cloud Storage

## Getting Started

1. Ensure the Following APIs are enabled (enable with gcloud services enable [service]):

    - iam.googleapis.com
    - storage-component.googleapis.com

1. [Ensure the default Google APIs service account (used by deployment manager) has permission to create roles](https://cloud.google.com/deployment-manager/docs/configuration/set-access-control-resources):

    ```sh
    gcloud projects add-iam-policy-binding [PROJECT_ID] \
    --member serviceAccount:[PROJECT_NUMBER]@cloudservices.gserviceaccount.com  \
    --role roles/iam.roleAdmin
    ```

    You can use `gcloud list projects` to get the project ID and number.

1. Deploy the deployment manager config in the `infrastructure` directory:

    ```sh
    gcloud deployment-manager deployments create lab --config infrastructure/deployment.yaml
    ```

1. Bind the Lab role to the student user or group

    - In macOS/Linux:

        ```sh
        member="[GROUP_OR_USER]"
        project_id=$(gcloud config list --format 'value(core.project)')
        role=$(gcloud iam roles list --project $project_id \
                                     --filter "name:projects/$project_id/roles/studentrole*" \
                                     --format "value(name)")
        gcloud projects add-iam-policy-binding $project_id \
        --member $member  \
        --role $role
        ```

    - In Windows (PowerShell):

        ```ps1
        $member = "[GROUP_OR_USER]"
        $project_id = gcloud config list --format 'value(core.project)'
        $role = gcloud iam roles list --project $project_id `
                                      --filter "name:projects/$project_id/roles/studentrole*" `
                                      --format "value(name)"
        gcloud projects add-iam-policy-binding $project_id `
        --member $member  `
        --role $role
        ```

    An example of `[GROUP_OR_USER]` is `user:student@gmail.com`.

## Following Along

1. Start a Google Cloud Shell session.

1. 

## Tearing Down

When finished, remove the GCP resources with:

- In macOS/Linux:

    ```sh
    bucket=$(gsutil ls -b gs://ca-lab-bucket-*)
    gsutil rm -r $bucket
    gcloud projects remove-iam-policy-binding $project_id \
        --member $member  \
        --role $role
    gcloud deployment-manager deployments delete -q lab
    ```

- In Windows (PowerShell):

    ```ps1
    $bucket = gsutil ls -b gs://ca-lab-bucket-*
    gsutil rm -r $bucket
    gcloud projects remove-iam-policy-binding $project_id `
        --member $member  `
        --role $role
    gcloud deployment-manager deployments delete -q lab
    ```
