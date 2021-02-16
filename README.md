# EKS_DAST_SAST_SECRETS_DETECTION

Notes: 
- production environment would have 2 separate pipelines CI and CD 
- EKS solution is fronted with a bastion host, nodes are located in private vpc and subnets with access only via jump box.
- maintainability and scalability aspect is achieved via code repository of azure-pipelines.yml file. As per requirements, the solution doesn't include automated EKS nodes updates
- Pipeline uses mostly bash because it's free. Azure Devops offers enterprise grade, paid tools on marketplace.
- k8 api is open to the world and shouldn't be used in production environments
- appilcation should be separated via namespaces
- Pipeline does not stop on errors in sast, dast, secret detection steps to perform a full scan. 
- K8 deployment roles are separated from the eks templates to accelerate deployment



Prerequisites:
- Access to AWS environments with aws cli and using Key and Secrets
- Service connection to AWS and K8


## EKS 
1. Adjust param.json file in eks/templates/ folder
2. EKSPublicAccessCIDRs sets the network with access to update EKS Cluster. 
2. Create EKS:
```
aws cloudformation create-stack --stack-name EKS-Cluster --template-body file://eks/templates/amazon-eks-entrypoint-new-vpc.template.yaml  --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND --parameters file://eks/templates/param.json
```
3. Create ECR to store conatiners
```
aws ecr create-repository \
    --repository-name juice-shop \
    --image-scanning-configuration scanOnPush=true
```
4. SSH to bastion to create roles
    - copy deploy-robot-conf.yaml to the bastion host and apply roles
    ```
    kubectl apply -f deploy-robot-conf.yaml
    ```
    - 
    - get service-account-nam from
    ```
    kubectl get serviceAccounts <service-account-name> -n <namespace> -o=jsonpath={.secrets[*].name}
    kubectl get serviceaccounts deploy-robot -o yaml -o=jsonpath={.secrets[*].name}
    ```
    - get the secret to create service Account in Azure Devops
    ```
    kubectl get secret <service-account-secret-name> -n <namespace> -o json
    kubectl get secret deploy-robot-token-r6jsw -o yaml
    ```
5. Create Kubernetes service connection using Service Account
 and follow instruction - https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/kubernetes/deploy?view=azure-devops
    

Criteria: 
- EKS Cluster fronted with Bastion Host 
- Deployment to EKS using code

## Security folder
1. Create Org in Azure Devops
2. Create new Pipeline
3. Create Service Connection to AWS Environments


Criteria:
- Docker
- Secrets Detection
- Sast
- Dast

## Pipeline

1. Create a build pipeline
2. Import azure-pipeline.yml from the repository to execute
- CI:
    - code build
    - SAST
    - DAST
    - Secrets Detection
    - Report on error with a card and feedback option
3. Go to Options to Enabled "Create work item on failure" to provide strategy to embrace the
continuous feedback when reporting issues to the team by create a ticket and sending to developement and/or security teams.

Criteria:
- single pipeline configuration file (YAML)
- maintainability and scalability in post-implementation ( AUtomated cluster deployment via variables in code ) 
- CD via code triggers
