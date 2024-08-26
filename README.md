# Google KMS Module

This module allows creating and managing KMS crypto keys and IAM bindings at both the keyring and crypto key level. An existing keyring can be used, or a new one can be created and managed by the module if needed.

When using an existing keyring be mindful about applying IAM bindings, as all bindings used by this module are authoritative, and you might inadvertently override bindings managed by the keyring creator.

## Examples

### Using an existing keyring

```hcl
module "kms" {
  source         = "github.com/dapperlabs-platform/terraform-google-kms?ref=tag"
  project_id     = "my-project"
  iam    = {
    "roles/owner" = ["user:user1@example.com"]
  }
  keyring        = { location = "europe-west1", name = "test" }
  keyring_create = false
  keys           = { key-a = null, key-b = null, key-c = null }
}
# tftest:skip
```

### Keyring creation and crypto key rotation and IAM roles

```hcl
module "kms" {
  source     = "../modules/kms"
  project_id = "my-project"
  key_iam = {
    key-a = {
      "roles/owner" = ["user:user1@example.com"]
    }
  }
  keyring = { location = "europe-west1", name = "test" }
  keys = {
    key-a = null
    key-b = { rotation_period = "604800s", labels = null }
    key-c = { rotation_period = null, labels = { env = "test" } }
  }
}
# tftest:modules=1:resources=5
```

### Crypto key purpose

```hcl
module "kms" {
  source      = "../modules/kms"
  project_id  = "my-project"
  key_purpose = {
    key-c = {
      purpose = "ASYMMETRIC_SIGN"
      version_template = {
        algorithm        = "EC_SIGN_P384_SHA384"
        protection_level = null
      }
    }
  }
  keyring     = { location = "europe-west1", name = "test" }
  keys        = { key-a = null, key-b = null, key-c = null }
}
# tftest:modules=1:resources=4
```

<!-- BEGIN_TF_DOCS -->
Copyright 2021 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.0.0 |
| <a name="requirement_google"></a> [google](#requirement\_google) | ~> 5.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_google"></a> [google](#provider\_google) | ~> 5.0 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [google_kms_crypto_key.default](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/kms_crypto_key) | resource |
| [google_kms_crypto_key_iam_binding.default](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/kms_crypto_key_iam_binding) | resource |
| [google_kms_key_ring.default](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/kms_key_ring) | resource |
| [google_kms_key_ring_iam_binding.default](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/kms_key_ring_iam_binding) | resource |
| [google_kms_key_ring.default](https://registry.terraform.io/providers/hashicorp/google/latest/docs/data-sources/kms_key_ring) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_iam"></a> [iam](#input\_iam) | Keyring IAM bindings for topic in {ROLE => [MEMBERS]} format. | `map(list(string))` | `{}` | no |
| <a name="input_key_iam"></a> [key\_iam](#input\_key\_iam) | Key IAM bindings for topic in {KEY => {ROLE => [MEMBERS]}} format. | `map(map(list(string)))` | `{}` | no |
| <a name="input_key_purpose"></a> [key\_purpose](#input\_key\_purpose) | Per-key purpose, if not set defaults will be used. If purpose is not `ENCRYPT_DECRYPT` (the default), `version_template.algorithm` is required. | <pre>map(object({<br>    purpose = string<br>    version_template = object({<br>      algorithm        = string<br>      protection_level = string<br>    })<br>  }))</pre> | `{}` | no |
| <a name="input_key_purpose_defaults"></a> [key\_purpose\_defaults](#input\_key\_purpose\_defaults) | Defaults used for key purpose when not defined at the key level. If purpose is not `ENCRYPT_DECRYPT` (the default), `version_template.algorithm` is required. | <pre>object({<br>    purpose = string<br>    version_template = object({<br>      algorithm        = string<br>      protection_level = string<br>    })<br>  })</pre> | <pre>{<br>  "purpose": null,<br>  "version_template": null<br>}</pre> | no |
| <a name="input_keyring"></a> [keyring](#input\_keyring) | Keyring attributes. | <pre>object({<br>    location = string<br>    name     = string<br>  })</pre> | n/a | yes |
| <a name="input_keyring_create"></a> [keyring\_create](#input\_keyring\_create) | Set to false to manage keys and IAM bindings in an existing keyring. | `bool` | `true` | no |
| <a name="input_keys"></a> [keys](#input\_keys) | Key names and base attributes. Set attributes to null if not needed. | <pre>map(object({<br>    rotation_period            = string<br>    labels                     = map(string)<br>    destroy_scheduled_duration = string<br>  }))</pre> | `{}` | no |
| <a name="input_project_id"></a> [project\_id](#input\_project\_id) | Project id where the keyring will be created. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_key_self_links"></a> [key\_self\_links](#output\_key\_self\_links) | Key self links. |
| <a name="output_keyring"></a> [keyring](#output\_keyring) | Keyring resource. |
| <a name="output_keys"></a> [keys](#output\_keys) | Key resources. |
| <a name="output_location"></a> [location](#output\_location) | Keyring location. |
| <a name="output_name"></a> [name](#output\_name) | Keyring name. |
| <a name="output_self_link"></a> [self\_link](#output\_self\_link) | Keyring self link. |
<!-- END_TF_DOCS -->