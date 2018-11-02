# google-cloud-storage-signed-urls

Lab to demonstrate how to create signed URLs for granting anyone with the URL access to objects in Cloud Storage.

![Final Environment](https://user-images.githubusercontent.com/3911650/47931260-e64a7c00-de93-11e8-8695-bb178d9de59d.png)

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

1. Create a key for the pre-created storage account:

    ```sh
    sa_email=$(gcloud iam service-accounts list --format='value(email)' | grep storage-signer) # service account email (ID)
    gcloud iam service-accounts keys create --iam-account $sa_email key.json
    ```

1. Upload a file to the pre-created bucket:

    ```sh
    curl -L https://github.com/cloudacademy/gcp-lab-artifacts/raw/master/gcs/ca.png -o ca.png
    bucket=$(gsutil ls -b | sed 's/\/$//') # bucket with trailing slash removed
    gsutil cp ca.png $bucket
    ```

1. Install the Python OpenSSL library (required for signing URLs):

    ```sh
    pip install pyopenssl --user
    ```

1. Grant the service account read access to the object:

    ```sh
    gsutil acl ch -u $sa_email:READ $bucket/ca.png
    ```

1. Create a signed URL to access the object for five minutes:

    ```sh
    gsutil signurl -d 5m key.json $bucket/ca.png
    ```

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
