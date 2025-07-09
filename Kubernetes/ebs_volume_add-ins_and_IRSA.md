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
✅ 1. Checked PVCs  
```
kubectl get pvc -n prometheus
```  
Output:  
```
prometheus-server                   Pending
storage-prometheus-alertmanager-0  Pending
```  

✅ 2. Described PVCs to View Events  
```
kubectl describe pvc prometheus-server -n prometheus
```

****Error observed:****

failed to provision volume with StorageClass "gp2": rpc error:
AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity

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
    
