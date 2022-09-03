# mrm-project
## AWS EKS 접속 절차 (ubuntu bash shell 기준)
### aws login 과정
```bash
aws configure
AWS Access Key ID [None]: {Access Key}
AWS Secret Access Key [None]: {Secret access key}
Default region name [None]: ap-northeast-2
Default output format [None]: json
```
     
### 환경 설정 (bash)
```bash
export region=ap-northeast-2
export arn=855607364597
export ecr=$arn.dkr.ecr.$region.amazonaws.com
```

### EKS config 설정
```bash
aws eks update-kubeconfig --name mrn-cluster
chmod 600 ~/.kube/config
```
   
### 각 기업의 namespace 사용
```bash
kubectl get ns
kubectl config set-context --current --namespace {기업 이니셜}
```
