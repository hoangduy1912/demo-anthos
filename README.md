

-------------------
Deploy on EKS
https://www.youtube.com/watch?v=jEsRHk24lQ8

export CLUSTER_NAME=$(terraform output -raw cluster_name)
export MEMBERSHIP_NAME="anthos-on-eks"
export AWS_REGION=$(terraform output -raw region)
aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME

export KUBECONFIG_CONTEXT=$(kubectl config current-context)
export OIDC_URL=$(aws eks describe-cluster --name $CLUSTER_NAME --region $AWS_REGION --query "cluster.identity.oidc.issuer" --output text)

gcloud container hub memberships register anthos-on-eks --context=arn:aws:eks:ap-southeast-1:535929111158:cluster/anthos-on-eks --kubeconfig="~/.kube/config" --enable-workload-identity --public-issuer-url=https://oidc.eks.ap-southeast-1.amazonaws.com/id/1E03D6DBB8972ED954CB4BE0AE85D7B7

kubectl create serviceaccount -n kube-system admin-user
kubectl create clusterrolebinding admin-user-binding --clusterrole cluster-admin --serviceaccount kube-system:admin-user

SECRET_NAME=$(kubectl get serviceaccount -n kube-system admin-user -o jsonpath='{$.secrets[0].name}')

kubectl get secret -n kube-system ${SECRET_NAME} -o jsonpath='{$.data.token}' \ 
| base64 -d | sed $'s/$/\\\n/g'