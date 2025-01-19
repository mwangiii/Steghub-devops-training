# Automate Infrastructure With IaC using Terraform 3 (Refactoring)

In two of our previous projects we have developed AWS Infrastructure code using Terraform and tried to run it from our local workstation. Now it is time to introduce some more advanced concepts and enhance our code.

Firstly, we will explore alternative Terraform [backends](https://developer.hashicorp.com/terraform/language/settings/backends/configuration) for learning purposes.

## Introducing Backend on S3

Each Terraform configuration can specify a backend, which defines where and how operations are performed, where state snapshots are stored, etc. Take a peek into what the states file looks like. It is basically where terraform stores all the state of the infrastructure in `json` format.

So far, we have been using the default backend, which is the `local backend` - it requires no configuration, and the states file is stored locally. This mode can be suitable for learning purposes, but it is not a robust solution, so it is better to store it in some more reliable and durable storage.
The second problem with storing this file locally is that, in a team of multiple DevOps engineers, other engineers will not have access to a state file stored locally on your computer.

To solve this, we will need to configure a backend where the state file can be accessed remotely other DevOps team members. There are plenty of different standard backends supported by Terraform that you can choose from. Since we are already using AWS - we can choose an [S3 bucket as a backend](https://developer.hashicorp.com/terraform/language/settings/backends/s3).

Another useful option that is supported by S3 backend is [State Locking](https://developer.hashicorp.com/terraform/language/state/locking) - it is used to lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state. State Locking feature for S3 backend is optional and requires another AWS service - [DynamoDB](https://aws.amazon.com/dynamodb/).

Let us configure it.

Here is our plan to re-initialize terraform to use S3 backend:

- Add S3 and DynamoDB resource blocks before deleting the local state file
- Update terraform block to introduce backend and locking
- Re-initialize terraform
- Delete the local `tfstate` file and check the one in S3 bucket
- Add `outputs`
- `terraform apply`

To get to know how lock in DynamoDB works, read [the following article](https://angelo-malatacca83.medium.com/aws-terraform-s3-and-dynamodb-backend-3b28431a76c1)


### - Create a file and name it `backend.tf`. Add the below code and replace the name of the S3 bucket we created in [previous project](https://github.com/alagbaski/steghub-projects/blob/main/Automate%20Infrastructure%20With%20IaC%20using%20Terraform%20-%20Part%201/Project.md).

```hcl
# Note: Since buckets are unique globally in AWS, you must give it a unique name.

resource "aws_s3_bucket" "terraform_state" {
  bucket = "opsmen-terraform-bucket"
  
  lifecycle {
    prevent_destroy = true
  }
}

# Enable versioning so we can see the full revision history of our state files

resource "aws_s3_bucket_versioning" "versioning" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Enable server-side encryption by default

resource "aws_s3_bucket_server_side_encryption_configuration" "state_encryption" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "AES256"
    }
  }
}
```
We must be aware that Terraform stores secret data inside the state files. Passwords, and secret keys processed by resources are always stored in there. Hence, we must consider to always enable encryption. You can see how we achieved that with [server_side_encryption_configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/serv-side-encryption.html).

### - Next, create a `DynamoDB` table to handle locks and perform consistency checks.

In previous projects, locks were handled with a local file as shown in __`terraform.tfstate.lock.info`__. Since we now have a team mindset, causing us to configure S3 as our backend to store state file, we will do the same to handle locking. Therefore, with a cloud storage database like DynamoDB, anyone running Terraform against the same infrastructure can use a central location to control a situation where Terraform is running at the same time from multiple different people.

Dynamo DB resource for locking and consistency checking:

```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }

  lifecycle {
    prevent_destroy = true
  }
}
```

Terraform expects that both S3 bucket and DynamoDB resources are already created before we configure the backend. So, let us run `terraform apply` to provision resources.

**Output 1:**
![S3](./images/s3-bucket.PNG)

**Output 2:**
![DynamoDB](./images/dynamo-db-table.PNG)


### - Configure S3 Backend

```hcl
terraform {
  backend "s3" {
    bucket         = "opsmen-terraform-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```
Now its time to re-initialize the backend. Run `terraform init` and confirm you are happy to change the backend by typing `yes`.

**Output:**

![Init](./images/terraform-init.PNG)

### - Verify the changes

Before doing anything if you opened AWS now to see what happened you should be able to see the following:

- `.tfstatefile` is now inside the S3 bucket

  **Output:**
  ![tfstate](./images/s3-tfstate.PNG)

- DynamoDB table which we create has an entry which includes state file status

  **Output:**
  ![dynamodb](./images/dynamo-db-table.PNG)

- Navigate to the DynamoDB table inside AWS and leave the page open in your browser. Run `terraform plan` and while that is running, refresh the browser and see how the lock is being handled:

  ![lock](./images/acquire-state-lock-01.PNG)

- After `terraform plan` completes, refresh DynamoDB table.

  ![lock](./images/acquire-state-lock-02.PNG)

### - Add Terraform Output

Before we run `terraform apply` let us add an output so that the S3 bucket Amazon Resource Names ARN and DynamoDB table name can be displayed.

Create a new file and name it output.tf and add below code.

```hcl
output "s3_bucket_arn" {
  value       = aws_s3_bucket.terraform_state.arn
  description = "The ARN of the S3 bucket"
}

output "dynamodb_table_name" {
  value       = aws_dynamodb_table.terraform_locks.name
  description = "The name of the DynamoDB table"
}
```

Now we have everything ready to go!

### - Let us run `terraform apply`

Terraform will automatically read the latest state from the S3 bucket to determine the current state of the infrastructure. Even if another engineer has applied changes, the state file will always be up to date.

Now, let's head over to the S3 console again, refresh the page, and click the grey “Show” button next to “Versions.” We should now see several versions of our terraform.tfstate file in the S3 bucket:

**Output 1:**
![Apply](./images/terraform-apply-01.PNG)

**Output 2:**
![Version](./images/s3-tfstate-versions.PNG)

With help of remote backend and locking configuration that we have just configured, collaboration is no longer a problem.

However, there is still one more problem: Isolation Of Environments. Most likely we will need to create resources for different environments, such as: `Dev`, `sit`, `uat`, `preprod`, `prod`, etc.

This separation of environments can be achieved using one of two methods:

a. Terraform Workspaces

b. Directory based separation using `terraform.tfvars` file


## When to use `Workspaces` or `Directory`

To separate environments with significant configuration differences, use a directory structure. Use workspaces for environments that do not greatly deviate from each other, to avoid duplication of your configurations. Try both methods in the sections below to help you understand which will serve your infrastructure best.

For now, we can read more about both alternatives [here](https://learn.hashicorp.com/tutorials/terraform/organize-configuration) and try both methods ourself, but we will explore them better in next projects.

## Security Groups refactoring with `dynamic block`

For repetitive blocks of code you can use dynamic blocks in Terraform, to get to know more how to use them - watch this video.

### Refactor Security Groups creation with `dynamic blocks`.

The terraform code for refactoring the security groups with dynamic blocks is available in this [repository](https://github.com/francdomain/project_18_terraform_code)

## EC2 refactoring with `Map` and `Lookup`

Remember, every piece of work you do, always try to make it dynamic to accommodate future changes. [Amazon Machine Image (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) is a regional service which means it is only available in the region it was created. But what if we change the region later, and want to dynamically pick up `AMI IDs` based on the available AMIs in that region? This is where we will introduce [Map](https://developer.hashicorp.com/terraform/language/functions/map) and [Lookup](https://developer.hashicorp.com/terraform/language/functions/lookup) functions.

Map uses a key and value pairs as a data structure that can be set as a default type for variables.

```hcl
variable "images" {
  type = "map"
  default = {
    us-east-1 = "image-1234"
    us-west-2 = "image-23834"
  }
}
```

To select an appropriate AMI per region, we will use a lookup function which has following syntax: __`lookup(map, key, [default])`__.

> Note: A default value is better to be used to avoid failure whenever the map data has no key.

```hcl
resource "aws_instace" "web" {
  ami  = "${lookup(var.images, var.region), "ami-12323"}
}
```
Now, the lookup function will load the variable `images` using the first parameter. But it also needs to know which of the key-value pairs to use. That is where the second parameter comes in. The key `us-east-1` could be specified, but then we will not be doing anything dynamic there, but if we specify the variable for region, it simply resolves to one of the keys. That is why we have used `var.region` in the second parameter.

## Conditional Expressions

If you want to make some decision and choose some resource based on a condition - you shall use Terraform Conditional Expressions.

In general, the syntax is as following: condition `? true_val : false_val`

Read following snippet of code and try to understand what it means:

```hcl
resource "aws_db_instance" "read_replica" {
  count               = var.create_read_replica == true ? 1 : 0
  replicate_source_db = aws_db_instance.this.id
}
```
- true #condition equals to 'if true'
- ? #means, set to '1`
- : #means, otherwise, set to '0'


## Terraform Modules and best practices to structure your `.tf` codes

By this time, you might have realized how difficult is to navigate through all the Terraform blocks if they are all written in a single long `.tf` file. As a DevOps engineer, you must produce reusable and comprehensive IaC code structure, and one of the tool that Terraform provides out of the box is [Modules](https://developer.hashicorp.com/terraform/language/modules).

Modules serve as containers that allow to logically group Terraform codes for similar resources in the same domain (e.g., Compute, Networking, AMI, etc.). One `root module` can call other `child modules` and insert their configurations when applying Terraform config. This concept makes your code structure neater, and it allows different team members to work on different parts of configuration at the same time.

You can also create and publish your modules to [Terraform Registry](https://registry.terraform.io/browse/modules) for others to use and use someone's modules in your projects.

Module is just a collection of `.tf` and/or `.tf.json` files in a directory.

You can refer to existing child modules from your `root module` by specifying them as a source, like this:

```hcl
module "network" {
  source = "./modules/network"
}
```
> Note that the path to 'network' module is set as relative to your working directory.

Or you can also directly access resource outputs from the modules, like this:

```hcl
resource "aws_elb" "example" {
  # ...

  instances = module.servers.instance_ids
}
```
In the example above, you will have to have module 'servers' to have output file to expose variables for this resource.

## Refactor your project using Modules

Let us review the [repository](https://github.com/francdomain/Automate_Infrastructure_With_IaC_using_Terraform_2)of project 17, you will notice that we had a single lsit of long file for creating all of our resources, but that is not the best way to go about it because it maks our code base vey hard to read and understand, and making future changes can bring a lot of pain.

## QUICK TASK:

Break down your Terraform codes to have all resources in their respective modules. Combine resources of a similar type into directories within a `modules` directory, for example, like this:

```css
- modules
  - ALB
  - EFS
  - RDS
  - Autoscaling
  - compute
  - VPC
  - security
```

**Output:**

![modules](./images/modules-dir.PNG)

Each module shall contain following files:

```css
- main.tf (or %resource_name%.tf) file(s) with resources blocks
- outputs.tf (optional, if you need to refer outputs from any of these resources in your root module)
- variables.tf (as we learned before - it is a good practice not to hard code the values and use variables)
```
It is also recommended to configure `providers` and `backends` sections in separate files.

> NOTE: It is not compulsory to use this naming convention.

After you have given it a try, you can check out this [repository](https://github.com/dareyio/PBL-project-18)

It is not compulsory to use this naming convention for guidiance or to fix your errors.

In the configuration sample from the repository, you can observe two examples of referencing the module:

a. Import module as a `source` and have access to its variables via `var` keyword:

```hcl
module "VPC" {
  source = "./modules/VPC"
  region = var.region
  ...
}
```

b. Refer to a module's output by specifying the full path to the output variable by using `module.%module_name%.%output_name%` construction:

```hcl
subnets-compute = module.network.public_subnets-1
```

## Complete the Terraform configuration

Complete the rest of the codes yourself, so, the resulted configuration structure in your working directory may look like this:

```css
└── PBL
    ├── modules
    |   ├── ALB
    |     ├── ... (module .tf files, e.g., main.tf, outputs.tf, variables.tf)
    |   ├── EFS
    |     ├── ... (module .tf files)
    |   ├── RDS
    |     ├── ... (module .tf files)
    |   ├── autoscaling
    |     ├── ... (module .tf files)
    |   ├── compute
    |     ├── ... (module .tf files)
    |   ├── network
    |     ├── ... (module .tf files)
    |   ├── security
    |     ├── ... (module .tf files)
    ├── main.tf
    ├── backends.tf
    ├── providers.tf
    ├── data.tf
    ├── outputs.tf
    ├── terraform.tfvars
    └── variables.tf
```

**Output:**

![module-dir](./images/modules-dir-01.PNG)

## Instantiating the Modules

**Run `terraform init -upgrade`**

**Output:**

![init](./images/init-modules.PNG)

**Run `terraform plan`**

**Output:**

![plan](./images/plan-modules.PNG)

**Run `terraform apply`**

**Output:**

![apply](./images/apply-modules.PNG)

> Note: Ignore the error output stating that `S3` and `DynamoDB` resources is already existing. Furthermore, we will learn how to make terraform stop tracking these resources from the `.terraform.tfstate.tf` to avoid conflicts when we run `terraform destroy`.

## Verifying Resources

**Output VPC:**
![vpc](./images/vpc.PNG)

**Output Subnets:**
![subnet](./images/subnets.PNG)

**Output Security Groups:**
![security](./images/security-groups.PNG)

**Output IGW:**
![igw](./images/igw.PNG)

**Output EIP:**
![eip](./images/eip.PNG)

**Output NAT:**
![nat](./images/nat-gw.PNG)

**Output Routes:**
![route](./images/routes-tables.PNG)

**Output EFS:**
![efs](./images/efs.PNG)

**Output RDS:**
![rds](./images/rds.PNG)

**Output Target Groups:**
![tg](./images/target-groups.PNG)

**Output ALB:**
![alb](./images/load-balancer.PNG)

**Output ASG:**
![asg](./images/auto-scaling-group.PNG)

**Output EC2:**
![ec2](./images/instances.PNG)


**Run `terraform state list`**

**Output 1:**

![state-list](./images/terraform-state-list-01.PNG)

**Output 2:**

![state-list](./images/terraform-state-list-02.PNG)

> Note: In other to make terraform to stop tracking resources from the `tfstate` file, run `terraform state rm <terraform-resource-name>`. E.g __`terraform state rm aws_s3_bucket.terraform_state`__. This was done because, when we run `terraform destroy` it won't destroy our `S3` and `DynamoDB` containing our `.terraform.tfstate` and `.terraform.lock.hcl` in the resources respectively.

Now, the code is much more well-structured and can be easily read, edited and reused by our DevOps team members.

__`BLOCKERS`:__ Our website would not be available because the userdata scripts we addedd to the launch template does not contain the latest endpoints for `EFS`, `ALB` and `RDS` and also our AMI is not properly configutred, so how do we fix this?

Going forward in project 19, we would se how to use packer to create AMIs, Terrafrom to create the infrastructure and Ansible to configure the infrasrtucture.

We will also see how to use terraform cloud for our backends.

### Pro-tips:

1. We can validate our codes before running `terraform plan` with [terraform validate](https://developer.hashicorp.com/terraform/cli/commands/validate) command. It will check if our code is syntactically valid and internally consistent.

2. In order to make our configuration files more readable and follow canonical format and style - We use [terraform fmt](https://developer.hashicorp.com/terraform/cli/commands/fmt) command. It will apply Terraform language style conventions and format our `.tf` files in accordance to them.

### Conclusion

We have been able to develop and refactor AWS Infrastructure as Code with Terraform.

The repository for the terraform modules is available here: [project_18_terraform_modules](https://github.com/francdomain/project_18_terraform_modules)

---