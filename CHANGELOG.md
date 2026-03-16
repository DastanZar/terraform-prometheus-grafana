# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- `versions.tf` with pinned provider versions (AWS ~>5.0, Terraform >=1.0)
- `terraform.tfvars.example` reference configuration file
- `.github/workflows/terraform-ci.yml` CI pipeline with fmt, validate, and tflint
- `sensitive = true` on `password` variable
- `.gitignore` with Terraform-specific patterns
- Overhauled README with badges, architecture diagram, variables table, and outputs table
- CHANGELOG.md for version tracking

## [1.0.0] - Initial Release

### Added
- Prometheus EC2 module with Let's Encrypt TLS
- Alertmanager EC2 module with Let's Encrypt TLS
- Grafana EC2 module with Let's Encrypt TLS
- S3 bucket for configuration storage
- IAM role with S3, EC2, and CloudWatch read access
- Security groups for HTTP/HTTPS inbound access
- AWS Session Manager support (optional)
- Separate EC2 instances for each monitoring component
