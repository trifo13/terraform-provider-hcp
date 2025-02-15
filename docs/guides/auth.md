---
subcategory: ""
page_title: "Authenticate with HCP - HCP Provider"
description: |-
    A guide to obtaining HCP credentials and adding them to provider configuration.
---

# Authenticate with HCP

The HCP provider accepts two forms of authentication:

- client credentials, obtained on the creation of a service principal key
- user session, obtained via browser login (as of `v0.45.0`)

Only one form is needed.

## Client credentials

Client credentials are recommended for CI and local development with the SDK or any tool consuming it.

The `client_id` and `client_secret` must come from a service principal key. Service principals and service principal keys can be created in the HCP portal with an existing user account. The service principal must be authorized to access the API. Initially, it has no permissions, so the IAM policy must be updated to grant it permissions.

-> **Note:** The `client_secret` can only be obtained on creation of the service principal key; it is not stored anywhere after that.

Follow these steps to create service principal with the `contributor` role and a service principal key.

### 1. Create a service principal
-> **Note:** HCP has two types of Service Principals. Organization-Level Service Principals and Project-Level Service Principals. Either can be used with the HCP Terraform Provider. To read more about their differences please see our [documentation page](https://cloud.hashicorp.com/docs/hcp/admin/iam/service-principals).

Once you have registered and logged into the HCP portal, navigate to the Access Control (IAM) page. Select the Service Principals tab and create a new service principal. Give it the role Contributor, since it will be writing resources.

### 2. Create a service principal key

Once the service principal is created, navigate to its detail page by selecting its name in the list. Create a new service principal key.

-> **Note:** Save the client ID and secret returned on successful key creation. The client secret will not be available after creation.

Save the client ID and secret as the environment variables HCP_CLIENT_ID and HCP_CLIENT_SECRET.

Or, configure the provider with the client ID and secret by copy-pasting the values directly into provider config.
-> **Warning:** Hard-coded credentials are not recommended in Terraform configuration outside of local testing and risks secret exposure if committed to a code repository.

```terraform
// Credentials can be set explicitly or via the environment variables HCP_CLIENT_ID and HCP_CLIENT_SECRET
provider "hcp" {
  client_id     = "service-principal-key-client-id"
  client_secret = "service-principal-key-client-secret"
}
```
-> **Note:** If a [Project-Level Service Principal](https://cloud.hashicorp.com/docs/hcp/admin/iam/service-principals) is used, specify the default `project_id` in your provider configuration.

```bash
HCP_CLIENT_ID="..."
HCP_CLIENT_SECRET="..."
```

When client credentials are set, they are always used by the HCP Provider client, regardless of an existing user session.

## User session with browser login

After `v0.45.0`, the HCP Provider supports user session via browser login. User session is ideal for getting started or one-off usage. It works for local development, but will periodically prompt for re-authentication.

To obtain user credentials, the client credential environment variables `HCP_CLIENT_ID` and `HCP_CLIENT_SECRET` **must be unset.**

Upon running `terraform apply` or `terraform plan`, your web browser will navigate to the HCP portal, where you will be prompted to login. Once logged in, you may create new or manage existing resources fully authenticated. Your session will last 24 hours before prompting you to reauthenticate.

```terraform
// If no credentials are set, a user session can be obtained through browser login.
provider "hcp" {}
```
