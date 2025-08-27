# Terraform Module & Root Configuration Best Practices

## Table of Contents
- [Naming & Structure](#naming--structure)
- [Variables](#variables)
- [Outputs](#outputs)
- [Data Sources](#data-sources)
- [Scripts & Static Files](#scripts--static-files)
- [Formatting & Expression Complexity](#formatting--expression-complexity)
- [Root Modules & Environments](#root-modules--environments)
- [Provider & Dependency Management](#provider--dependency-management)

---

## Naming & Structure

### Adopt a consistent naming convention
- Use underscores (`_`) to delimit multiple words in **configuration object names**.  
  *This does not apply to `name` arguments.*
- Prefer `main` for a resource that is the only one of its type in a module.
- Resource names should be **singular** and **not repeat** the resource type.

**Examples**

✅ Recommended
```hcl
resource "vsphere_virtual_machine" "web_server" {
  name = "web-server"
}
```

❌ Not recommended
```hcl
resource "vsphere-virtual-machine" "web-server" {
  name = "web-server"
}
```

✅ Recommended
```hcl
resource "vsphere_virtual_machine" "main" { ... }
```
❌ Not recommended
```hcl
resource "vsphere_virtual_machine" "main_vsphere_virtual_machine" { ... }
```

---

## Variables

- Declare all variables in `variables.tf` with **descriptions** and **types**.
- Use **descriptive** names relevant to the purpose.
- Numeric inputs (disk sizes, RAM, etc.) must include units in the name (e.g., `ram_size_gb`).  
  - Storage units → binary prefixes: **kibi**, **mebi**, **gibi** (powers of 1024).  
  - Other units → decimal prefixes: **kilo**, **mega**, **giga** (powers of 1000).
- Boolean variables: prefer **positive** names (e.g., `enable_external_access`).
- Provide **default values** only when environment-independent (e.g., disk size).  
  No defaults for environment-specific values (e.g., `project_id`).
- Use empty defaults (empty strings/lists/maps) **only** if an empty value is valid for the API.
- Be judicious—expose only variables that must vary. Prefer **locals** for repeated literals.

---

## Outputs

- Organize outputs in `outputs.tf` with **meaningful descriptions**.
- Document output meanings in `README.md` (auto-generate with tools like `terraform-docs`).
- Expose **all useful values** that callers may need to reference or share.
- Ensure outputs reference **resources** (not input variables) to create correct dependencies.

**Example**
```hcl
# ✅ Recommended
output "name" {
  description = "Name of VM"
  value       = vsphere_virtual_machine.main.name
}

# ❌ Not recommended
output "name" {
  description = "Name of VM"
  value       = var.name
}
```

> For shared modules, include **at least one output per resource** so callers can infer dependencies and ordering.

---

## Data Sources

- Place data sources **near** the resources that consume them.  
- If they become numerous, move to a dedicated `data.tf`.
- Fetch environment-relative data using variables or resource interpolation.

---

## Scripts & Static Files

- **Limit custom scripts**—use only when Terraform resources don’t support needed behavior.  
  - Scripts invoked by Terraform → put under `scripts/`.  
  - Helper scripts not called by Terraform → put under `helpers/` and document usage (`--help`, arguments, examples).
- **Static files** (e.g., startup scripts) go under `files/`.  
- Place long HereDocs in external files and reference with `file()`.  
- Templates read with `templatefile()` should use the `.tftpl` extension in a `templates/` directory.

---

## Formatting & Expression Complexity

- Run `terraform fmt`—all Terraform files must conform to it.
- Keep interpolations simple; split complex logic into **locals**.
- Avoid more than one ternary per line; build up logic via multiple locals.


### Use `count` for conditional creation
```hcl
variable "readers" {
  description = "..."
  type        = list(string)
  default     = []
}

resource "resource_type" "reference_name" {
  # Do not create this resource if the list is empty
  count = length(var.readers) == 0 ? 0 : 1
  ...
}
```
- Be careful using user-specified variables to compute `count` when dependent resources don’t exist—Terraform can’t plan.  
  Prefer separate `enable_x` booleans for gating logic.

### Use `for_each` for iterated resources
- Prefer `for_each` when you need to create many resources keyed by meaningful IDs.


### Expose labels as a variable
Allow flexible resource labeling via the module interface. Provide a `labels` variable with an empty map default:

```hcl
variable "labels" {
  description = "A map of labels to apply to contained resources."
  type        = map(string)  # or map(any) for mixed types
  default     = {}
}
```

### Inline Submodules

- Use **inline submodules** to organize complex logic into smaller units and deduplicate common resources.  
  Place them under `modules/<submodule-name>` **within** the shared module.
- Treat inline submodules as **private** implementation details unless explicitly documented for external use.
- **Refactoring caution:** Terraform doesn’t auto-track refactored resources; moving resources into submodules can trigger recreation.  
  - Mitigate with `moved` blocks to preserve resource history and state.
- Outputs from internal modules are **not** automatically exposed—**re-export** what callers need at the parent module level.

---

## Root Modules & Environments

### Minimize the number of resources per root module
- Keep a single root configuration from growing too large.  
- All resources in a root module are refreshed on each `terraform plan/apply`.  
- **Rule of thumb:** *don’t exceed 100 resources per state* (ideally only a few dozen).

### Use separate directories for each application/service
- Manage applications/projects independently in **dedicated directories**.  
- A “service” may be an app or shared infra (e.g., networking).  
- Nest all code for a service under one directory (with subdirectories as needed).

### Split applications into environment-specific subdirectories
Use a two-tier layout: **modules** (implementation) and **environments** (roots):

```
SERVICE-DIRECTORY/
 ├── OWNERS
 ├── modules/
 │    └── <service-name>/
 │         ├── main.tf
 │         ├── variables.tf
 │         ├── outputs.tf
 │         ├── provider.tf
 │         └── README
 │    └── ...other modules…
 └── environments/
      ├── dev/
      │    ├── backend.tf
      │    └── main.tf
      ├── qa/
      │    ├── backend.tf
      │    └── main.tf
      └── prod/
           ├── backend.tf
           └── main.tf
```

### Use environment directories (and default workspace only)
- Share code across environments by **referencing modules**.  
- In service modules, hard-code common inputs; require only env-specific inputs as variables.
- Each environment directory **must contain**:
  - `backend.tf` → declares backend state (e.g., Cloud Storage)
  - `main.tf` → instantiates the service module
- Each environment directory corresponds to the **default workspace** for that env.  
  Avoid multiple CLI workspaces because:
  - Harder to inspect configuration per workspace
  - Shared backend becomes a single point of failure
  - Code readability suffers with `terraform.workspace` conditionals

---

## Provider & Dependency Management

### Pin to minor provider versions
- In root modules, declare each provider and pin to a **minor version** using `~>`.  
  This allows patch upgrades automatically while keeping a stable target.
- Store provider constraints in `versions.tf`.

```hcl
terraform {
  required_providers {
    vsphere = {
      source = "hashicorp/vsphere"
      version = "2.12.0"
    }
  }
}
```

### Use tfvars for runtime inputs
- Provide variables via a default `terraform.tfvars`.  
- Avoid `-var` flags or ad-hoc var-files—these are ephemeral and error-prone.

### Commit the dependency lock file
- Check in `.terraform.lock.hcl` to source control to track provider selections and changes.

