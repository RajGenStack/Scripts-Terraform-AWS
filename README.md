# Terraform: EC2 Instance with a CI Toolchain

A single-file Terraform configuration that launches an Ubuntu EC2 instance in the default VPC and installs a CI toolchain at first boot: Jenkins, Docker, Maven, Git, Nginx and pip.

![Terraform](https://img.shields.io/badge/Terraform-844FBA?style=flat-square&logo=terraform&logoColor=white)
![Amazon EC2](https://img.shields.io/badge/Amazon_EC2-FF9900?style=flat-square&logo=amazonec2&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat-square&logo=jenkins&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

## What it creates

| Resource | Configuration |
|---|---|
| Security group `allow_ssh` | In the default VPC; inbound SSH (22) from anywhere; all outbound traffic allowed |
| EC2 instance | `t2.medium` with a public IP, AMI `ami-0360c520857e3138f` (us-east-1), key pair `sarah-key-acc` |
| User data | Upgrades packages; installs `git`, `maven`, `docker.io`, `nginx` and `python3-pip`; enables Docker and Nginx; adds the Jenkins apt repository and installs Jenkins |

## Usage

1. Set `key_name` in `aws-ec2-instance.tf` to an EC2 key pair you own in us-east-1. The AMI ID is region-specific, so look up the current Ubuntu AMI if you change region.
2. Apply:

   ```bash
   terraform init
   terraform apply
   ```

3. Connect, using the public IP from `terraform show` or the EC2 console:

   ```bash
   ssh -i <your-key>.pem ubuntu@<public-ip>
   ```

4. Installation continues in the background after first boot. Follow it with `sudo tail -f /var/log/cloud-init-output.log`, then read Jenkins' initial admin password:

   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

Remove everything with `terraform destroy`.

## Reaching Jenkins and Nginx

The security group only opens port 22, so Jenkins (8080) and Nginx (80) are not reachable from the internet. Either add ingress rules for those ports restricted to your own IP, or use an SSH tunnel:

```bash
ssh -i <your-key>.pem -L 8080:localhost:8080 ubuntu@<public-ip>
# then open http://localhost:8080
```

## Security note

SSH is open to `0.0.0.0/0`. Replace it with your own address (`<your-ip>/32`) before leaving the instance running.

---

<div align="center">
  <sub>Maintained by <a href="https://github.com/RajGenStack">Rajan Kumar</a> · <a href="https://www.linkedin.com/in/rajan-kumar42">LinkedIn</a></sub>
</div>
