# Data Grid Customized Openshift S2I Template

## Steps to deploy this sample Data Grid app

1. Create a new project/namespace on your Openshift Cluster (eg: local Minishift, CDK or `oc cluster up` if you are a Linux user)
```bash
oc login ...

oc new-project dg-s2i-demo
```

2. Import this template
```bash
oc create -f https://raw.githubusercontent.com/rafaeltuelho/datagrid-s2i-example/master/openshift/resources/datagrid71-postgresql-persistent-s2i.json
```

3. Create the `datagrid-app-secret` sample secret (with sample self-signed keystores) used for HTTPS and JGroups internal cluster secured communication.
```bash
oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/secrets/datagrid-app-secret.json
```

4. Assign the `view` role to the `default` Service Account. This is requeired by the JGroups internal cluster communication.
```bash
oc policy add-role-to-user view system:serviceaccount:$(oc project -q):default -n $(oc project -q)
```

5. Create the app using Openshift catalog or command line.

### Testing your Data Grid using the REST Server

... from a command prompt:

* Create an entry into `DEMO` Cache
```bash
curl -v -k --basic --user 'admin':'Data@grid1' \
-X PUT \
https://secure-datagrid-app-demo-jdg.apps.[your local IP].nip.io/rest/DEMO/k1 \
-d "Hello World!"
```

* Get the entry value from the Cache
```bash
curl -v -k --basic --user 'admin':'r3dh4t1!' \
https://secure-datagrid-app-demo-jdg.apps.[your local IP].nip.io/rest/DEMO/k1
```

## Content

This sample repo contains the following files:

* configuration
  * [`clustered-openshift.xml`](configuration/clustered-openshift.xml): config file based on the original using the same placeholders (## PLACEHOLDER ##) processed by the image lauch script. **Add your customizations in this file**. 
  > It will be picked by the Data Grid (xPaaS) s2i builder image, processed by the lauch scripts and finally be placed into `/opt/datagrid/standalone/configuration` inside the container...

  * [`rest-application-roles.properties`](configuration/rest-application-roles.properties): just a sample the default JBoss creadentials mapping file used for authentication
  * [`rest-application-users.properties`](configuration/rest-application-users.properties): same here

* model
  * [`original-placeholders-clustered-openshift.xml`](model/original-placeholders-clustered-openshift.xml): original `clustered-openshift.xml` config file processed by the image launch scripts during deployment. This is were the `##PLACEHOLDER##` are replaced by the templates ENV VARs/parameters during deployment. **It's here just for reference...**
  * [`sample-datagrid71-template-generated-clustered-openshift.xml`](mode/sample-datagrid71-template-generated-clustered-openshift.xml): This is a config file after deployment. **It's here just for reference...**

* openshift
  * resources
    * [`datagrid71-postgresql-persistent-s2i.json`](openshift/resources/datagrid71-postgresql-persistent-s2i.json): **template that do the deployment magic of this sample!**