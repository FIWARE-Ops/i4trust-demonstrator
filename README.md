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

This app-of-apps deployment incorporates repositories for the different organisations and environments in the data space of the reference example:
* **[FIWARE-Ops/i4trust-marketplace](https://github.com/FIWARE-Ops/i4trust-marketplace):** Marketplace and satellite
* **[FIWARE-Ops/i4trust-provider](https://github.com/FIWARE-Ops/i4trust-provider):** Packet Delivery Company (Service Provider)
* **[FIWARE-Ops/i4trust-consumer](https://github.com/FIWARE-Ops/i4trust-consumer)** Happy Pets and No Cheaper (Service Consumers, two instances of this repository are deployed)



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


### Configuration

The deployment is configured via different `values.yaml` files in the folder [i4trust-demonstrator/](./i4trust-demonstrator/). 
There is a separate file for each organisation of this demonstrator:
* Marketplace + Satellite: [i4trust-demonstrator/values-marketplace.yaml](./i4trust-demonstrator/values-marketplace.yaml)
* Packet Delivery (Service Provider): [i4trust-demonstrator/values-pdc.yaml](./i4trust-demonstrator/values-pdc.yaml)
* Happy Pets (Service Consumer): [i4trust-demonstrator/values-happypets.yaml](./i4trust-demonstrator/values-happypets.yaml)
* No Cheaper (Service Consumer): [i4trust-demonstrator/values-nocheaper.yaml](./i4trust-demonstrator/values-nocheaper.yaml)

Several flags allow to enable or disable certain features:
* Satellite: Per default, this deployment uses a test version of the iSHARE satellite, which should be NOT used in any kind of production 
  environment. If an external satellite instance should be used, the deployment of the test satellite can be switched off in 
  [i4trust-demonstrator/values-marketplace.yaml](./i4trust-demonstrator/values-marketplace.yaml). Make sure to edit the satelite references 
  in the other `values.yaml` files.
* Marketplace: During deployment, certain features can be already created, like catalogs or offerings. Configuration is done 
  in the file: [i4trust-demonstrator/values-marketplace.yaml](./i4trust-demonstrator/values-marketplace.yaml)
  - Plugins: Loading of plugins is configured by the parameter `loadPlugins.pluginsEnabled`
  - Addresses: Consumers need a billing addresses, which can be automatically created during deployment via the 
	parameter `initData.createAddress.addressEnabled`. An array needs to be provided for the addresses to be created.
  - Catalog: Initial catalogs can be created for the service provider. This is configured with the parameter `initData.createCatalog.catalogEnabled`.
	An array needs to be provided for the catalogs to be created.
  - Offerings: Product specifications and offerings can be created automatically during deployment. The basic and premium Packet Delivery 
	Service offerings for the standard OIDC flow can be created with `initData.createOfferings.standardI4Trust.basicEnabled` 
	and `initData.createOfferings.standardI4Trust.premiumEnabled`.
  - Indexes: When catalogs and/or offerings have been created automatically, the filling of indexes should be enabled 
	via `initData.fillIndexes.indexesEnabled`


### Uninstall

For removing all components and deleting the applications and namespace, simply remove the branch.



## Credentials

Different accounts are created automatically with default passwords.

| Component     | Username               | Password          | Comment |
|---------------|------------------------|-------------------|---------|
| Keyrock Marketplace | admin@test.com | admin | Admin user of the marketplace |
| Keyrock Packet Delivery | admin@test.com | admin | Admin user of the Provider Keyrock IDP |
| Keyrock Packet Delivery | operator@packetdelivery.com | operator | Operator employee user of Packet Delivery |
| Keyrock Happy Pets | admin@test.com | admin | Admin user of the Happy Pets Keyrock IDP |
| Keyrock Happy Pets | operator@happypets.com | operator | Operator employee user of Happy Pets |
| Keyrock Happy Pets Shop | admin@test.com | admin | Admin user of the Happy Pets Shop Keyrock IDP |
| Keyrock Happy Pets Shop | max.prime@mymail.com | max | Prime user of the Happy Pets shop system |
| Keyrock Happy Pets Shop | steve.standard@mymail.com | steve | Standard user of the Happy Pets shop system |
| Keyrock No Cheaper | admin@test.com | admin | Admin user of the No Cheaper Keyrock IDP |
| Keyrock No Cheaper | operator@nocheaper.com | operator | Operator employee user of No Cheaper |
| Keyrock No Cheaper Shop | admin@test.com | admin | Admin user of the No Cheaper Shop Keyrock IDP |
| Keyrock No Cheaper Shop | cheaty.prime@mymail.com | cheaty | Prime user of the No Cheaper shop system |
| Keyrock No Cheaper Shop | bob.standard@mymail.com | bob | Standard user of the No Cheaper shop system |

Furthermore there are user policies created for the shop users in the Keyrock instances of the Happy Pets and 
No Cheaper shop. Such policies represent the access rights which are delegated by the shop organisations to the shop 
users. These policies provide the following access rights on user level:
| Keyrock instance | Username                   | Entity / Delivery order ID               | Permitted actions         |
|------------------|----------------------------|------------------------------------------|---------------------------|
| Happy Pets Shop  | max.prime@mymail.com       | urn:ngsi-ld:DELIVERYORDER:HAPPYPETS001   | GET(*), PATCH(pta,pda)    |
| Happy Pets Shop  | steve.standard@mymail.com  | urn:ngsi-ld:DELIVERYORDER:HAPPYPETS002   | GET(*)                    |
| No Cheaper Shop  | cheaty.prime@mymail.com    | urn:ngsi-ld:DELIVERYORDER:NOCHEAPER001   | GET(*), PATCH(pta,pda)    |
| No Cheaper Shop  | bob.standard@mymail.com    | urn:ngsi-ld:DELIVERYORDER:NOCHEAPER002   | GET(*)                    |

The entry in the brackets of the column `Permitted actions`, e.g., `(pta,pda)`, denotes the attributes of the 
delivery order entity that can be accessed. The column `Entity / Delivery order ID` displays the ID of the entity, 
which can be accessed by the shop user.  
Note that the shop organisations themselves need the corresponding access rights at the service provider, before the shop users 
can finally access the delivery orders. These access rights can be acquired by the shop organisations on the marketplace. 

Root CA, keys and certificates have been created and self-signed using openssl. Keys and certificates used for this demonstrator 
can be found in the [certs folder](./certs). These should never be used in any kind of production enviroment or on a 
contineously running environment.  
Below table displays the assigned EORIs assigned to the different organisations and their keys/certificates:
| Organisation           | EORI                       |
|------------------------|----------------------------|
| Satellite              | EU.EORI.FIWARESATELLITE    |
| Marketplace            | EU.EORI.DEMARKETPLACE      |
| Packet Delivery        | EU.EORI.DEPROVIDER         |
| Happy Pets             | EU.EORI.DECONSUMERONE      |
| No Cheaper             | EU.EORI.DECONSUMERTWO      |
