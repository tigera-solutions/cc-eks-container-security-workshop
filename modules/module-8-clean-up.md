# Module 8 - Clean up

1. Delete application stack to clean up any `loadbalancer` services.

   ```bash
   kubectl delete -f manifests/dev-app-manifest.yaml
   kubectl delete -f manifests/kubernetes-manifests.yaml
   kubectl delete -f manifests/website.yaml
   ```

2. Delete EKS cluster.

   ```bash
   source ~/labVars.env
   eksctl delete cluster --name $CLUSTERNAME --region $REGION
   ```
   
3. Delete Cloud9 instance.

   Navigate to `AWS Console` > `Services` > `Cloud9` and remove your workspace environment, e.g. `tigera-workspace`.

4. Delete IAM role created for this workshop.

   Use your local shell to run the follow AWS CLI commands:

   ```bash
   IAM_ROLE='tigera-workshop-admin'
   ADMIN_POLICY_ARN=$(aws iam list-policies --query 'Policies[?PolicyName==`AdministratorAccess`].Arn' --output text)
   aws iam detach-role-policy --role-name $IAM_ROLE --policy-arn $ADMIN_POLICY_ARN
   aws iam remove-role-from-instance-profile --instance-profile-name $IAM_ROLE --role-name $IAM_ROLE
   aws iam delete-instance-profile --instance-profile-name $IAM_ROLE
   aws iam delete-role --role-name $IAM_ROLE
   ```

   If this command fails, you can remove the role via AWS Console once you delete the Cloud9 instance

---

[:arrow_left: Module 7 - Traffic visualization inside your Kubernetes Cluster](/modules/module-7-visibility.md) <br>

[:leftwards_arrow_with_hook: Back to Main](/README.md) 