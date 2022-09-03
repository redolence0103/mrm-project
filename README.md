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
export arn=*****
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

### ECR (container repository) 접속 및 repository 생성
```bash
aws ecr get-login-password --region $region | docker login --username AWS --password-stdin $arn.dkr.ecr.$region.amazonaws.com
aws ecr create-repository --repository-name demo --image-scanning-configuration scanOnPush=true --region $region
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-northeast-2:*****:repository/demo",
        "registryId": "*****",
        "repositoryName": "demo",
        "repositoryUri": "*****.dkr.ecr.ap-northeast-2.amazonaws.com/demo",
        "createdAt": 1662167741.0,
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": true
        }
    }
}
```
### ECR container image 생성에서 push 까지(example)
- Dockerfile 생성
```docker
FROM ubuntu:18.04

# Install dependencies
RUN apt-get update && \
 apt-get -y install apache2

# Install apache and write hello world message
RUN echo 'Hello World!' > /var/www/html/index.html

# Configure apache
RUN echo '. /etc/apache2/envvars' > /root/run_apache.sh && \
 echo 'mkdir -p /var/run/apache2' >> /root/run_apache.sh && \
 echo 'mkdir -p /var/lock/apache2' >> /root/run_apache.sh && \ 
 echo '/usr/sbin/apache2 -D FOREGROUND' >> /root/run_apache.sh && \ 
 chmod 755 /root/run_apache.sh

EXPOSE 80

CMD /root/run_apache.sh
```
- Docker build 및 image 확인
```bash
docker build -t hello-world .
docker images --filter reference=hello-world

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              e9ffedc8c286        4 minutes ago       241MB
```
- Docker image 실행 확인
```bash
docker run -t -p 18080:80 hello-world
```
- ecr docker login
```bash
aws ecr get-login-password --region $region | docker login --username AWS --password-stdin $arn.dkr.ecr.$region.amazonaws.com
```
- ecr repository 생성
```bash
aws ecr create-repository --repository-name demo --image-scanning-configuration scanOnPush=true --region $region
```
- docker image Tag 변경 및 ecr로 push
```bash
docker tag hello-world:latest $ecr/demo:latest
docker push $ecr/demo:latest
```
- push 된 image 확인
```bash
docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                     NAMES
4c77889da3c8   hello-world   "/bin/sh -c /root/ru…"   13 minutes ago   Up 13 minutes   0.0.0.0:18080->80/tcp, :::18080->80/tcp   trusting_keldysh
docker stop 4c77889da3c8
docker rm 4c77889da3c8
docker images
REPOSITORY                                               TAG     IMAGE ID       CREATED          SIZE
855607364597.dkr.ecr.ap-northeast-2.amazonaws.com/demo   latest  fc624389f89c   15 minutes ago   202MB

docker rmi fc624389f89c
docker images
docker run -d -p 18080:80 $ecr/demo
Unable to find image '855607364597.dkr.ecr.ap-northeast-2.amazonaws.com/demo:latest' locally
latest: Pulling from demo
dad0f37c70a6: Already exists 
5fa3a034fcd5: Pull complete 
fe6d729b6014: Pull complete 
cdc788146c85: Pull complete 
Digest: sha256:b5e0f633cd0f23f5fa38cbd38f1040301cbdce88d0672e341f3e60a39166f53f
Status: Downloaded newer image for 855607364597.dkr.ecr.ap-northeast-2.amazonaws.com/demo:latest
f3327a445214a4ec6a6ae1559d19e7e669421f1db49f04c7dd8e13c8580dee63
curl localhost:18080
Hello World!
```
