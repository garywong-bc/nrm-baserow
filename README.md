# baserow

### Description

Baserow is an open source online database tool and Airtable alternative. Create your own database without technical experience. Our user friendly no-code tool gives you the powers of a developer without leaving your browser.

These are templates used to deploy baserow to the BC Government's OpenShift cluster.


### Installation

#### Recommended

Process and apply build templates.
```
oc process -f backend.build.yml | oc apply -f -
oc process -f mjml.build.yml | oc apply -f -
oc process -f web-frontend.build.yml | oc apply -f -
```

Process and apply deploy templates.
```
oc process -f backend.deploy.yml | oc apply -f -
oc process -f database.deploy.yml | oc apply -f -
oc process -f mjml.deploy.yml | oc apply -f -
oc process -f web-frontend.deploy.yml | oc apply -f -
```

#### Simultaneous

It is possible, but not advisable, to process and apply all templates at once.  OpenShift and the templates are generally able to handle the chaos.

```
for i in $(ls *.yml); do (oc process -f $i | oc apply -f -); done
```

### Labels

The default labels app=baserow and component=<component> are assigned to all objects.  This is useful for troubleshooting.

#### Object grouping

View a summary of the entire stack and associated objects.

```
oc get all,pvc -l app=baserow
```

View each group of component objects separately.  These are backend, database, mjml and web-frontend.

```
oc get all,pvc -l app=baserow -l component=backend
oc get all,pvc -l app=baserow -l component=database
oc get all,pvc -l app=baserow -l component=mjml
oc get all,pvc -l app=baserow -l component=web-frontend
```

### Removal

#### Removal

The entire stack and all associated objects can be deleted.  This is useful for testing and starting over from scratch.

Stack deletion.

```
oc delete all,pvc -l app=baserow
```

Component deletion.

```
oc delete all,pvc -l app=baserow -l component=backend
oc delete all,pvc -l app=baserow -l component=database
oc delete all,pvc -l app=baserow -l component=mjml
oc delete all,pvc -l app=baserow -l component=web-frontend
```

### Reference

[Repository](https://gitlab.com/bramw/baserow)

[baserow.io](https://baserow.io/)

[Getting started](https://baserow.io/docs/index)

[Environment variables](https://baserow.io/docs/getting-started%2Fintroduction)

[Backend API guide](https://baserow.io/docs/getting-started%2Fapi)

[Backend API redoc](https://api.baserow.io/api/redoc/)


### Issues

To do:

1. Close the backend private port.  Traffic through the private port is currently failing.
2. Deployment across namespaces has not been verified.
3. Backend ALLOWED_HOSTS are not being updated.  A workaround uses the ALLOWED_HOSTS_FIX variable in backend.build.yml.
4. Use more secure passwords.  The database is currently using `baserow` for the username, password and db name.
5. mjml has not been tested.
6. OpenShift Overview Panel does not usually link to the web-frontend.
7. Backend liveness probe is causing restarts.  It has temporarily been commented out.
8. Backend and web-frontend are using :latest tags.  Please specify a stable release number for production.
9. There's always something else!
