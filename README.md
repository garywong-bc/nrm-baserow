# BaseRow on OpenShift

<!-- TOC -->

- [BaseRow on OpenShift](#baserow-on-openshift)
  - [Description](#description)
  - [Installation](#installation)
    - [Network Security Policies](#network-security-policies)
    - [Recommended](#recommended)
    - [Simultaneous](#simultaneous)
    - [Public URL](#public-url)
  - [Labels](#labels)
    - [Object grouping](#object-grouping)
  - [Cleanup](#cleanup)
    - [Removal](#removal)
  - [Reference](#reference)
  - [Issues](#issues)

<!-- /TOC -->

### Description

Baserow is an open source online database tool and Airtable alternative. Create your own database without technical experience. Our user friendly no-code tool gives you the powers of a developer without leaving your browser.

These are templates used to deploy baserow to the BC Government's OpenShift cluster.

### Installation

#### Network Security Policies

Ensure the NSP has been created in each of `-dev`, `-test`, `-prod`. For example:

```bash
oc -n <namespace> process -f https://raw.githubusercontent.com/BCDevOps/platform-services/master/security/aporeto/docs/sample/quickstart-nsp.yaml NAMESPACE=245e18-dev | oc -n <namespace> create -f -
```

#### Recommended

Process and apply build templates.

```bash
oc -n <namespace> process -f backend.build.yml      | oc apply -f -
oc -n <namespace> process -f mjml.build.yml         | oc apply -f -
oc -n <namespace> process -f web-frontend.build.yml | oc apply -f -
```

Process and apply deploy templates.

```bash
oc -n <namespace> process -f database.deploy.yml     | oc apply -f -
oc -n <namespace> process -f backend.deploy.yml      | oc apply -f -
oc -n <namespace> process -f mjml.deploy.yml         | oc apply -f -
```

Ensure that the database pod has been deployed and is accepting connections before:

```bash
oc -n <namespace> process -f web-frontend.deploy.yml | oc apply -f -
```

#### Simultaneous

It is possible, but not advisable, to process and apply all templates at once. OpenShift and the templates are generally able to handle the chaos.

```
for i in $(ls *.yml); do (oc process -f $i | oc apply -f -); done
```

#### Public URL

The URL will of of the form:

`https://baserow-web-frontend-<namespace>.apps.silver.devops.gov.bc.ca/dashboard`

### Labels

The default labels `app=baserow` and `component=<component>` are assigned to all objects. This is useful for troubleshooting.

#### Object grouping

View a summary of the entire stack and associated objects.

```
oc get all,pvc -l app=baserow
```

View each group of component objects separately. These are `backend`, `database`, `mjml` and `web-frontend`.

```bash
oc get all,pvc -l app=baserow -l component=backend
oc get all,pvc -l app=baserow -l component=database
oc get all,pvc -l app=baserow -l component=mjml
oc get all,pvc -l app=baserow -l component=web-frontend
```

### Cleanup

#### Removal

The entire stack and all associated objects can be deleted. This is useful for testing and starting over from scratch.

Entire stack deletion.

```bash
oc -n <namespace> delete all,pvc -l app=baserow
oc -n <namespace> delete secret/baserow-database
```

Component deletion.

```bash
oc -n <namespace> delete all,pvc -l app=baserow -l component=backend
oc -n <namespace> delete all,pvc -l app=baserow -l component=database
oc -n <namespace> delete all,pvc -l app=baserow -l component=mjml
oc -n <namespace> delete all,pvc -l app=baserow -l component=web-frontend
```

### Reference

[Repository](https://gitlab.com/bramw/baserow)

[baserow.io](https://baserow.io/)

[Getting started](https://baserow.io/docs/index)

[Environment variables](https://baserow.io/docs/getting-started%2Fintroduction)

[Backend API guide](https://baserow.io/docs/getting-started%2Fapi)

[Backend API redoc](https://api.baserow.io/api/redoc/)

### Issues

To Do:

- Close the backend private port. Traffic through the private port is currently failing.
- Deployment across namespaces has not been verified.
  - Store image in `-tools` namespace, and pull from there
- Backend ALLOWED_HOSTS are not being updated. A workaround uses the ALLOWED_HOSTS_FIX variable in `backend.build.yml`.
- `mjml` has not been tested.
- OpenShift Overview Panel does not usually link to the web-frontend.
- `backend` liveness probe is causing restarts. It has temporarily been commented out.
- `backend` and `web-frontend` are using `:latest` tags. Please specify a stable release number for production.
- Tighten up the generic Network Security Policy, to the bare minimum to allow the pods to communicate with each other.

Done:

- Use more secure passwords. The database was using a hardcoded password, but the deploy now generates a new password on each deployment.
- Generate a new SECRET_KEY has on each deploy
