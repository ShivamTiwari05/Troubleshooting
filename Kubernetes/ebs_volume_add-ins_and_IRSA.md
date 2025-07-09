****Initial Observation****

On deploying Prometheus with Helm, some pods were stuck in Pending:  
```
kubectl get pods -n prometheus
```

Output:  
```
prometheus-alertmanager-0                  0/1     Pending
prometheus-server-b48bbcb5c-p7nqp          0/2     Pending
```

🔍 Investigation Steps
***step 1***  
```
 kubectl describe pod ebs-csi-controller-5d6b79698f-42bns -n kube-system
```
****Output:****   
 ```  
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m43s  default-scheduler  Successfully assigned kube-system/ebs-csi-controller-5d6b79698f-42bns to ip-192-168-95-121.ap-south-1.compute.internal
  Normal  Pulled     2m43s  kubelet            Container image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/aws-ebs-csi-driver:v1.45.0" already present on machine
  Normal  Created    2m43s  kubelet            Created container: ebs-plugin
  Normal  Started    2m42s  kubelet            Started container ebs-plugin
  Normal  Pulling    2m42s  kubelet            Pulling image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/csi-provisioner:v5.3.0-eksbuild.2"
  Normal  Pulled     2m41s  kubelet            Successfully pulled image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/csi-provisioner:v5.3.0-eksbuild.2" in 1.51s (1.51s including waiting). Image size: 37685976 bytes.
  Normal  Created    2m41s  kubelet            Created container: csi-provisioner
  Normal  Started    2m41s  kubelet            Started container csi-provisioner
  Normal  Pulling    2m41s  kubelet            Pulling image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/csi-attacher:v4.9.0-eksbuild.2"
  Normal  Pulled     2m39s  kubelet            Successfully pulled image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/csi-attacher:v4.9.0-eksbuild.2" in 1.413s (1.413s including waiting). Image size: 35652896 bytes.
  Normal  Created    2m39s  kubelet            Created container: csi-attacher
  Normal  Started    2m39s  kubelet            Started container csi-attacher
  Normal  Pulling    2m39s  kubelet            Pulling image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/csi-snapshotter:v8.2.0-eksbuild.2"
  Normal  Pulled     2m38s  kubelet            Successfully pulled image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/csi-snapshotter:v8.2.0-eksbuild.2" in 1.404s (1.404s including waiting). Image size: 33614317 bytes.
  Normal  Created    2m38s  kubelet            Created container: csi-snapshotter
  Normal  Started    2m38s  kubelet            Started container csi-snapshotter
  Normal  Pulling    2m38s  kubelet            Pulling image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/csi-resizer:v1.14.0-eksbuild.2"
  Normal  Pulled     2m36s  kubelet            Successfully pulled image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/csi-resizer:v1.14.0-eksbuild.2" in 1.406s (1.406s including waiting). Image size: 35684724 bytes.
  Normal  Created    2m36s  kubelet            Created container: csi-resizer
  Normal  Started    2m36s  kubelet            Started container csi-resizer
  Normal  Pulled     2m36s  kubelet            Container image "602401143452.dkr.ecr.ap-south-1.amazonaws.com/eks/livenessprobe:v2.16.0-eksbuild.3" already present on machine
  Normal  Created    2m36s  kubelet            Created container: liveness-probe
  Normal  Started    2m36s  kubelet            Started container liveness-probe
```



✅ 2. Checked PVCs  
```
kubectl get pvc -n prometheus
```  
Output:  
```
prometheus-server                   Pending
storage-prometheus-alertmanager-0  Pending
```  

✅ 3. Described PVCs to View Events  
```
kubectl describe pvc prometheus-server -n prometheus
```

