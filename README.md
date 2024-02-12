# terraform-google-secure-web-proxy

## Description

### Detailed
This module was generated from [terraform-google-module-template](https://github.com/terraform-google-modules/terraform-google-module-template/), which by default generates a module that simply creates a GCS bucket. As the module develops, this README should be updated.

The resources/services/activations/deletions that this module will create/trigger are:

- Create a GCS bucket with the provided name

### PreDeploy
- Google cloud VPC
- Subnet in the region 
- Proxy only subnet
- Regional self-magaged certificate in the region

## Usage

Basic usage of this module is as follows:

```hcl
module "secure_web_proxy" {
  gateway_name     = "simple-swp"
  project_id       = var.project_id
  region           = var.region
  certificate_urls = [google_certificate_manager_certificate.this.id]
  network          = google_compute_network.this.id
  subnetwork       = google_compute_subnetwork.resource_subnet.id

  policy = {
    name        = "simple-proxy-policy"
    description = "Policy for secure web proxy"
  }

  rules = {
    "allow-example1-com" = {
      enabled         = true
      description     = "Allow example1.com host traffic."
      priority        = 100
      session_matcher = "host() == 'example1.com'"
      basic_profile   = "ALLOW"
    },
    "allow-url-list-1" = {
      enabled         = true
      description     = "All the URLs in URL list test-url-list-1."
      priority        = 102
      session_matcher = "inUrlList(host(), 'projects/${var.project_id}/locations/${var.region}/urlLists/test-url-list-1')"
      basic_profile   = "ALLOW"
    },
  }

  url_lists = {
    "test-url-list-1" = {
      description = "url-list-1 description."
      values      = ["www.example.com", "about.example.com", "github.com/example-org/*"]
    }
  }
}
```

Functional examples are included in the
[examples](./examples/) directory.

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| certificate\_urls | A fully-qualified certificates URL reference. The proxy presents a Certificate (selected based on SNI) when establishing a TLS connection. | `list(string)` | n/a | yes |
| delete\_swg\_autogen\_router\_on\_destroy | boolean option to also delete auto generated router by the gateway creation. | `bool` | `true` | no |
| gateway\_name | The name of secure web proxy gateway to be created. | `string` | n/a | yes |
| ip\_address | Static IP reservation for SWP. When no address is provided, an IP from the input subnetwork is allocated. | `string` | `""` | no |
| labels | Map of labels for secure web proxy gateway. | `map(string)` | `{}` | no |
| network | URI of the subnetwork for which this secure web proxy will be created. | `string` | n/a | yes |
| policy | Gateway security policy configuration. | <pre>object({<br>    name        = string<br>    description = string<br>    tls_inspection_policy = optional(object({<br>      name    = string<br>      ca_pool = string<br>    }))<br>  })</pre> | n/a | yes |
| project\_id | The Google Cloud project ID where the secure web proxy will be deployed. | `string` | n/a | yes |
| region | The region in which the secure web proxy components will be created. | `string` | n/a | yes |
| rules | Security policy rules configuration. | <pre>map(object({<br>    enabled             = optional(bool, true)<br>    description         = optional(string, "SWP rules created by terraform")<br>    priority            = number                                                # Lower number corresponds to higher precedence.<br>    session_matcher     = optional(string, "inIpRange(source.ip, '0.0.0.0/0')") # By default, open all source ips.<br>    application_matcher = optional(string)<br>    basic_profile       = optional(string, "ALLOW") # Supports ALLOW or DENY.string<br>  }))</pre> | `null` | no |
| scope | Scope determines how configuration across multiple gateway instances are merged. The configuration for multiple gateway instances with the same scope will be merged as presented as a single coniguration to the proxy. Defaults to name of the region. Max length - 64 characters. | `string` | `""` | no |
| subnetwork | URI of the subnetwork for which this secure web proxy will be created. | `string` | n/a | yes |
| url\_lists | URL lists that can be used within SWP rules. Attribute values supports: FQDNs and URLs. | <pre>map(object({<br>    description = optional(string, "URL lists created by terraform")<br>    values      = list(string)<br>  }))</pre> | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| gateway\_id | Identifier for the secure web proxy gateway. |
| policy\_id | Identifier of the secure web proxy gateway policy. |
| rule\_ids | Identifiers of the secure web proxy rules created. |
| url\_list\_ids | Identifiers of the secure web proxy url lists. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Requirements

These sections describe requirements for using this module.

### Software

The following dependencies must be available:

- [Terraform][terraform] >= v1.3.0
- [Terraform Provider for GCP][terraform-provider-gcp] plugin >= v5.1.0

### Service Account

A service account with the following roles must be used to provision
the resources of this module:

- Storage Admin: `roles/storage.admin`

The [Project Factory module][project-factory-module] and the
[IAM module][iam-module] may be used in combination to provision a
service account with the necessary roles applied.

### APIs

A project with the following APIs enabled must be used to host the
resources of this module:

- Google Cloud Storage JSON API: `storage-api.googleapis.com`

The [Project Factory module][project-factory-module] can be used to
provision a project with the necessary APIs enabled.

## Contributing

Refer to the [contribution guidelines](./CONTRIBUTING.md) for
information on contributing to this module.

[iam-module]: https://registry.terraform.io/modules/terraform-google-modules/iam/google
[project-factory-module]: https://registry.terraform.io/modules/terraform-google-modules/project-factory/google
[terraform-provider-gcp]: https://www.terraform.io/docs/providers/google/index.html
[terraform]: https://www.terraform.io/downloads.html

## Security Disclosures

Please see our [security disclosure process](./SECURITY.md).
