# 使用 Dragonfly Operator 配置 S3 快照
在本指南中，我们将了解如何通过 Dragonfly Operator 配置 Dragonfly 实例以使用 S3 作为备份位置。虽然在环境中（通过文件或 env）拥有 AWS 凭证就足以使用 S3，但在本指南中，我们将使用[服务帐户的 AWS IAM 角色](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)来向 Dragonfly Pod 提供凭证。

由于该机制使用 OIDC（OpenID Connect）来验证服务帐户，因此我们还将获得凭证隔离和凭证自动轮换的好处。这样我们就可以避免必须传递长期有效的凭据。这一切都是由 EKS 自动完成的。

## [先决条件](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#prerequisites "Direct link to Prerequisites")
* [安装了 Dragonfly](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation)的 Kubernetes 集群

## 创建 EKS[集群](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#create-an-eks-cluster "创建 EKS 集群的直接链接")
```bash
eksctl create cluster --name df-s3 --region us-east-1  
```
## 创建并关联 IAM OIDC 提供程序到您的[集群](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#create-and-associate-iam-oidc-provider-for-your-cluster%20%22%E4%B8%BA%E6%82%A8%E7%9A%84%E9%9B%86%E7%BE%A4%E5%88%9B%E5%BB%BA%E5%B9%B6%E5%85%B3%E8%81%94%20IAM%20OIDC%20%E6%8F%90%E4%BE%9B%E5%95%86%E7%9A%84%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%22)
按照[AWS 文档](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)，为您的集群创建并关联 IAM OIDC 提供商。

这是后续步骤发挥作用所必需的步骤。

## 创建 S3[存储桶](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#create-an-s3-bucket "创建 S3 存储桶的直接链接")
现在，我们将创建一个 S3 存储桶来存储快照。可以使用 AWS 控制台或 AWS CLI 创建此存储桶。

```bash
aws s3 mb s3://df-s3 

```
## 创建一个Policy读取特定 S3[存储桶](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#create-a-policy-to-read-a-specific-s3-bucket%20%22%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%E5%88%9B%E5%BB%BA%E8%AF%BB%E5%8F%96%E7%89%B9%E5%AE%9A%20S3%20%E5%AD%98%E5%82%A8%E6%A1%B6%E7%9A%84%E7%AD%96%E7%95%A5%22)
我们现在将创建一个策略，允许 Dragonfly 实例读取和写入我们在上一步中创建的 S3 存储桶。

```bash
cat <<EOF > policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::df-s3/*",
                "arn:aws:s3:::df-s3"
            ]
        }
    ]
}
EOF
```
```bash
aws iam create-policy --policy-name dragonfly-backup --policy-document file://policy.json
```
## 将policy与[角色](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#associate-the-policy-with-a-role%20%22%E5%B0%86%E7%AD%96%E7%95%A5%E4%B8%8E%E8%A7%92%E8%89%B2%E5%85%B3%E8%81%94%E7%9A%84%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%22) 关联
现在，我们将在上一步中创建的策略与角色相关联。该角色将由调用的服务帐户使用，`dragonfly-backup`该服务帐户也将在此步骤中创建。

替换`<account-no>`为您的 AWS 帐号。

```bash
eksctl create iamserviceaccount --name dragonfly-backup --namespace default --cluster df-s3 --role-name dragonfly-backup --attach-policy-arn arn:aws:iam::<account-no>:policy/dragonfly-backup --approve
```
## 使用该服务[帐户](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#create-a-dragonfly-instance-with-that-service-account%20%22%E4%BD%BF%E7%94%A8%E8%AF%A5%E6%9C%8D%E5%8A%A1%E5%B8%90%E6%88%B7%E5%88%9B%E5%BB%BA%20Dragonfly%20%E5%AE%9E%E4%BE%8B%E7%9A%84%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%22) 创建一个Dragonfly实例
让我们使用上一步中创建的服务帐户创建一个 Dragonfly 实例。我们还将配置快照位置为我们在前面步骤中创建的 S3 存储桶。

```bash
kubectl apply -f - <<EOF
apiVersion: dragonflydb.io/v1alpha1
kind: Dragonfly
metadata:
  name: dragonfly-sample
spec:
  replicas: 1
  serviceAccountName: dragonfly-backup
  snapshot:
    dir: "s3://df-s3"
EOF
```
## 验证 Dragonfly 实例是否正在[运行](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#verify-that-the-dragonfly-instance-is-running "直接链接到验证 Dragonfly 实例是否正在运行")
```bash
kubectl describe dragonfly dragonfly-sample
```
## 加载数据并终止 Dragonfly[实例](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#load-data-and-terminate-the-dragonfly-instance "直接链接加载数据并终止 Dragonfly 实例")
现在，我们将加载一些数据到 Dragonfly 实例中，然后终止 Dragonfly 实例。

```bash
kubectl run -it --rm --restart=Never redis-cli --image=redis:7.0.10 -- redis-cli -h dragonfly-sample.default SET 1 2
```
```bash
kubectl delete pod dragonfly-sample-0
```
## [验证](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#verification "直接链接到验证")
### 验证备份是否在 S3[存储桶](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#verify-that-the-backups-are-created-in-the-s3-bucket "直接链接到验证备份是否已在 S3 存储桶中创建")
```bash
aws s3 ls s3://df-s3
```
### 验证数据是否自动[恢复](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-s3#verify-that-the-data-is-automatically-restored "直接链接验证数据是否自动恢复")
```bash
kubectl run -it --rm --restart=Never redis-cli --image=redis:7.0.10 -- redis-cli -h dragonfly-sample.default GET 1
```
正如您所看到的，数据会自动从S3存储桶中恢复。这是因为 Dragonfly 实例配置为使用 S3 存储桶作为快照位置。