****Error observed:****  
```
Events:
  Type     Reason                Age                  From                                                                                      Message
  ----     ------                ----                 ----                                                                                      -------
  Normal   WaitForFirstConsumer  2m31s                persistentvolume-controller                                                               waiting for first consumer to be created before binding
  Warning  ProvisioningFailed    2m31s                ebs.csi.aws.com_ebs-csi-controller-5d6b79698f-ps9xg_677f3c5e-57a9-4a21-b0cd-78ad4ed54b81  failed to provision volume with StorageClass "gp2": rpc error: code = Internal desc = Could not create volume "pvc-30e5db71-3bf2-49b0-9a89-bd0157974b41": could not create volume in EC2: operation error EC2: CreateVolume, get identity: get credentials: failed to refresh cached credentials, failed to retrieve credentials, operation error STS: AssumeRoleWithWebIdentity, https response error StatusCode: 403, RequestID: 552909e8-e4c8-43ad-bd34-0dbcbbe18d60, api error AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
  Warning  ProvisioningFailed    2m30s                ebs.csi.aws.com_ebs-csi-controller-5d6b79698f-ps9xg_677f3c5e-57a9-4a21-b0cd-78ad4ed54b81  failed to provision volume with StorageClass "gp2": rpc error: code = Internal desc = Could not create volume "pvc-30e5db71-3bf2-49b0-9a89-bd0157974b41": could not create volume in EC2: operation error EC2: CreateVolume, get identity: get credentials: failed to refresh cached credentials, failed to retrieve credentials, operation error STS: AssumeRoleWithWebIdentity, https response error StatusCode: 403, RequestID: c2c2a4e4-39df-4e1d-a2b8-1a37cfd7bfa2, api error AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
  Warning  ProvisioningFailed    2m28s                ebs.csi.aws.com_ebs-csi-controller-5d6b79698f-ps9xg_677f3c5e-57a9-4a21-b0cd-78ad4ed54b81  failed to provision volume with StorageClass "gp2": rpc error: code = Internal desc = Could not create volume "pvc-30e5db71-3bf2-49b0-9a89-bd0157974b41": could not create volume in EC2: operation error EC2: CreateVolume, get identity: get credentials: failed to refresh cached credentials, failed to retrieve credentials, operation error STS: AssumeRoleWithWebIdentity, https response error StatusCode: 403, RequestID: 97e99363-153f-431c-ba3b-341139c3157d, api error AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
  Warning  ProvisioningFailed    2m24s                ebs.csi.aws.com_ebs-csi-controller-5d6b79698f-ps9xg_677f3c5e-57a9-4a21-b0cd-78ad4ed54b81  failed to provision volume with StorageClass "gp2": rpc error: code = Internal desc = Could not create volume "pvc-30e5db71-3bf2-49b0-9a89-bd0157974b41": could not create volume in EC2: operation error EC2: CreateVolume, get identity: get credentials: failed to refresh cached credentials, failed to retrieve credentials, operation error STS: AssumeRoleWithWebIdentity, https response error StatusCode: 403, RequestID: 1ae87ab8-965f-4cf6-a5dc-690f6533a45b, api error AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
  Warning  ProvisioningFailed    2m16s                ebs.csi.aws.com_ebs-csi-controller-5d6b79698f-ps9xg_677f3c5e-57a9-4a21-b0cd-78ad4ed54b81  failed to provision volume with StorageClass "gp2": rpc error: code = Internal desc = Could not create volume "pvc-30e5db71-3bf2-49b0-9a89-bd0157974b41": could not create volume in EC2: operation error EC2: CreateVolume, get identity: get credentials: failed to refresh cached credentials, failed to retrieve credentials, operation error STS: AssumeRoleWithWebIdentity, https response error StatusCode: 403, RequestID: 0411ae8b-d36c-4441-82ed-be6665ec9d3e, api error AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
  Warning  ProvisioningFailed    2m                   ebs.csi.aws.com_ebs-csi-controller-5d6b79698f-ps9xg_677f3c5e-57a9-4a21-b0cd-78ad4ed54b81  failed to provision volume with StorageClass "gp2": rpc error: code = Internal desc = Could not create volume "pvc-30e5db71-3bf2-49b0-9a89-bd0157974b41": could not create volume in EC2: operation error EC2: CreateVolume, get identity: get credentials: failed to refresh cached credentials, failed to retrieve credentials, operation error STS: AssumeRoleWithWebIdentity, https response error StatusCode: 403, RequestID: d219f3d5-8686-4ab2-b29e-4dbce4999069, api error AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
  Warning  ProvisioningFailed    88s                  ebs.csi.aws.com_ebs-csi-controller-5d6b79698f-ps9xg_677f3c5e-57a9-4a21-b0cd-78ad4ed54b81  failed to provision volume with StorageClass "gp2": rpc error: code = Internal desc = Could not create volume "pvc-30e5db71-3bf2-49b0-9a89-bd0157974b41": could not create volume in EC2: operation error EC2: CreateVolume, get identity: get credentials: failed to refresh cached credentials, failed to retrieve credentials, operation error STS: AssumeRoleWithWebIdentity, https response error StatusCode: 403, RequestID: f38a8f3c-0c24-4a52-bf20-9a9d1349085b, api error AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
  Normal   Provisioning          24s (x8 over 2m31s)  ebs.csi.aws.com_ebs-csi-controller-5d6b79698f-ps9xg_677f3c5e-57a9-4a21-b0cd-78ad4ed54b81  External provisioner is provisioning volume for claim "prometheus/prometheus-server"
  Warning  ProvisioningFailed    24s                  ebs.csi.aws.com_ebs-csi-controller-5d6b79698f-ps9xg_677f3c5e-57a9-4a21-b0cd-78ad4ed54b81  failed to provision volume with StorageClass "gp2": rpc error: code = Internal desc = Could not create volume "pvc-30e5db71-3bf2-49b0-9a89-bd0157974b41": could not create volume in EC2: operation error EC2: CreateVolume, get identity: get credentials: failed to refresh cached credentials, failed to retrieve credentials, operation error STS: AssumeRoleWithWebIdentity, https response error StatusCode: 403, RequestID: 0c61abbd-88fd-40a4-beeb-7338475bda41, api error AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
  Normal   ExternalProvisioning  8s (x11 over 2m31s)  persistentvolume-controller                                                               Waiting for a volume to be created either by the external provisioner 'ebs.csi.aws.com' or manually by the system administrator. If volume creation is delayed, please verify that the provisioner is running and correctly registered.
```

