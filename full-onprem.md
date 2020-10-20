# How to install Quod AI on premise

## Prerequisites

Please install the following CLI tools:

- Helm 3
- kubectl

## Preparation

1. Reserve 5 IPs and domains for deployment.

   We recommend the following domains: 
   - webhook.quod-ai.acme.org
   - api.quod-ai.acme.org
   - matomo.quod-ai.acme.org
   - sentry.quod-ai.acme.org
   - quod-ai.acme.org
    
    ---
    📝 Keep track of the domains, IPs and allocation IDs of the IPs (QS1)
    
    ---
1. Create a Kubernetes cluster on your cloud provider according to our hardware requirements

## Deploy Quod AI

1. Download the deployment files from [Google Drive](https://drive.google.com/drive/u/0/folders/1sd2hHDBpt-wV2eBdK9ncKmqXO1H6Krmz). If you don't have permissions to access the folder, please contact support at quod dot ai.
1. Connect to your cluster using `kubectl`
1. Install docker credentials to pull images from our private registry
    ```shellscript
    sh install-docker-credentials.sh
    ```
1. If you have a Kubernetes [TLS secret file](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls), skip to the next step. Otherwise follow the instructions below.
    1. Make a copy of the TLS template
        ```shellscript
        cp tls-secret.template.yaml tls-secret.yaml
        ```
    1. Edit `tls-secret.yaml`
       
       ---
       📝 Keep track of name of TLS secret at `metadata.name` (QS2)

        ---

1. Apply your TLS secret
```shellscript
kubectl apply -f tls-secret.yaml
```
1. Install Sentry helm chart
   1. Edit `sentry-helm/values.yaml`. 
   2. Deploy Sentry
    ```shellscript
    helm install sentry ./sentry-helm --wait
    ```
   3. Go to `sentry.quod-ai.acme.org` to setup a sentry project. 

        ---
        📝 Keep track of the project url (QS3)

        ---

1. Install Bitbucket Application Link
    1. Follow instructions from [Atlassian](https://developer.atlassian.com/server/jira/platform/oauth/)
    2. You will only need to finish `Step 1: Configure`
   
        ---
        📝 Keep the value of `Consumer key` and `Private key` (QS4)
        
        ---

2. Install QuodAI helm chart
    1. Fill our required values in `quod-helm/values.yaml`. 
    2. Deploy Quod AI
        ```shellscript
        helm install quod ./quod-helm --wait
        ```
3. Setup Quod AI for your organization
   1. Go to the URL for Quod AI web in step QS1 `https://quod-ai.acme.org`
