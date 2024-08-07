# Project factory

The Project Factory (or PF) builds on top of your foundations to create and set up projects (and related resources) to be used for your workloads.
It is organized in folders representing environments (e.g., "dev", "prod"), each implemented by a stand-alone terraform [process factory](../../../../blueprints/factories/README.md).

## Design overview and choices

<p align="center">
  <img src="diagram.svg" alt="Project factory diagram">
</p>

A single factory creates projects in a well-defined context, according to your resource management structure. For example, in the diagram above, each Team is structured to have specific folders projects for a given environment, such as Production and Development, per the resource management structure configured in stage `01-resman`.

Projects for each environment across different teams are created by dedicated service accounts, as exemplified in the diagram above. While there's no intrinsic limitation regarding where the project factory can create a projects, the IAM bindings for the service account effectively enforce boundaries (e.g., the production service account shouldn't be able to create or have any access to the development projects, and vice versa).

The project factory stage lightly wraps the underlying [project-factory module](../../../../modules/project-factory/), including Shared VPC service project attachment, VPC SC perimeter membership, etc.
  
## How to run this stage

This stage is meant to be executed after "foundational stages" (i.e., stages [`00-bootstrap`](../../0-bootstrap), [`01-resman`](../../1-resman), 02-networking (either [Peering/VPN](../../2-networking-a-simple), [NVA (w/ optional BGP support)](../../2-networking-b-nva) and [`02-security`](../../2-security)) have been run.

It's of course possible to run this stage in isolation, by making sure the architectural prerequisites are satisfied (e.g., networking), and that the Service Account running the stage is granted the appropriate roles.

### Provider and Terraform variables

As all other FAST stages, the [mechanism used to pass variable values and pre-built provider files from one stage to the next](../../0-bootstrap/README.md#output-files-and-cross-stage-variables) is also leveraged here.

The commands to link or copy the provider and terraform variable files can be easily derived from the `stage-links.sh` script in the FAST root folder, passing it a single argument with the local output files folder (if configured) or the GCS output bucket in the automation project (derived from stage 0 outputs). The following examples demonstrate both cases, and the resulting commands that then need to be copy/pasted and run.

```bash
../../../stage-links.sh ~/fast-config

# copy and paste the following commands for '3-project-factory'

ln -s ~/fast-config/providers/3-project-factory-providers.tf ./
ln -s ~/fast-config/tfvars/0-globals.auto.tfvars.json ./
ln -s ~/fast-config/tfvars/0-bootstrap.auto.tfvars.json ./
ln -s ~/fast-config/tfvars/1-resman.auto.tfvars.json ./
ln -s ~/fast-config/tfvars/2-networking.auto.tfvars.json ./
ln -s ~/fast-config/tfvars/2-security.auto.tfvars.json ./
```

```bash
../../../stage-links.sh gs://xxx-prod-iac-core-outputs-0

# copy and paste the following commands for '3-project-factory'

gcloud storage cp gs://xxx-prod-iac-core-outputs-0/providers/3-project-factory-providers.tf ./
gcloud storage cp gs://xxx-prod-iac-core-outputs-0/tfvars/0-globals.auto.tfvars.json ./
gcloud storage cp gs://xxx-prod-iac-core-outputs-0/tfvars/0-bootstrap.auto.tfvars.json ./
gcloud storage cp gs://xxx-prod-iac-core-outputs-0/tfvars/1-resman.auto.tfvars.json ./
gcloud storage cp gs://xxx-prod-iac-core-outputs-0/tfvars/2-networking.auto.tfvars.json ./
gcloud storage cp gs://xxx-prod-iac-core-outputs-0/tfvars/2-security.auto.tfvars.json ./
```

If you're not using FAST, refer to the [Variables](#variables) table at the bottom of this document for a full list of variables, their origin (e.g., a stage or specific to this one), and descriptions explaining their meaning.

Besides the values above, the project factory is driven by YAML data files, with one file per project. Please refer to the underlying [project factory module](../../../../modules/project-factory/) documentation for details on the format.

Once the configuration is complete, run the project factory with:

```bash
terraform init
terraform apply
```

<!-- TFDOC OPTS files:1 show_extra:1 -->
<!-- BEGIN TFDOC -->
## Files

| name | description | modules |
|---|---|---|
| [main.tf](./main.tf) | Project factory. | <code>project-factory</code> |
| [outputs.tf](./outputs.tf) | Module outputs. |  |
| [variables-fast.tf](./variables-fast.tf) | None |  |
| [variables.tf](./variables.tf) | Module variables. |  |

## Variables

| name | description | type | required | default | producer |
|---|---|:---:|:---:|:---:|:---:|
| [billing_account](variables-fast.tf#L17) | Billing account id. If billing account is not part of the same org set `is_org_level` to false. | <code title="object&#40;&#123;&#10;  id           &#61; string&#10;  is_org_level &#61; optional&#40;bool, true&#41;&#10;&#125;&#41;">object&#40;&#123;&#8230;&#125;&#41;</code> | ✓ |  | <code>0-bootstrap</code> |
| [factories_config](variables.tf#L17) | Path to folder with YAML resource description data files. | <code title="object&#40;&#123;&#10;  hierarchy &#61; optional&#40;object&#40;&#123;&#10;    folders_data_path &#61; string&#10;    parent_ids        &#61; optional&#40;map&#40;string&#41;, &#123;&#125;&#41;&#10;  &#125;&#41;&#41;&#10;  projects_data_path &#61; string&#10;  budgets &#61; optional&#40;object&#40;&#123;&#10;    billing_account       &#61; string&#10;    budgets_data_path     &#61; string&#10;    notification_channels &#61; optional&#40;map&#40;any&#41;, &#123;&#125;&#41;&#10;  &#125;&#41;&#41;&#10;&#125;&#41;">object&#40;&#123;&#8230;&#125;&#41;</code> | ✓ |  |  |
| [prefix](variables-fast.tf#L30) | Prefix used for resources that need unique names. Use a maximum of 9 chars for organizations, and 11 chars for tenants. | <code>string</code> | ✓ |  | <code>0-bootstrap</code> |

## Outputs

| name | description | sensitive | consumers |
|---|---|:---:|---|
| [projects](outputs.tf#L17) | Created projects. |  |  |
| [service_accounts](outputs.tf#L27) | Created service accounts. |  |  |
<!-- END TFDOC -->