❌***Root Cause (Two Parts)*** 
🛑****1. Missing EBS CSI Driver Add-on****

The AWS EBS CSI driver was not installed in the cluster initially. This is required to handle dynamic EBS volume provisioning.
🛑 2. ****Misconfigured IRSA (IAM Role for Service Account)****  

Even after installing the EBS CSI driver, IRSA failed due to:  

    The trust policy had an invalid sub:  

    "system:serviceaccount:prometheus:*"  ❌ (wildcards not allowed)  

    The EBS CSI controller runs in kube-system, not prometheus.  
  
✅ Resolution Steps  
1. 🔧 Installed AWS EBS CSI Driver  

Installed the add-on using:  
```
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster <your-cluster-name> \
  --service-account-role-arn arn:aws:iam::647187953046:role/<EBS_CSI_IAM_ROLE> \
  --force
```
    This enabled the CSI controller to handle EBS volume provisioning.

2. 🛠️ Created Proper Service Account  

In namespace kube-system:  
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ebs-csi-controller-sa
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::647187953046:role/<EBS_CSI_IAM_ROLE>
```
3. ✅ Fixed Trust Policy on IAM Role  

Updated the IAM Role trust policy to match the actual service account:  
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::647187953046:oidc-provider/oidc.eks.ap-south-1.amazonaws.com/id/7344D22ED59064F8EA97532181474C2E"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.ap-south-1.amazonaws.com/id/7344D22ED59064F8EA97532181474C2E:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa",
          "oidc.eks.ap-south-1.amazonaws.com/id/7344D22ED59064F8EA97532181474C2E:aud": "sts.amazonaws.com"
        }
      }
    }
  ]
}
```  
4. 🔄 Restarted EBS CSI Controller  
```
kubectl rollout restart deployment ebs-csi-controller -n kube-system
```

5. 📦 PVCs Successfully Bound  
  
PVCs for Prometheus were automatically retried and successfully bound to dynamically provisioned EBS volumes.  
```
kubectl get pvc -n prometheus
```

Output:  
```
prometheus-server                   Bound
storage-prometheus-alertmanager-0  Bound
```
  
✅ Final Outcome  
  
    ✅ AWS EBS CSI driver was installed and properly configured.  
  
    ✅ IAM trust policy and IRSA were fixed.  
  
    ✅ Prometheus PVCs successfully bound to EBS volumes.  
  
    ✅ Pending pods moved to Running.  
    
