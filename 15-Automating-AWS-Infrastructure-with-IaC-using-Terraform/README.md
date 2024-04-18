# Automating AWS Infrastructure with IaC using Terraform Part 1
We are going to use Terraform to build AWS Infrastructure for 2 website as shown in the diagram below:

![architecture](./images/architecture.png)

## Prerequisites before you begin writing Terraform code
* Open your terminal and install AWSCLI Version 2 using the commands shown below:

```sh
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
```

![curl awscli](./images/1.%20curl%20awscli.png)

```sh
sudo installer -pkg ./AWSCLIv2.pkg -target /
```

![sudo installer awscliv2](./images/1.%20sudo%20installer%20awscli.png)

* Install terraform using the command shown below:

```sh
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

![brew install terraform](./images/1.%20brew%20install%20terraform.png)

_**Note**: If you do not have homebrew installed on your macOS, [install it here](https://brew.sh)_.

* Create an IAM user, name it `terraform` _(ensure that the user has only programatic access to your AWS account)_ and grant this user `AdministratorAccess` permissions.

* Copy the **secret access key** and **acces key ID** and save them in a notepad temporarily.

* Run the `aws configure` command to have access to AWSCLI and paste the the **Access Keys** you copied when creating the `terraform` IAM user.

![aws configure](./images/1.%20aws%20configure.png)

* Create an `s3` bucket using the command shown below:

```sh
aws s3api create-bucket --bucket <bucket_name> --region <aws_region>
```

![create s3 bucket](./images/1.%20create-bucket.png)

* Install python and boto3 using the following commands:

```sh
brew install python
```

![install python](./images/1.%20brew%20install%20python.png)

```sh
python -m ensurepip
```

![python -m ensurepip](./images/1.%20python%20-m%20ensurepip.png)

```sh
pip install boto3
```

![pip install boto3](./images/1.%20pip%20install%20boto3.png)

```sh
python3 -m pip install --upgrade pip
```

* Configure programmatic access from your workstation to connect to AWS using thr access keys copied above and a [Python SDK (boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html). Ensure you have [Python 3.6](https://www.python.org/downloads/) or higher on your workstation. Run the `python` command and paste the codebase shown below:

```sh
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```

![import boto3](./images/1.%20import%20boto3%20s3.png)

## VPC | Subnets | Security Groups

Open your Visual Studio Code and:
* Create a folder called `pbl`
* Create a file in the folder and name it `main.tf`

```sh
mkdir pbl
cd pbl
touch main.tf
```

![mkdir pbl touch main.tf](./images/2.%20mkdir%20pbl%20cd%20pbl%20touch%20main.tf.png)

The setup should look like the image shown below:

![setup](./images/2.%20setup.png)

_Set up Terraform CLI as per [this instruction](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)_.

### Provider & VPC Resource Section
* Add `AWS` as a provider and a resource to create a VPC in the `main.tf` file. _(The **Provider** block informs Terraform that we intend to build infrastructure within AWS and the **Resource** block will create a VPC)_.

```sh
provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
}
```

![aws provider and vpc](./images/3.%20aws%20provider%20and%20vpc.png)

**Note**: You can change the configuration above to create your VPC in a region that is closer to you.

* Run the `terraform init` command to download the necessary plugins for Terraform to work. _(These plugins are used by `providers` and `provisioners`.)_ 

![terraform init](./images/3.%20terraform%20init.png)

**Note**: There is a new directory created called `.terraform\...`, this is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means you must execute `terraform init` again to download them.

![.terraform\...](./images/3.%20terraform\....png)

* Let us create the only resource we just defined `aws_vpc`. But before we do that, run the `terraform plan` command to check what Terraform intends to create before we tell it to go ahead and create it.

![terraform plan](./images/3.%20terraform%20plan.png)

* Run the `terraform apply` command and type **yes** if you are satisfied with the planned changes.

![terraform apply](./images/3.%20terraform%20apply.png)

The following observations were made after executing the `terraform apply` command:

1. A new file is created `terraform.tfstate`. This is how terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added or destroyed based on the entire terraform code that is being developed.

2. If you also observed closely, you would realise that another file `terraform.tfstate.lock.info` gets created during planning and apply. But this file gets deleted immediately. This is what Terraform uses to track who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same. It prevents duplicates and conflicts.

Its content is usually in the format shown below:

It is a `json` format file that stores information about a user: user's `ID`, what operation he/she is doing, timestamp and location of the `state` file.

## Subnets Resource Section
According to our architectural design, we require 6 subnets:
* 2 Public Subnets
* 2 Private Subnets for Webservers
* 2 Private Subnets for Data Layer

### Creating the first 2 Public Subnets
* Copy and paste the configuration below to the `main.tf` file:

```sh
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"
}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1b"
}
```

![create public subnets](./images/4.%20create%202%20public%20subnets.png)

_In order to create 2 subnets, we must declare 2 **resource blocks** (i.e. one for each subnet). The `vpc_id` argument is used to reference the value of the VPC `id` by setting it to `aws_vpc.main.id`. This way, Terraform knows inside which VPC to create the subnet._

* Run the `terraform plan` command.

![terraform plan](./images/4.%20terraform%20plan.png)

* Run the `terraform apply -auto-approve` command.

![terraform apply -auto-approve](./images/4.%20terraform%20apply%20-auto-approve.png)

The following observations were made after executing the `terraform apply -auto-approve` command:

1. **Hard Coded Values**: Rememeber our best practice hint from the beginning? Both the `availability_zone` and `cidr_block` arguments are hard coded. We should always endeavour to make our configurations dynamic and reusable.

2. **Multiple Resource Block**: Notice that we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resoure that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So we need to optimize this by introducing a `count` argument.

#### Fixing The Problems By Code Refactoring
The following steps are taken to improve our code by refactoring it:

Run the `terrafom destroy -auto-approve` command to destroy to the current infrastructure. _**Note:** Do not destroy an infrastructure that has been deployed to production._

![terraform destroy -auto-approve](./images/4.%20terraform%20destroy%20-auto-approve.png)

**Fixing Hard Coded Values**: Variables are introduced to remove hard coding. 

* Starting with the provider block, declare a variable named `region`, give it a default value and update the provider section by referring to the declared variable.

```sh
variable "region" {
    default = "us-east-1"
}

