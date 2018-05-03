# Data Grid Customized Openshift S2I Template

##Steps to deploy this sample Data Grid app

1. Create a new project/namespace on your Openshift Cluster (eg: local Minishift, CDK or `oc cluster up` if you are a Linux user)

2. Import this template

3. Create the `datagrid-app-secret` sample secret (with sample self-signed keystores) used for HTTPS and JGroups internal cluster secured communication.

4. Assign the namespace `viewer` role to the `default` Service Account. This is requeired by the JGroups internal cluster communication.

5. Create the app using Openshift catalog or command line.

### Testing your Data Grid using the REST Server

from a command prompt:

* Create an entry into `DEMO` Cache
```
curl -v -k --basic --user 'admin':'Data@grid1' \
-X PUT \
https://secure-datagrid-app-demo-jdg.apps.10.2.2.2.nip.io/rest/DEMO/k1 \
-d "Hello World!"
```

* Get the entry value from the Cache
```
curl -v -k --basic --user 'admin':'r3dh4t1!' https://secure-datagrid-app-demo-jdg.apps.10.2.2.2.nip.io/rest/DEMO/k1
```