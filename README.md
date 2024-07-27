# Overview
Datashim is a Kubernetes Framework to provide easy access to S3 and NFS Datasets within pods. It orchestrates the provisioning of Persistent Volume Claims and ConfigMaps needed for each Dataset.

## Using manifests
If you prefer, you can install Datashim using the manifests provided. Start by creating the dlf namespace with:
Start by creating the dlf namespace with:
 
create ns

  kubectl create ns dlf

In order to quickly deploy Datashim

  kubectl apply -f https://raw.githubusercontent.com/datashim-io/datashim/master/release-tools/manifests/dlf.yaml

  kubectl label namespace default monitor-pods-datasets=enabl

### Post-install steps
Ensure that Datashim has been deployed correctly and ready by using the following command:

  kubectl wait --for=condition=ready pods -l app.kubernetes.io/name=datashim -n dlf

To use Datashim, we need to create a Dataset: we can do so by editing and running the following:

cat <<EOF | kubectl apply -f -
apiVersion: datashim.io/v1alpha1
kind: Dataset
metadata:
  name: example-dataset
spec:
  local:
    type: COS
    accessKeyID:  test@1234
    secretAccessKey: test@1234
    endpoint: https://minio.rockvilletech.com:9000
    bucket: demo
    region: us-east-1 
EOF

If everything worked, you should now see a PVC named example-dataset which you can mount in your pods. Assuming you have labeled your namespace with monitor-pods-datasets=enabled as mentioned in the post-install steps, you will now be able to mount the PVC in a pod as simply as this:


apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes:
    - name: "example-dataset"
      persistentVolumeClaim:
        claimName: "example-dataset"
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: "/mount/dataset1" 
          name: "example-dataset"
