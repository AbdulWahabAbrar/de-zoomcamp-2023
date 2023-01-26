# Week 1 Homework Part B

I had already used Terraform to create a BigQuery schema and a Cloud Storage when I was going through "DE Zoomcamp 1.4.1" video and did not realize it is a part of Homework. 

But I re-ran the commands for authenticating gcloud by keyfile and ran the terraform commands which says I already have the resource in Google Cloud. 
Hence, I am copying the output I got after running the command and I am pasting the images as well from Google Cloud.

```bash
(base) abdulwahab@de-zoomcamp:~/data-engineering-zoomcamp$ cd week_1_basics_n_setup/
(base) abdulwahab@de-zoomcamp:~/data-engineering-zoomcamp/week_1_basics_n_setup$ cd 1_terraform_gcp/
(base) abdulwahab@de-zoomcamp:~/data-engineering-zoomcamp/week_1_basics_n_setup/1_terraform_gcp$ cd terraform/
(base) abdulwahab@de-zoomcamp:~/data-engineering-zoomcamp/week_1_basics_n_setup/1_terraform_gcp/terraform$ export GOOGLE_APPLICATION_CREDENTIALS=~/.gc/igneous-ethos-375619.json
(base) abdulwahab@de-zoomcamp:~/data-engineering-zoomcamp/week_1_basics_n_setup/1_terraform_gcp/terraform$ gcloud auth activate-service-account --key-file $GOOGLE_APPLICATION_CREDENTIALS
Activated service account credentials for: [471234288639-compute@developer.gserviceaccount.com]
(base) abdulwahab@de-zoomcamp:~/data-engineering-zoomcamp/week_1_basics_n_setup/1_terraform_gcp/terraform$ terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.50.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
(base) abdulwahab@de-zoomcamp:~/data-engineering-zoomcamp/week_1_basics_n_setup/1_terraform_gcp/terraform$ terraform plan
var.project
  Your GCP Project ID

  Enter a value: igneous-ethos-375619

google_storage_bucket.data-lake-bucket: Refreshing state... [id=dtc_data_lake_igneous-ethos-375619]
google_bigquery_dataset.dataset: Refreshing state... [id=projects/igneous-ethos-375619/datasets/trips_data_all]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
(base) abdulwahab@de-zoomcamp:~/data-engineering-zoomcamp/week_1_basics_n_setup/1_terraform_gcp/terraform$ terraform apply
var.project
  Your GCP Project ID

  Enter a value: igneous-ethos-375619

google_storage_bucket.data-lake-bucket: Refreshing state... [id=dtc_data_lake_igneous-ethos-375619]
google_bigquery_dataset.dataset: Refreshing state... [id=projects/igneous-ethos-375619/datasets/trips_data_all]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```

Below are the images for the available resources in Google Cloud:

## BigQuery Schema Screenshot showing the available resource

![alt text](https://github.com/AbdulWahabAbrar/de-zoomcamp-2023/blob/main/BigQuery.png "BigQuery Schema")

## Clous Storage Screenshot showing the available bucket

![alt text](https://github.com/AbdulWahabAbrar/de-zoomcamp-2023/blob/main/Cloud%20Storage%20.png "Cloud Storage")
