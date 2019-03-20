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
