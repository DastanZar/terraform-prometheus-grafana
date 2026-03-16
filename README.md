# terraform-prometheus-grafana

[![Terraform CI](https://github.com/DastanZar/terraform-prometheus-grafana/actions/workflows/terraform-ci.yml/badge.svg)](https://github.com/DastanZar/terraform-prometheus-grafana/actions/workflows/terraform-ci.yml)
[![Terraform](https://img.shields.io/badge/Terraform-%E2%89%A5%201.0-7B42BC?logo=terraform)](https://www.terraform.io/)
[![AWS Provider](https://img.shields.io/badge/AWS%20Provider-~%3E%205.0-FF9900?logo=amazon-aws)](https://registry.terraform.io/providers/hashicorp/aws)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Production-grade Terraform module that provisions a full observability stack — **Prometheus**, **Alertmanager**, and **Grafana** — on AWS EC2 with S3-backed configuration management and Let's Encrypt TLS.

## Architecture

```
 Internet
    |
    v
[Security Group: HTTP/HTTPS]
    |
    +------------------+------------------+
    |                  |                  |
    v                  v                  v
[Prometheus EC2]  [Alertmanager EC2]  [Grafana EC2]
  t2.nano           t2.nano             t2.nano
  port 9090         port 9093           port 3000
    |                  |                  |
    +------------------+------------------+
                       |
                       v
              [S3 Config Bucket]
              prometheus.yml
              alertmanager.yml
              grafana.ini
                       |
               [IAM Role]
            S3 Read | EC2 Read
            CloudWatch Read
```

## Features

- **Prometheus** — metrics scraping and alerting rules
- **Alertmanager** — alert routing, deduplication, and notification
- **Grafana** — dashboards and data visualisation
- **S3-backed config** — sync configuration without re-provisioning
- **HTTPS** — automatic TLS via Let's Encrypt (certbot)
- **IAM least-privilege** — scoped S3/EC2/CloudWatch read access
- **Optional SSM** — AWS Session Manager for SSH-free access
- **GitHub Actions CI** — `terraform fmt`, `validate`, and `tflint` on every PR

## Requirements

| Tool | Version |
|------|---------|
| Terraform | >= 1.0 |
| AWS Provider | ~> 5.0 |
| AWS CLI | any |

## Usage

```hcl
provider "aws" {
  region = "us-east-1"
}

module "monitoring" {
  source = "github.com/DastanZar/terraform-prometheus-grafana"

  hostname_prometheus   = "prometheus.example.com"
  hostname_alertmanager = "alertmanager.example.com"
  hostname_grafana      = "grafana.example.com"

  config_bucket_name  = "my-monitoring-config"
  password            = var.monitoring_password
  letsencrypt_email   = "admin@example.com"
}
```

Initialise and apply:

```bash
terraform init
terraform plan -var-file=terraform.tfvars
terraform apply
```

## Configuration (S3)

Sync your config to S3 before or after provisioning:

```
config/
  alertmanager/
    alertmanager.yml
  grafana/
    grafana.ini
    provisioning/datasources/datasource.yml
  prometheus/
    prometheus.yml
    rules.yml
```

```bash
aws s3 sync config/ s3://my-monitoring-config/
```

## Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `hostname_prometheus` | Prometheus FQDN | — | Yes |
| `hostname_alertmanager` | Alertmanager FQDN | — | Yes |
| `hostname_grafana` | Grafana FQDN | — | Yes |
| `config_bucket_name` | S3 bucket name | — | Yes |
| `password` | Web UI password (sensitive) | — | Yes |
| `letsencrypt_email` | Let's Encrypt email | — | Yes |
| `instance_type_prometheus` | EC2 instance type | `t2.nano` | No |
| `instance_type_alertmanager` | EC2 instance type | `t2.nano` | No |
| `instance_type_grafana` | EC2 instance type | `t2.nano` | No |
| `key_name` | EC2 key pair name | `""` | No |
| `allow_session_manager` | Enable SSM access | `false` | No |

## Outputs

| Output | Description |
|--------|-------------|
| `public_ip_prometheus` | Prometheus public IP |
| `public_ip_alertmanager` | Alertmanager public IP |
| `public_ip_grafana` | Grafana public IP |
| `private_ip_prometheus` | Prometheus private IP |
| `private_ip_alertmanager` | Alertmanager private IP |
| `private_ip_grafana` | Grafana private IP |

## Security Notes

- `password` is marked `sensitive = true` — never appears in plan output
- IAM roles grant only the minimum required permissions
- Security groups restrict inbound to HTTP/HTTPS; internal traffic uses port 3000
- `.tfvars` files are gitignored by default

## Contributing

PRs welcome. Please run `terraform fmt` and ensure `terraform validate` passes before opening a pull request.

## License

MIT
