# ansible-role-aws-pci-network

Create a segregated network in AWS to support hosting a HA PCI compliant web application:
- VPC
- 2 (or more) public subnets (with public IPs)
- 2 (or more) private subnets (without public IPs)
- Internet gateway
- NAT gateway -- *incurs hourly and bandwidth charges in your AWS account*
- Routing for public + private subnets

## Requirements

- Ansible 2.9 or higher
- All variable names shown in the example group_vars/all.yml are required.
- If not using aws_billing_tags, specify an empty list `[]`.
- Don't specify a "Name" tag in aws_billing_tags, since names will be auto generated.

## Accessing AWS-generated resource IDs

All tasks in the role create registered variables. Subsequent playbook runs also provide these variables.

Add -v to your ansible command line to have the playbook emit the regstered variable results to your terminal.

Returned data structures are complex, so it's up to the role consuemr to determine what they need, and how to access it. The role itself accesses registerd variables a lot, so there are already plenty of examples built in.

## Example playbook

**group_vars/all.yml**:
```yaml
---
aws_resource_tag_slug: BigCorp    # No spaces or punctuation. This slug will be used to construct meaningful 'Name' tag values.
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
