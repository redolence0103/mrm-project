# mrm-project
## AWS EKS 접속 절차 (ubuntu bash shell 기준)
1. aws login 과정
   aws configure
     AWS Access Key ID [None]: {Access Key}
     AWS Secret Access Key [None]: {Secret access key}
     Default region name [None]: ap-northeast-2
     Default output format [None]: json
     
2. 환경 설정 (bash)
   export region=ap-northeast-2
   export arn=855607364597
   export ecr=$arn.dkr.ecr.$region.amazonaws.com

3. EKS config 설정
   aws eks update-kubeconfig --name mrn-cluster
   chmod 600 ~/.kube/config
   
4. 각 기업의 namespace 사용
   kubectl get ns
   kubectl config set-context --current --namespace {기업 이니셜}
