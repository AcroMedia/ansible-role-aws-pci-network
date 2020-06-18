# ansible-role-aws-pci-network

Create a segregated network in AWS to support hosting a HA PCI compliant web application:
- VPC
- 2 (or more) public subnets (with public IPs)
- 2 (or more) private subnets (without public IPs)
- Internet gateway
- NAT gateway -- *incurs hourly and bandwidth charges in your AWS account*
- Routing for public + private subnets

## Requirements

- Ansible 2.9.1 or higher
- All variable names shown in the example group_vars/all.yml are required.
- If not using aws_billing_tags, specify an empty list `[]`.
- Don't specify a "Name" tag in aws_billing_tags, since names will be auto generated.

## Accessing AWS-generated resource IDs

All tasks in the role create registered variables. Subsequent playbook runs also provide these variables.

Add -v to your ansible command line to have the playbook emit the regstered variable results to your terminal.

Returned data structures are complex, so it's up to the role consuemr to determine what they need, and how to access it. The role itself accesses registerd variables a lot, so there are already plenty of examples built in.

## Required variables

##### aws_resource_tag_slug

  - No spaces or punctuation. This will be used to construct meaningful resource names that may also be used programmatically.

##### aws_resource_env_slug
 - No spaces or punctuation. This will be used to construct meaningful resource names that may also be used programmatically.

##### aws_resource_name_suffix
 - Required, but can be a zero length string. Defaults to 'by Ansible'. Will be added to resource titles and descriptions (but not ids) to indicate the resource was created programmatically. Spaces are ok, but avoid punctuation. You can set this to an empty string `''` if you don't need it.

##### aws_vpc_region
 - The AWS region you want to create your VPC in.

##### aws_vpc_private_net_cidr
 - E.g. `10.0.0.0/16`. This *MUST* have the netmask suffix.

##### aws_vpc_public_subnets
 - A list of subnet and availability zones for your *public* instances.
 - Any EC2 instances created in these subnets will be assigned Public IPv4 addresses.
 - Not all availability zones support all instance types. For example, in Us-east-1, there is one zone that doesn't support "t3" or "m5" instanes, so make sure you test. You may need to destroy and recreate your network, selecting a different zone.
 - Example:
  ```yaml
  aws_vpc_public_subnets:
    - cidr: 10.0.1.0/24
      az: us-east-1a
      app_node_count: 0
    - cidr: 10.0.2.0/24
      az: us-east-1b
      app_node_count: 0
  ```

##### aws_vpc_private_subnets
- A list of subnet and availability zone info your *private* instances.
- Instances created in these subnets will not be assigned public IP addresses. They will only be accessible via bastion host.
- Example:
 ```yaml
  aws_vpc_private_subnets:
    - cidr: 10.0.101.0/24
      az: us-east-1a
      app_node_count: 2
    - cidr: 10.0.102.0/24
      az: us-east-1b
      app_node_count: 2
  ```
**aws_billing_tags**: Arbitrary name value pairs for your own use. If not using, specify an empty list `[]`. All resources created will be tagged with the name/values you create here.

**aws_api_access_key**: The IAM API key that will be used to create resources inside AWS.

**aws_api_secret_key**: The IAM secret key that corresponds to `aws_api_access_key`. Please don't commit this to your playbook without first encrypting the value with Ansible Vault.


## Example playbook

**group_vars/all.yml**:
```yaml
---
aws_resource_tag_slug: bigcorp
aws_resource_env_slug: production
aws_resource_name_suffix: by Ansible
aws_vpc_region: 'us-east-1'
aws_vpc_private_net_cidr: '10.0.0.0/16'
aws_vpc_public_subnets:
  - cidr: 10.0.11.0/24
    az: us-east-1a
  - cidr: 10.0.22.0/24
    az: us-east-1b
aws_vpc_private_subnets:
  - cidr: 10.0.33.0/24
    az: us-east-1a
  - cidr: 10.0.44.0/24
    az: us-east-1b
aws_billing_tags:
  Client: "{{ aws_resource_tag_slug }}"
  Workload: Production
aws_api_access_key: 'YOUR_OWN_API_ACCESS_KEY'
aws_api_secret_key: !vault |
    $ANSIBLE_VAULT;1.1;AES256
    00000000000000000000000000000000000000000000000000000000000000000000000000000000
    00000000000_YOUR_OWN_ANSIBLE_VAULT_ENCRYPTED_API_SECRET_KEY_00000000000000000000
    00000000000000000000000000000000000000000000000000000000000000000000000000000000
    000000000000000000000000000000000000000000000000
```

**requirements.yml**:
```yaml
---
- name: acromedia.aws-pci-network
  src: https://github.com/AcroMedia/ansible-role-aws-pci-network
  version: origin/master

```
**playbook.yml**:
```yaml
---
- name: Configure AWS networking
  hosts: localhost
  connection: local
  become: false
  gather_facts: false
  tags:
    - vpc
  roles:
    - name: Configure AWS network resources
      role: acromedia.aws-pci-network
```
