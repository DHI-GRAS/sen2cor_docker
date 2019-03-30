# AWS Batch REF Internal Notes

Make sure you have AWS CLI installed. If you don't follow the steps [here](https://docs.aws.amazon.com/batch/latest/userguide/get-set-up-for-aws-batch.html).

In the AWS Batch console, before you can run a job you must:

1. Create a **job definition**.
2. Create a **compute environment**.
3. Create a **job queue**.


Several types of IAM roles are needed for AWS Batch jobs:

- In job definitions, a *job role* is needed. This is for:
- In the compute environment:
	+ a *service role* is for:
		* The batch service to start and stop the resources it needs to run the job.
	+ an *instance role* is for:
		- The individual EC2s that your ECS containers are running on to have the permission to connect to other AWS resources (S3 or a database, for example).


## Compute environments

There are types of instances:

- On-Demand
- Spot

### Maximum Price

If you select Spot instances, you must set the max percentage of a standard on-demand price that you're willing to pay for a spot instance. You will use any resource that is up to that maximum percentage price you set. 

For example, if the current market price of a Spot instance is 28% of the On-Demand price and your maximum price is 40% of the On-Demand price, you will pay 28% of the On-Demand price.

### Spot fleet role

If you select Spot instances, you'll also need to assign a Spot fleet role.

- This role grants the Spot Fleet pemission to bid on, launch, tage, and terminate instances on your behalf.

- **Not sure what this means:** "You must also have the AWSServiceRoleForEC2Spot and AWSServiceRoleForECSSpotFleet *service-linked* roles for Amazon EC2 Spot and Spot Fleet."

### Allowed instance types

 
Choose the instance type (ex- m5.xlarge), instance family (m5, p3, etc), or 'optimal' (picks instance types from C, M and R families that match the demand of job queues).

### Launch template

?

### Launch template version

  
Presumably version control for launch templates.


### Minimum vCPUs


"By keeping this set to 0 you will not have instance time wasted when there is no work to be run. If you set this above zero you will maintain that number of vCPUs at all times."


### Desired vCPUs 

Number of EC2 vCPUs that compute environment should launch with. AWS Batch can increase desired vCPUs in computer environment and add EC2 instances up to the maximum vCPUs (or decrease back down to min vCPUs).

### Maximum vCPUs


Max number compute environment can scale out to, irregardless of job queue. Note, this is also cap by the number of EC2 instance type limits. 

### Enable user-specified Ami ID
AWS Batch compute environments have their own AWS ECS-optimized AMI for compute resources. It's possible to use your own AMI as well if you check this box. (leaving at default for now).


## Job Queues

"Jobs are submitted to a job queue, where they reside until they are able to be scheduled to run in a compute environment. An AWS account can have multiple job queues. For example, you might create a queue hat uses Amazon EC2 Spot Instances for low-priority jobs. Job queues have a priority that is used by the scheduler to determine which jobs in the which queue should be evaluated for execution first."


## Configure Sen2Cor container to talk to S3

Docker containers' storage driver API (https://docs.docker.com/registry/storage-drivers/) is versatile enough to cover both local filesystems as well as common key:value object store systems such as AWS S3 or GCP GCS. 



## Question: How does a file from a batch job get copied from S3 to a machine's local disk?

```
docker run -it -v ~/radiant/sen2cor_test_data/aux_data:/Sen2Cor-02.05.05-Linux64/lib/python2.7/site-packages/sen2cor/aux_data -v ~/radiant/sen2cor_test_data/unzipped_scenes:/var/sentinel2_data/unzipped_scenes 939788573396.dkr.ecr.us-west-2.amazonaws.com/sen2cor
```


```
/Sen2Cor-02.05.05-Linux64/bin/L2A_Process /var/sentinel2_data/unzipped_scenes/S2A_MSIL1C_20190122T075221_N0207_R135_T36NXF_20190122T091707.SAFE --resolution=10 --GIP_L2A /root/sen2cor/2.5/cfg/L2A_GIPP_with_dem.xml
```


NOTE: NEED TO MAKE SURE CAN RETRIEVE DEM FROM PUBLIC URL OTHERWISE NEED TO CONFIGURE GATEWAY OR PUT DEM IN S3


to-do:

- make sure batchit and aws cli are installed (and configured) on docker image. https://github.com/base2genomics/batchit
- put correction data on efs/
- configure above scripts to have product-ids as pass-ins
- write container run script using batchit as an example (pull in container environmental vairables)

test!

Notes on 2 primary ways to mount EFS to an ECS container:

- Docker Volumes
	+ built-in local driver or a thrid-party volume driver can be used. if third-party driver is used, it should be installed on the container instance beffore task is launched. Docker volumes are managed by docker and a directory is created in `/var/lib/docker/volumes` on the container instance that contains the volume data.
- Bind Mounts

need to configure container instance AMI to mount the Amazon EFS file system *before* the Docker daemon starts. 

Thus, will need to create custom ECS AMI. 

In order to create AMI, first setup EC2 with ECS container and follow [tutorial details](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_container_instance.html) to mount EFS. Then, once you've verified that the running container has access to the mounted volumes, save AMI. Details on creating the actual compute resource AMI are [here](https://docs.aws.amazon.com/batch/latest/userguide/create-batch-ami.html).

decided to use ami instead.
```
aws s3 ls s3://sentinel-s2-l1c/products/2019/1/22/S2A_MSIL1C_20190122T075221_N0207_R135_T36NXF_20190122T091707 --request-payer requester
```
lists the file.

need to decide:

1. should file be copied before docker daemon launch and then mounted or after launch? if before, docker run command is more similar to laptop
2. think about what environmental variables can be set with batchit - are these set in the host instance or in the container?


important:

create new ecs spot configuration and autoscaling group with new ami!