provider "aws" {
    region = var.region
}
```

* Do the same to the `cidr` value in the `vpc` block and all the other arguments.

```sh
variable "region" {
    default = "us-east-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

provider "aws" {
    region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support
}
```

![fixing hard coded values](./images/5.%20fixing%20hard%20coded%20values.png)

**Fixing Multiple Resource Blocks**: This is where concepts like **Loops & Data Sources** are introduced.

Terrafrom has a functionality that allows us to pull data which exposes information to us. For example, every region has Availability Zones (AZ). Different regions have from 2 to 4 Availability Zones. With over 20 geographic regions and over 70 Availability Zones served by AWS, it is impossible to keep up with the latest information by hard coding the names of Availability Zones. Hence, we will explore the use of Terraforms's **Data Sources** to fetch the information outside of Terraform. In this case, from **AWS**.

Let us fetch Availability Zones from AWS and replace the hard coded value in the subnet's `availabilty_zone` section.

```sh
# Get list of availability zones
data "aws_availability_zones" "available" {
    state = "available"
}
```

To make use of this new `data` resource, we will need to introduce a `count` argument in the subnet block as shown below:

```sh
# Create public subnet1
resource "aws_subnet" "public" { 
    count                   = 2
    vpc_id                  = aws_vpc.main.id
    cidr_block              = "172.16.1.0/24"
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```

Let us quickly understand what is going on here.
1. `count` tells us that we need 2 subnets. Therefore, Terraform will invoke a loop to create 2 subnets.
2. `data` resource will return a list object that contains a list of Availabilty Zones. Internally, terraform will receive data like this:

```sh
  ["us-east-1a", "us-east-1b", "us-east-1c", "us-east-1d", "us-east-1e", "us-east-1f"]
```

Each of them is an index, the first one is index `0` while the other is index `1`. If the data returned had more than 2 records then the index numbers would continue to increment.

Therefore, each time Terraform goes into a loop to create a subnet, it must be created in the retrieved Availability Zone from the list. Each loop will need the index number to determine what Availability Zone the subnet will be created. That is why we have `data.aws_availability_zones.available.names[count.index]` as the value for `availability_zone`. When the first loop runs, the first index will be `0` therefore the Availability Zone will be `us-east-1a`. The pattern will repeat for the second loop.

But we still have a problem. When you the `terraform apply -auto-approve` command with this configuration, it may succeed for the first time but by the time it goes into the second loop, it will fail because we still have `cidr_block` hard coded. The same `cidr_block` cannot be created twice within the same VPC. So we have little more work to do.

Run the `terraform destroy -auto-approve` command to remove all the AWS resources.

#### Let's Make The `cidr_block` Dynamic
The `cidrsubnet()` function is introduced, it accepts 3 parameters. The first use cases will be updating the configuration then exploring its internals.

```sh
# Create public subnet1
resource "aws_subnet" "public" { 
    count                   = 2
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```

![cidr_block dynamic](./images/6.%20cidr_block%20dynamic.png)

The `cidrsubnet()` function works like an algorithm to dynamically create a subnet CIDR per Availability Zone. Regardless of the number of subents created, it takes care of the cidr value per subnet.

Its parameters are:
1. `prefix` parameter must be given in CIDR notation same as for VPC.
2. `newbits` parameter is the number of additional bits with which to extend the prefix. For example, if given a prefix ending with /16 and a newbits value of 4, the resulting subnet address will have a length 0f /20.
3. `netnum` parameter is a whole number that can be represented as a binary integer with no more than `newbits` binary digits which will be used to populate the additional bits added to the prefix.

You can experiment how this works by entering the `terraform console` and keep changing the figure to see the output.
* Open your terminal and run `terraform cosole` command
* Type `cidrsubnet("172.16.0.0/16", 4, 0)` and hit enter
* Notice the output has an increase in the prefix length by 4
* Type `exit` and hit enter to log out of the console.

![terraform console](./images/6.%20terraform%20console.png)

#### Removing Hard Coded `count` Value
If we cannot hard code a value we want, then we need a way to dynamically provide the value based on some input. Since the `data` resource returns all the Availability Zones within a region, it makes sense to count the number of Availability Zones returned and pass that number to the `count` argument.

To do this, we can introduce the `length()` function which basically determines the length of a give list, map or string.

Since `data.aws_availability_zones.available.names` returns a list like `["us-east-1a", "us-east-1b", "us-east-1c", "us-east-1d", "us-east-1e", "us-east-1f"]` we can pass it into a `length` function and get number of the Availability Zones.

To test this take the following steps:
* Run the `terraform console` command
* Paste the code below:
```sh
length(["us-east-1a", "us-east-1b", "us-east-1c", "us-east-1d", "us-east-1e", "us-east-1f"])
```
* The output will be `6`.

![removing hard coded count value](./images/7.%20removing%20hard%20coded%20count%20-%20value%20terraform%20console.png)

Now we can simply update the public subnet block like this:

```sh
# Create public subnet1
resource "aws_subnet" "public" { 
    count                   = length(data.aws_availability_zones.available.names)
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```

The following observations were made:
1. What we have now is sufficient to create the subnet resource required but it is not satisfying our business requimrent of just `2` subnets.
2. The `length` function will return number **6** to the `count` argument but what we actually need is `2`.

Now lets fix this:
* Declare a variabe to store the desired number of public subnets and set the default value to `2`.

```sh
variable "preferred_number_of_public_subnets" {
    default = 2
}
```

* Update the `count` argument with a condito. Terraform needs to check first if there is a desired number of subnets, otherwise use the data returned by the `length` function. See how it is presented below:

```sh
# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```

![main.tf](./images/8.%20main_tf.png)

Now lets break it down:
1. `var.preferred_number_pf_public_subnets == null` checks if the value of the variable is set to `null` or has some value defined.
2. `?` and `length(data.aws_availability_zones.available.names)` means if the first part is true, then use this. In other words, if preferred number of public subents is `null` (not known) then set the value to the data returned by `length` function.
3. `:` and `var.preferred_number_of_public_subnets` means if the first condition is false (i.e. preferred number of public subnets is `not null`) then set the value to whatever is defined in `var.preferred_number_of_public_subnets`.

The entire configuration on the `main.tf` file
should now look like this:

```sh
variable "region" {
    default = "us-east-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

  variable "preferred_number_of_public_subnets" {
    default = 2
}

provider "aws" {
    region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
}

# Get list of availability zones
data "aws_availability_zones" "available" {
    state = "available"
}

# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```

![main.tf](./images/9.%20main_tf.png)

_**Note**: Try changing the value of `preferred_number_of_public_subnets` variable to `null` and notice how many subnets get created._

Run the `terraform apply -auto-approve` command to apply the changes.

![terraform apply -auto-approve](./images/8.%20terraform%20apply%20-auto-approve.png)

Run the `terraform destroy -auto-approve` command to delete all the AWS resources.

![terraform destroy -auto-approve](./images/8.%20terraform%20destroy%20-auto-approve.png)

## Variables & tfvars
### Intoducing variables.tf & terraform.tfvars
Instead of having a long list of variables in the `main.tf` file, we can make our code a lot more readable and better structured by moving put some parts of the configuration content to other files. Hence, we will put all variable declarations in a separate file and provide non-default values to each of them.

The following steps are taken to achieve this:
1. Create a new file and name it `variables.tf`
2. Copy all the variable declaratiosn into the new file.
3. Create another file and name it `teraform.tfvars`
4. Set values for each of the variables.

#### The `main.tf` file Should look like this

```sh
# Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
}

# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```

![main.tf](./images/9.%20main_tf.png)

#### The `variables.tf` file should look like this

```sh
variable "region" {
      default = "us-east-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

  variable "preferred_number_of_public_subnets" {
      default = null
}
```

![variables.tf](./images/9.%20variables_tf.png)

#### The `terraform.tfvars` file should look like this

```sh
region = "us-east-1"

vpc_cidr = "172.16.0.0/16" 

enable_dns_support = "true" 

enable_dns_hostnames = "true"  

preferred_number_of_public_subnets = 2
```

![terraform.tfvars](./images/9.%20terraform_tfvars.png)

Run the `terraform plan` to ensure that everything works.

![terraform plan](./images/10.%20terraform%20plan.png)

Run the `terraform apply -auto-approve` command to execute the planned changes.

![terraform apply](./images/10.%20terraform%20apply.png)