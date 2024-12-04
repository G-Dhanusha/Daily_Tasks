**Implement Cross-Region Replication (Optional)**
    * Set up Cross-Region Replication (CRR) to duplicate data from your primary bucket to a secondary bucket in another AWS region.
    * Make sure versioning is enabled on both buckets and create IAM policies that permit replication.
    * Test the replication by uploading files to the primary bucket and verifying their presence in the secondary bucket.

#### Step 1: Create Two buckets in different regions

* Create Bucket

    * Give unique Bucket name    
    * `Object Ownership` > ACLs enabled
    * `Block Public Access` > Uncheck Box
    * `Bucket Versioning` > Enable

* Create buckets in two different regions

   ![preview](replica/aws1.png)

#### Step2: Create 2 IAM Roles

1) Create an IAM Role name it is `S3-Replication-Role`

    * Create a New Role
        * Click `Roles` > `Create Role`
        * Select `AWS Service` >`S3`
        * Attach Policies: `AmazonS3FullAccess`
        * After Role Creation > Attach `custom inline policy` like below
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": "s3:PutReplicationConfiguration",
                    "Resource": "arn:aws:s3:::source-bucket-replication1"
                }
            ]
        }
        ```
    * Choose S3 (Supports S3 Replication).

        ![preview](replica/aws3.png)
        ![preview](replica/aws2.png)

2) create an IAM role - `s3-batch-operations`

* Navigate to the IAM Console

    * Create a New Role name it like `S3BatchOperationRole`
    * Click `Roles` > `Create Role`
    * Select `AWS Service` >`S3` > `S3 Batch Operations`
    * Attach Policies: `AmazonS3FullAccess`

        ![preview](replica/aws4.png)
        ![preview](replica/aws6.png)
        ![preview](replica/aws5.png)

3) `Destination Bucket` : The Destination bucket before replicating files
from Source bucket
        ![preview](replica/aws8.png)
        
        
4) `Source Bucket`: Upload few files to the Source bucket and Create a Replication Rule on the Source Bucket
and 
    
    ![preview](replica/aws7.png)

    * Create a Replication Rule on the Source Bucket

        * Go to `Management tab` > `Replication Rules`
        * `Create Replication Rule`
            * Name the Rule like 's3Replica'
            * Source bucket:
                * Choose rule scope >  `Apply to all objects in the buckets`
            * Destination bucket
                * select destination bucket > `Browse` (destination bucket)
            * Assign IAM Role:
                * Select the previously created `S3ReplicationRole`

        * click `save`

            ![preview](replica/aws9.png) 
            ![preview](replica/aws10.png)
    
    * After clicking save, you will get popup like below. 
        * Select `No` > `submit`

            ![preview](replica/aws11.png)
            ![preview](replica/aws12.png)

5) Till now we created replication rule, Now `create replication job`

    * Go to `S3` > `Batch Operations `> `Create Job`
    
        ![preview](replica/aws13.png)
        ![preview](replica/aws14.png)
        ![preview](replica/aws15.png)
        ![preview](replica/aws16.png)
        ![preview](replica/aws17.png)

    * Select options like above > `Review and create job`

        ![preview](replica/aws18.png)
        ![preview](replica/aws19.png)

    * Later you can check your object tranfer status like below 
        1) New

            ![preview](replica/aws20.png)
        2) Preparing
            
            ![preview](replica/aws21.png)

        3) Awaiting you conformition to run like below
            ![preview](replica/aws22.png)
            ![preview](replica/aws23.png)
            ![preview](replica/aws24.png)
            
            * After above status , you need to select `job` > click `Run Job` 

                ![preview](replica/aws29.png)
            
        * After clicking `Run Job`. You will get statua like below

            * `Ready`, `Completing` and `completed`

            ![preview](replica/aws25.png)
            ![preview](replica/aws26.png)

* You can check now which all `Source Bucket` objects replicated to `Destination Bucket `
    * Source Bucket before replication, 
        * We have only two files
        ![preview](replica/aws7.png)

        * After replication, report file generated to source path as per provied path in batch 
        ![preview](replica/aws28.png)

    * Destination Bucket 

        * Before Replication

            ![preview](replica/aws8.png)

        * After Replication

            ![preview](replica/aws27.png)
    

[Refer here](https://gargeebhatnagar.medium.com/cross-region-bucket-replication-of-existing-data-using-amazon-s3-batch-operation-996f6bfce90a)







    


