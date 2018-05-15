# Terraform Openstack Tutorial
This tutorial will go through a simple terraform script that creates a web server on an openstack client.

## Prerequisites
- [Install openstack](https://github.com/dcfield/stackinabox.io)
- Log into openstack user in the terminal
- `sudo su - stack`

## Tasks
1. Install terraform
2. Create sample.tf file
3. Create openstack instance using terraform instead of GUI
4. Create + destroy multiple openstack instances using terraform instead of GUI

## 1. Install terraform
- You should be logged into the openstack terminal
- `wget https://releases.hashicorp.com/terraform/0.9.8/terraform_0.9.8_linux_amd64.zip`
- `unzip terraform_0.9.8_linux_amd64.zip`
- `sudo mv terraform /usr/local/bin/`
- Confirm terraform binary is accessible: `terraform --version`



## 2. Create sample.tf file

### Openstack values
First, we need to get some openstack variables in order to fill in our sample.tf file.

#### user_name
- Usually `admin` or `demo`

#### password
- The password for your openstack version. On stackinabox, it is `nomoresecret`

#### auth_url
- In the openstack GUI, go to the `Project` tab => `API Access` => `Download OpenStack RC File` => `Openstack RC File (Identity API v3)`
- Open the downloaded file with a text editor
- Get the value for `OS_AUTH_URL` (eg. `http://10.0.2.15/identity/v3`)
- This is your `auth_url`
- Close the text editor


### Instance values
Now we need some values to create out instance. We will be using an image and a flabor, so we need these IDs.

#### image_id
- `Project` tab => `Compute` => `Images`
- Click on the image name and grab the ID
- If no image exists, create one and get its ID

#### flavor_id
- `Admin` tab => `Compute` => `Flavors`
- Get the ID of one of the available flavors
- eg. I am using m1.tiny with ID 1. The IDs will change between systems and openstack versions

#### network
- `Project` tab => `Network` => `Networks`
- Note the name of the network

#### region
- Go to terminal
- `openstack region list`
- Note the name of a region eg. `RegionOne`


### Create terraform file
- `cd` to openstack root directory
- `cd ~/devstack` (I have devstack installed as part of [stackinabox](https://github.com/dcfield/stackinabox.io))
- Create a new terraform file
- `sudo vi ./terraform_sample.tf`
- Paste the following and input your own values. Anything with a * needs your own values.

````
# Configure the Openstack provider
provider "openstack" {
  user_name   = "*admin"
  tenant_name = "demo"
  password    = "*nomoresecret"
  auth_url    = "*http://10.0.2.15/identity/v3"
}

# Create a web server
resource "openstack_compute_instance_v2" "server_A" {
  name = "server_A"
  image_id = "*12345667-abcd-12cd-1234abcd9876"
  flavor_id = "*1"
  security_groups = ["default"]
  network{
    name = "*test"
  }
  region = "*RegionOne"
}

````
- Exit back to the terminal



## 3. Create openstack instance using terraform instead of GUI
- Initialize, check what we are managing and start the process of terraform
- `terraform init`
- `terraform plan`
- `terraform apply`

- If this is successful, you should see your new instance in openstack at `Project` tab => `Compute` => `Instances`



## 4. Create & destroy multiple openstack instance using terraform instead of GUI
- Go back to sample.tf
- We can change and add more instances all through terraform

````
# Configure the OpenStack Provider
provider "openstack" {
  *user_name   = "admin"
  tenant_name = "demo"
  *password    = "nomoresecret"
  *auth_url    = "http://10.0.2.15/identity/v3"
}

# Change server_A’s name to something else to see what happens
resource "openstack_compute_instance_v2" "custom_A" {
  name = "custom_A"
  image_id = "12345667-abcd-12cd-1234abcd9876"
  flavor_id = "1"
  security_groups = ["default"]
  network{
    name = "test"
  }
  region = "RegionOne"
}

# Let’s create another instance using the same image and flavour (optional) but a different name
resource "openstack_compute_instance_v2" "custom_B" {
  name = "custom_B"
  image_id = "12345667-abcd-12cd-1234abcd9876"
  flavor_id = "1"
  security_groups = ["default"]
  network{
    name = "test"
  }
  region = "RegionOne"
}

````

- Call terraform
- `terraform plan`
- `terraform apply`

- Go to your instances in the Openstack GUI
- We have destroyed the previous instace and created 2 new ones - all through terraform!
- It is much simpler to call up instances via terraform than to do it manually. All you need are some ID's.
