# i4Trust Reference Example Demonstrator

App-of-apps for the full i4Trust reference example

> :bulb: This repository just provides a setup for temporary demonstration purposes. It is not recommended to be used in a production enviroment. Credentials are visible in clear text and are not encrypted. Installations should be deleted when demonstrations/presentations/etc. have finished. 

The GitHub actions of this repo are configured to deploy a full instance with all components 
required for this demonstrator, as soon as a branch is created. It is meant for a temporary deployment only. 
Note that the deployment should be deleted after 
each presentation/demo/etc., since there are only test accounts registered and credentials are visible in clear text in this 
repo.

Before moving this installation to a production environment, make sure to encrypt all credentials, keys, etc., e.g., 
using [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets).

All scripts are developed for using an OpenShift Kubernetes cluster, but can be easily adapted for any 
kind of infrastructure.


## Deployment

It is required to setup two GitHub secrets in the 
repository ([also check this manual](https://github.com/FIWARE-Ops/marinera/blob/main/documentation/GITHUB_CI.md#openshift-service-account-permissions)):
* `OPENSHIFT_SERVER`: Server URL of the OpenShift cluster
* `OPENSHIFT_TOKEN`: Token from an OpenShift service account with sufficient permissions for creation/deletion of projects and applications, role assignments and deployments via Helm charts (e.g., with `cluster-admin` role) 

In order to deploy all components, simply create a branch which is named differently than `main`. 
The GitHub action will deploy all components to the namespace `i4t-{BRANCH_NAME}`. 

* Branches named `no-deploy/**` will not be deployed. This is useful in the case that one first wants to 
  develop a new configuration without deploying it immediately. For deployment after finishing the development, 
  one simply creates a branch out of the `no-deploy/**` branch named differently than `main` and `no-deploy/**`.

Routes for externally exposed services are automatically created and hostnames are set dynamically. In order to 
retrieve the created hostnames, one can run, e.g., 
```shell
kubectl -n i4t-{BRANCH_NAME} get routes
```
or check in the OpenShift console or in ArgoCD.

For the marketplace, when the branch was called `demo`, this might give you something like
```shell
NAME                                                               HOST/PORT                                                                    PATH   SERVICES                                PORT    TERMINATION     WILDCARD
route.route.openshift.io/marketplace-biz-ecosystem-logic-proxy-0   marketplace-biz-ecosystem-logic-proxy-0-i4t-demo.apps.fiware.fiware.dev          marketplace-biz-ecosystem-logic-proxy   <all>   edge/Redirect   None
```
The marketplace logic proxy would be available under the URL: `https://marketplace-biz-ecosystem-logic-proxy-0-i4t-demo.apps.fiware.fiware.dev`.


### Uninstall

For removing all components and deleting the applications and namespace, simply remove the branch.



## Credentials

Different accounts are created automatically with default passwords.

| Component     | Username               | Password          | Comment |
|---------------|------------------------|-------------------|---------|
| Keyrock Marketplace | admin@test.com | admin | Admin user of the marketplace |

Root CA, keys and certificates have been created and self-signed using openssl. Keys and certificates used for this demonstrator 
can be found in the [certs folder](./certs). These should never be used in any kind of production enviroment or on a 
contineously running environment.  
Below table displays the assigned EORIs assigned to the different organisations and their keys/certificates:
| Organisation           | EORI                       |
|------------------------|----------------------------|
| Satellite              | EU.EORI.FIWARESATELLITE    |
| Marketplace            | EU.EORI.DEMARKETPLACE      |
