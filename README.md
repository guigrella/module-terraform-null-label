# module-terraform-null-label
Terraform module designed to generate consistent names and tags for resources. Use `terraform-null-label` to implement a strict naming convention.

A label follows the following convention: `{environment}-{name}-{squad}-{attributes}`. The delimiter (e.g. `-`) is configurable.
* squad

Other labels are also required:
* bu (As default value we use `{zedelivery}`)
* costcenter
* tribe

You can add any aditional tags if you want to

**NOTE:** Module forked from Cloudposse [terraform-null-label](https://github.com/cloudposse/terraform-null-label)
The `null` refers to the primary Terraform [provider](https://www.terraform.io/docs/providers/null/index.html) used in this module.

Releases of this module from `0.12.0` onward support `HCL2` and only work with Terraform 0.12 or newer.


---

## Usage
**IMPORTANT:** The `master` branch is used in `source` just as an example. In your code, do not pin to `master` because there may be breaking changes between releases.
Instead pin to the release tag (e.g. `?ref=tags/x.y.z`) of one of our latest releases


### Defaults

Cloud Posse Terraform modules share a common `context` object that is meant to be passed from module to module.
The context object is a single object that contains all the input values for `terraform-null-label`.
However, each input value can also be specified individually by name as a standard Terraform variable,
and the value of those variables, when set to something other than `null`, will override the value
in the context object. In order to allow chaining of these objects, where the context object input to one
module is transformed and passed to the next module, all the variables default to `null` or empty collections.
The actual default values used when nothing is explicitly set are describe in the documentation below.

For example, the default value of `delimiter` is shown as `null`, but if you leave it set to null,
`terraform-null-label` will actually use the default delimiter `-` (hyphen).

A non-obvious but intentional consequence of this design is that once a module sets a non-default value,
future modules in the chain cannot reset the value back to the original default. Insted, the new setting
becomes the new default for downstream modules. Also, collections are not overwritten, they are merged,
so once a tag is added, it will remain in the tag set and cannot be removed, although its value can
be overwritten.

### Simple Example

```hcl
module "lab_bastion_infracore_label" {
  source      = "git::https://github.com/cloudposse/terraform-null-label.git?ref=master"
  environment = "lab"
  name        = "bastion"
  squad       = "infracore"
  bu          = "zedelivery"
  costcenter  = "142"
  tribe       = "infracloud"
  terraform   = "true"
  tags = {
    "foo" = "bar"
  }
}
```

This will create an `id` with the value of `lab-bastion-infracore` because when generating `id`, the default order is `environment`, `name`, `squad`

Now reference the label when creating an instance:

```hcl
resource "aws_instance" "lab_bastion_infracore" {
  instance_type = "t1.micro"
  tags          = module.lab_bastion_infracore_.tags
}
```

Or define a security group:

```hcl
resource "aws_security_group" "lab_bastion_infracore" {
  vpc_id = var.vpc_id
  name   = module.lab_bastion_infracore.id
  tags   = module.lab_bastion_infracore.tags
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.12.0, < 0.14.0 |

## Providers

No provider.

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| additional\_tag\_map | Additional tags for appending to tags\_as\_list\_of\_maps. Not added to `tags`. | `map(string)` | `{}` | no |
| attributes | Additional attributes (e.g. `1`) | `list(string)` | `[]` | no |
| context | Single object for setting entire context at once.<br>See description of individual variables for details.<br>Leave string and numeric variables as `null` to use default value.<br>Individual variable settings (non-null) override settings in context object,<br>except for attributes, tags, and additional\_tag\_map, which are merged. | <pre>object({<br>    enabled             = bool<br>    namespace           = string<br>    environment         = string<br>    stage               = string<br>    name                = string<br>    delimiter           = string<br>    attributes          = list(string)<br>    tags                = map(string)<br>    additional_tag_map  = map(string)<br>    regex_replace_chars = string<br>    label_order         = list(string)<br>    id_length_limit     = number<br>  })</pre> | <pre>{<br>  "additional_tag_map": {},<br>  "attributes": [],<br>  "delimiter": null,<br>  "enabled": true,<br>  "environment": null,<br>  "id_length_limit": null,<br>  "label_order": [],<br>  "name": null,<br>  "namespace": null,<br>  "regex_replace_chars": null,<br>  "stage": null,<br>  "tags": {}<br>}</pre> | no |
| delimiter | Delimiter to be used between `namespace`, `environment`, `stage`, `name` and `attributes`.<br>Defaults to `-` (hyphen). Set to `""` to use no delimiter at all. | `string` | `null` | no |
| enabled | Set to false to prevent the module from creating any resources | `bool` | `null` | no |
| environment | Environment, e.g. 'lab', 'dev', 'hom', OR 'prod' | `string` | `null` | yes |
| id\_length\_limit | Limit `id` to this many characters.<br>Set to `0` for unlimited length.<br>Set to `null` for default, which is `0`.<br>Does not affect `id_full`. | `number` | `null` | no |
| label\_order | The naming order of the id output and Name tag.<br>Defaults to ["namespace", "environment", "stage", "name", "attributes"].<br>You can omit any of the 5 elements, but at least one must be present. | `list(string)` | `null` | no |
| name | Solution name, e.g. 'app' or 'jenkins' | `string` | `null` | yes |
| regex\_replace\_chars | Regex to replace chars with empty string in `namespace`, `environment`, `stage` and `name`.<br>If not set, `"/[^a-zA-Z0-9-]/"` is used to remove all characters other than hyphens, letters and digits. | `string` | `null` | no |
| squad | Squad, e.g. 'infracore', 'p2p', 'card', for more check squad list | `string` | `null` | yes |
| bu | bu, e.g. The default value is 'zedelivery' | `string` | `zedelivery` | no |
| costcenter | costcenter, A number for the cost center, check cost center list | `string` | `null` | yes |
| tribe | tribe, A tribe name, check tribe name list list | `string` | `null` | yes |
| terraform | to know if the resource was created with terraform | `string` | `true` | no |
| tags | Additional tags (e.g. `map('BusinessUnit','XYZ')` | `map(string)` | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| additional\_tag\_map | The merged additional\_tag\_map |
| attributes | List of attributes |
| context | Merged but otherwise unmodified input to this module, to be used as context input to other modules.<br>Note: this version will have null values as defaults, not the values actually used as defaults. |
| delimiter | Delimiter between `namespace`, `environment`, `stage`, `name` and `attributes` |
| enabled | True if module is enabled, false otherwise |
| environment | Normalized environment |
| id | Disambiguated ID restricted to `id_length_limit` characters in total |
| id\_full | Disambiguated ID not restricted in length |
| id\_length\_limit | The id\_length\_limit actually used to create the ID, with `0` meaning unlimited |
| label\_order | The naming order actually used to create the ID |
| name | Normalized name |
| normalized\_context | Normalized context of this module |
| regex\_replace\_chars | The regex\_replace\_chars actually used to create the ID |
| squad | Normalized stage |
| tags | Normalized Tag map |
| bu | Normalized bu |
| costcenter | Normalized costcenter |
| tribe | Normalized tribe |
| terraform | Normalized terraform |
| tags\_as\_list\_of\_maps | Additional tags as a list of maps, which can be used in several AWS resources |

