# Dataflow-deployment
Working Dataflow example deployment


1. Setup Maven 
2. Ensure that export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-11.0.10.jdk/Contents/Home is set and JDK 11.0.10 is installed.
```
mvn archetype:generate \
      -DarchetypeGroupId=org.apache.beam \
      -DarchetypeArtifactId=beam-sdks-java-maven-archetypes-examples \
      -DarchetypeVersion=2.27.0 \
      -DgroupId=org.example \
      -DartifactId=word-count-beam \
      -Dversion="0.1" \
      -Dpackage=org.apache.beam.examples \
      -DinteractiveMode=false
```

2. CD into word-count-beam
   ```       pom.xml               src
    ```
3. Execute the following:
```
mvn compile exec:java \    ✔  at 16:36:34 
      -Dexec.mainClass=org.apache.beam.examples.WordCount \
      -Dexec.args="--output=counts"
```

4. Submit the job manually to Dataflow with the following:


mvn -Pdataflow-runner compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount -Dexec.args="--project=proxysql-298420 --stagingLocation=gs://staging-bucket-foo/staging --output=gs://staging-bucket-foo/output --runner=DataflowRunner --region=europe-west2"


Note: Ensure that prerequisite buckets have been created.


Example output:
```
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< org.example:word-count-beam >---------------------
[INFO] Building word-count-beam 0.1
[INFO] --------------------------------[ jar ]---------------------------------
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/beam/beam-runners-google-cloud-dataflow-java/2.27.0/beam-runners-google-cloud-dataflow-java-2.27.0.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/beam/beam-runners-google-cloud-dataflow-java/2.27.0/beam-runners-google-cloud-dataflow-java-2.27.0.pom (24 kB at 59 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/com/google/apis/google-api-services-clouddebugger/v2-rev20200501-1.30.10/google-api-services-clouddebugger-v2-rev20200501-1.30.10.pom
Downloaded from central: https://repo.maven.apache.org/maven2/com/google/apis/google-api-services-clouddebugger/v2-rev20200501-1.30.10/google-api-services-clouddebugger-v2-rev20200501-1.30.10.pom (5.3 kB at 51 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/com/google/apis/google-api-services-dataflow/v1b3-rev20200713-1.30.10/google-api-services-dataflow-v1b3-rev20200713-1.30.10.pom
Downloaded from central: https://repo.maven.apache.org/maven2/com/google/apis/google-api-services-dataflow/v1b3-rev20200713-1.30.10/google-api-services-dataflow-v1b3-rev20200713-1.30.10.pom (5.3 kB at 51 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/beam/beam-runners-google-cloud-dataflow-java/2.27.0/beam-runners-google-cloud-dataflow-java-2.27.0.jar
Downloading from central: https://repo.maven.apache.org/maven2/com/google/apis/google-api-services-dataflow/v1b3-rev20200713-1.30.10/google-api-services-dataflow-v1b3-rev20200713-1.30.10.jar
Downloading from central: https://repo.maven.apache.org/maven2/com/google/apis/google-api-services-clouddebugger/v2-rev20200501-1.30.10/google-api-services-clouddebugger-v2-rev20200501-1.30.10.jar
Downloaded from central: https://repo.maven.apache.org/maven2/com/google/apis/google-api-services-clouddebugger/v2-rev20200501-1.30.10/google-api-services-clouddebugger-v2-rev20200501-1.30.10.jar (71 kB at 905 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/com/google/apis/google-api-services-dataflow/v1b3-rev20200713-1.30.10/google-api-services-dataflow-v1b3-rev20200713-1.30.10.jar (354 kB at 3.0 MB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/beam/beam-runners-google-cloud-dataflow-java/2.27.0/beam-runners-google-cloud-dataflow-java-2.27.0.jar (464 kB at 1.8 MB/s)
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ word-count-beam ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/cwills/dataflow-wordcount/terraform-google-dataflow/examples/simple_example/word-count-beam/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ word-count-beam ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- exec-maven-plugin:1.6.0:java (default-cli) @ word-count-beam ---
Feb 06, 2021 4:47:15 PM org.apache.beam.sdk.extensions.gcp.options.GcpOptions$GcpTempLocationFactory tryCreateDefaultBucket
INFO: No tempLocation specified, attempting to use default bucket: dataflow-staging-us-central1-196122731815
Feb 06, 2021 4:47:17 PM org.apache.beam.runners.dataflow.DataflowRunner fromOptions
INFO: PipelineOptions.filesToStage was not specified. Defaulting to files from the classpath: will stage 210 files. Enable logging at DEBUG level to see which files will be staged.
Feb 06, 2021 4:47:19 PM org.apache.beam.runners.dataflow.DataflowRunner run
INFO: Executing pipeline on the Dataflow Service, which will have billing implications related to Google Compute Engine usage and other Google Cloud Services.
```



Via terraform :

Example terraform main.tf


```
provider "google" {
  version = "~> 2.18.0"
  region  = var.region
}

resource "random_id" "random_suffix" {
  byte_length = 4
}

locals {
  gcs_bucket_name = "tmp-dir-bucket-${random_id.random_suffix.hex}"
}

module "vpc" {
  source       = "terraform-google-modules/network/google"
  version      = "~> 1.0"
  project_id   = var.project_id
  network_name = "dataflow-network"

  subnets = [
    {
      subnet_name   = "dataflow-subnetwork"
      subnet_ip     = "10.1.3.0/24"
      subnet_region = "europe-west2"
    },
  ]

  secondary_ranges = {
    dataflow-subnetwork = [
      {
        range_name    = "my-secondary-range"
        ip_cidr_range = "192.168.64.0/24"
      },
    ]
  }
}

module "dataflow-bucket" {
  source        = "../../modules/dataflow_bucket"
  name          = local.gcs_bucket_name
  region        = var.region
  project_id    = var.project_id
  force_destroy = var.force_destroy
}

module "dataflow-job" {
  source                = "../../"
  project_id            = var.project_id
  name                  = "wordcount-terraform-example-foo1"
  on_delete             = "cancel"
  region                = var.region
  zone                  = var.zone
  max_workers           = 1
  template_gcs_path     = "gs://dataflow-templates/latest/Word_Count"
  temp_gcs_location     = module.dataflow-bucket.name
  service_account_email = var.service_account_email
  network_self_link     = module.vpc.network_self_link
  subnetwork_self_link  = module.vpc.subnets_self_links[0]
  machine_type          = "n1-standard-1"

  parameters = {

    inputFile = "gs://test_template/shakespeare/kinglear.txt"
    output    = "gs://${local.gcs_bucket_name}/output/my_output"

  }
}




module "dataflow-job-2" {
  source                = "../../"
  project_id            = var.project_id
  name                  = "wordcount-terraform-example-foo3"
  on_delete             = "cancel"
  region                = var.region
  zone                  = var.zone
  max_workers           = 1
  template_gcs_path     = "gs://dataflow-templates/latest/Word_Count"
  temp_gcs_location     = module.dataflow-bucket.name
  service_account_email = var.service_account_email
  network_self_link     = module.vpc.network_self_link
  subnetwork_self_link  = module.vpc.subnets_self_links[0]
  machine_type          = "n1-standard-2"

  parameters = {
    inputFile = "gs://test_template/shakespeare/kinglear.txt"
    output    = "gs://${local.gcs_bucket_name}/output/my_output"


  }
}



```
