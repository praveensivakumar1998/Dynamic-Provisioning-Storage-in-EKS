# Dynamic-Provisioning-Storage-in-EKS
Dynamic volume provisioning in EKS allows Kubernetes to automatically create storage volumes in AWS based on the specifications in the PersistentVolumeClaim (PVC) manifest and attach these volumes to the worker nodes. This feature simplifies storage management by automating the creation and binding of storage resources.

## Key Points:
  1. Persistent Volume in EKS: Works with EC2 instances. Persistent volumes are not supported on Fargate profiles.
  2. Pod Lifecycle: If a pod attached to a persistent volume is terminated, the volume can be re-provisioned for another pod from the same deployment.

## Dynamic vs Static Provisioning:

### Static Provisioning:
  * Manually create PersistentVolume (PV) manifest in the cluster.
  * Manually create a volume in AWS EBS.
  * Store the volumeId in the PersistentVolume manifest.
  * Does not require the EBS CSI driver.

### Dynamic Provisioning:
  * Automatically create volumes based on PVC manifests.
  * No need to manually create volumes or persistent storage.
  * Requires the EBS CSI driver to be installed in the cluster.


# Steps for Dynamic Provisioning in EKS:

## Step-01: Create IAM policy
  * Go to Services -> IAM
  * Create a Policy
      - Select JSON tab and copy paste the below JSON
        ```
            {
              "Version": "2012-10-17",
              "Statement": [
                {
                  "Effect": "Allow",
                  "Action": [
                    "ec2:AttachVolume",
                    "ec2:CreateSnapshot",
                    "ec2:CreateTags",
                    "ec2:CreateVolume",
                    "ec2:DeleteSnapshot",
                    "ec2:DeleteTags",
                    "ec2:DeleteVolume",
                    "ec2:DescribeInstances",
                    "ec2:DescribeSnapshots",
                    "ec2:DescribeTags",
                    "ec2:DescribeVolumes",
                    "ec2:DetachVolume"
                  ],
                  "Resource": "*"
                }
              ]
            }
        ```
  * Review the same in Visual Editor
  * Click on Review Policy
  * Name: Amazon_EBS_CSI_Driver
  * Description: Policy for EC2 Instances to access Elastic Block Store
  * Click on Create Policy
## Step-02: Get the IAM role Worker Nodes using and Associate this policy to that role
**Get Worker node IAM Role ARN**
```
  kubectl -n kube-system describe configmap aws-auth
```
**from output check rolearn**
``
  rolearn: arn:aws:iam::180789647333:role/eksctl-eksdemo1-nodegroup-eksdemo-NodeInstanceRole-IJN07ZKXAWNN
``
  * Go to Services -> IAM -> Roles
  * Search for role with name eksctl-eksdemo1-nodegroup and open it
  * Click on Permissions tab
  * Click on Attach Policies
  * Search for Amazon_EBS_CSI_Driver and click on Attach Policy

  ## Step: 3:
**Install EBS CSI Driver**

  Ensure the EBS CSI driver is installed in the cluster. This driver is essential for dynamic volume provisioning.

  ```
  kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"
  ```
  ## Step: 4:
**Create StorageClass Manifest**

   Define a StorageClass resource in your manifest. Specify the appropriate provisioner and other parameters.
 
  ```
  link
  ```

  ## Step: 5:
**Create PersistentVolumeClaim Manifest**

  Define a PVC resource in your manifest. Reference the StorageClass created in the previous step and specify the volume type and size.

  ```
  link
  ```
  ## Step: 5:
  **Specify the PersistentVolumeClaim in Pod/Deployment Resources:**

  Reference the PVC in your Pod or Deployment manifests to ensure the volumes are attached to the appropriate pods.

  ```
  link
  ```
---

**Important Note on volumeBindingMode: WaitForFirstConsumer:**

  ### Volume Binding Mode:

Setting volumeBindingMode to WaitForFirstConsumer delays the binding and provisioning of the volume until a pod using the PVC is scheduled. This ensures that the volume is created in the same availability zone as the pod, optimizing resource usage and performance.

  ### Validation:

  **To validate that dynamic provisioning is working:**

  * Deploy the StorageClass and PVC manifests.
  * Deploy the application (Pod/Deployment) manifest.
  * Verify that the PVC status transitions from Pending to Bound once the pod is scheduled.
  * Check that an EBS volume is created in AWS and attached to the appropriate EC2 instance.
  
