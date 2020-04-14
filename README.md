# Deploy Node-RED onto OpenShift 4.3 with IBM Cloud Object Storage

## Setup

Before beginning, ensure that you have:

1. Access to an [OpenShift 4.x](https://cloud.ibm.com/kubernetes/catalog/create?platformType=openshift) on IBM Cloud
2. Provisioned [IBM Cloud Object Storage (COS)](https://cloud.ibm.com/catalog/services/cloud-object-storage) storage
3. Retrieved the `apikey` from your COS service ([instructions](https://cloud.ibm.com/docs/openshift?topic=openshift-object_storage#create_cos_secret))
4. Retrieved the `GUID` of your COS service ([instructions](https://cloud.ibm.com/docs/openshift?topic=openshift-object_storage#service_credentials))
4. Decided on a project name, e.g. `editor-demo`
5. Install Helm 3 and associated charts repos ([instructions](https://cloud.ibm.com/docs/openshift?topic=openshift-helm#install_v3))
6. Downloaded Yaml Templating Tool from ([get-ytt](https://get-ytt.io)). To learn more about this project, have a look at [this IBM Developer blog post](https://developer.ibm.com/depmodels/cloud/blogs/yaml-templating-tool-to-simplify-complex-configuration-management/).

## Install IBM Cloud Object Storage driver onto OpenShift

1. Install IBM COS plugin onto your OpenShift cluster in the `default` project (`oc project default`) using IBM Cloud's [instructions](https://cloud.ibm.com/docs/openshift?topic=openshift-object_storage#install_cos).
2. Create a new project with the name you chose, e.g. `oc new-project editor-demo`. 
3. Switch to this new project, e.g. `oc project editor-demo`.
4. Using the retrieved `apikey` for your COS service, create a secret named `cos-write-access` inside the project with 

```
oc create secret generic cos-write-access --type=ibm/ibmc-s3fs --from-literal=api-key=<apikey> --from-literal=service-instance-id=<GUID>
```

5. Verify that the secret has been created with `oc get secret`.
6. Choose a [storage class](https://cloud.ibm.com/docs/openshift?topic=openshift-object_storage#configure_cos) to use with your Node-RED deployment. In the demo `all.yaml`, I selected `ibmc-s3fs-standard-regional`. **Standard** for hot data used frequently, **regional** as I did not need the resiliency of multiple regions.
7. Decide on a name of the bucket. A new bucket is created when a persistent volume claim (PVC) is applied. By default, the name of the project is set as the bucket's name.

## How to use this repository

1. Clone the repo.
2. Fill in the configuration variables at the top of the file, i.e. `app_name`, `domain`, `cos_storage_class`, `cos_secret_name`, `cos_storage_space` and `image_name`. Only `app_name` and `domain` are required. Other fields are optional.
3. Run `ytt -f all.yaml > configured.yaml`. You will get something similar to the below output

```
$ oc apply -f configured.yaml
service/editor-demo-svc created
route.route.openshift.io/secure-route created
persistentvolumeclaim/editor-demo-pvc created
configmap/editor-settings created
deployment.apps/editor-demo-deploy created
```

4. Check when the deployment is complete with `oc get deploy --watch`.
5. When the deployment's current state matches the desired state, run `oc get route secure-route` to get the URL. Copy the HOST/PORT link, prepend `https://` to the link and navigate to this link in your favourite browser. 

Alternatively, you can also run `oc get route secure-route -o=jsonpath="{@.spec.host}"` to get just the host. If you're feeling fancy and working on a Linux-based OS or macOS, you can also run the following command to get the link.

```
echo "https://$(oc get route secure-route -o=jsonpath={@.spec.host})"
```
