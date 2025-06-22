****Error: memcache.go:265] unhandled error err= couldn t get current server api group list: the server has asked for the client to provide credentials****
1. There must be IAM Role with policy created before the initialization of creation of both nodegroup and Cluster. 
   To do so add "depends upon" bock  which must be present in both nodegroup and cluster resource and it must point to the IAM policy respectively. 
2. Add "access_config" block to provide the authentication_mode, admin_permission, addons, upgrade_policy etc as mentioned below

```
access_config {
    authentication_mode = "API"
    bootstrap_cluster_creator_admin_permissions = true
    }
    bootstrap_self_managed_addons = true
    tags = var.tags
    version = var.eks_version
    upgrade_policy {
    support_type = "STANDARD"
    }
    depends_on = [ aws_iam_role_policy_attachment.eks_cluster_policy ]
}
```
