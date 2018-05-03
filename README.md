# Data Grid Customized Openshift S2I Template

The ideia behind [this template](openshift/resources/datagrid71-postgresql-persistent-s2i.json) is demonstrate how you can customize the configuration of your Data Grid Cluster using the [Official Red Hat xPaaS Image for JBoss Data Grid 7.x](https://access.redhat.com/documentation/en-us/red_hat_jboss_data_grid/7.1/html-single/data_grid_for_openshift/).

This is just a an extension of the original Openshift Template for Data Grid 7.1 ([**`datagrid71-postgresql-persistent.json`**](https://github.com/jboss-openshift/application-templates/blob/master/datagrid/datagrid71-postgresql-persistent.json)). The original Data Grid xPaaS template offers a bunch of ([*parameters* and *environment variables*](https://access.redhat.com/documentation/en-us/red_hat_jboss_data_grid/7.1/html-single/data_grid_for_openshift/#jdg-configuration-environment-variables) you can use to cusomize the Data Grid during the Deployment process on Openshift Platform. **But at the moment it does not cover every configuration details you can use to setup your Data Grid Cluster**. For exemple you can't configure/define things like: *cache base configurations*, *cache writting modes*, *memory based eviction policy*, change the *passivation mode* for a specifc jdbc-cache-store and so on...

Holpefully the Data Grid xPaaS image is also a S2I Builder Image. So all the capabilities offered by the Openshift S2I approach can be leveraged!. See the [image documentation for more details](https://access.redhat.com/documentation/en-us/red_hat_jboss_data_grid/7.1/html-single/data_grid_for_openshift/#using_the_jdg_for_openshift_image_source_to_image_s2i_process).

As an extension [this template](openshift/resources/datagrid71-postgresql-persistent-s2i.json) combines the two capabilities to customize your Data Grid Deployment on Openshift:
 * the original xPaaS image template parameters; and
 * the s2i approach.

Basically you have to add/customize your Data Grid Config using the [`clustered-openshift.xml`](configuration/clustered-openshift.xml) file and than deploy it using the extended template.
During the deployment the **s2i scripts** will copy your `configuration/clustered-openshift.xml` and than the launch scripts will process the original ``## PLACEHOLDERS ##`` accordnly to the parameters defined in the template. 

See the steps below...

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

5. Create the app using Openshift web console (from catalog) or using `oc` CLI.
```bash
oc new-app custom-dg-cluster --name=custom-dg-cluster \
-e APPLICATION_NAME=custom-dg-cluster \
-e USERNAME=admin \
-e PASSWORD='Data@grid1' \
-e JAVA_OPTS_APPEND='-Dcustom_opts_mark=custom-dg-cluster -Xms512m -Xmx512m' \
```

> the template defines default values for the other supported parameters. If you need to change any of them see the template definition to get the name of the parameter.

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
curl -v -k --basic --user 'admin':'Data@grid1' \
https://secure-datagrid-app-demo-jdg.apps.[your local IP].nip.io/rest/DEMO/k1
```

## Content

This sample repo contains the following files:

* `configuration`
  * [`clustered-openshift.xml`](configuration/clustered-openshift.xml): config file based on the original using the same placeholders (## PLACEHOLDER ##) processed by the image lauch script. **Add your customizations in this file**. 

  > The content of this directory will be copied by the Data Grid (xPaaS) s2i builder image, processed by the launch scripts and finally be placed into `/opt/datagrid/standalone/configuration` inside the container...

* `model`
  * [`original-placeholders-clustered-openshift.xml`](model/original-placeholders-clustered-openshift.xml): original `clustered-openshift.xml` config file processed by the image launch scripts during deployment. This is were the `##PLACEHOLDER##` are replaced by the templates ENV VARs/parameters during deployment. **It's here just for reference...**
  * [`sample-datagrid71-template-generated-clustered-openshift.xml`](mode/sample-datagrid71-template-generated-clustered-openshift.xml): This is a sample config file after deployment (after being processed by the launch scripts). **It's here just for reference...**

* `openshift`
  * `resources`
    * [`datagrid71-postgresql-persistent-s2i.json`](openshift/resources/datagrid71-postgresql-persistent-s2i.json): **template that do the deployment magic of this sample!**