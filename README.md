# F5-XC-AWS-TGW-SNODE

This repository consists of Terraform templates to bring up a f5-xc-aws-tgw-snode object.

## Usage

- Clone this repo with: `git clone --recurse-submodules https://github.com/cklewar/f5-xc-aws-tgw-snode`
- Enter repository directory with: `cd f5-xc-aws-tgw-snode`
- Obtain F5XC API certificate file from Console and save it to `cert` directory
- Pick and choose from below examples and add mandatory input data and copy data into file `main.tf.example`.
- Rename file __main.tf.example__ to __main.tf__ with: `rename main.tf.example main.tf`
- Initialize with: `terraform init`
- Apply with: `terraform apply -auto-approve` or destroy with: `terraform destroy -auto-approve`

## f5-xc-aws-tgw-snode module usage example

````hcl
````