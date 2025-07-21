****Error:****  crash loop back off  

Reason:  
ISRA Authentication issue: IRSA was not provisioned to provide the pod access for PVC (persistent volume Claim)  

Steps involved in troubleshoting:    
1) checking the pod status  
2) describing the pods  
3) looking at the logs : there was clear mention about the issue

Resoulution:  
1) creating IAM role and attach policy  
2) configure trust policy  
3) Creating service account  
4)  Service account Yaml and deploy it  
5)  deploy the pod with service account Or if pod is already deployed, patch it and then restart the pod.



For pod Patch:  

```
kubectl patch deployment <your-deployment-name> \
  -n default \
  -p '{"spec": {"template": {"spec": {"serviceAccountName": "my-app-sa"}}}}'
```



for Restart:  
```
kubectl rollout restart deployment <your-deployment-name> -n default
```
