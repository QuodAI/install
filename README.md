# How to install Quod AI on premise

## Prerequisites

Please install the following CLI tools:

- envsubst
- Helm 3
- kubectl

## Deploy Quod AI

1. Reserve an external IP for VPN connection. 

	See documentation for [Google Cloud](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address), [AWS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html).
1. Install Kubernetes 
	
	Install a Kubernetes cluster on your cloud provider. The minimum requirement for the cluster is:
	
	- 5 vCPUs
	- 10 GB Memory
	- At least 2 nodes with more than 2 vCPUs and 4 GB Memory
		
1. Deploy Quod AI
	1. Download the deployment files from [Google Drive](https://drive.google.com/drive/folders/16AhYhBLm_ulZFXuzETAmfiCgjtJrL5nq?usp=sharing). If you don't have permissions to access the folder, please contact [support@quod.ai](mailto:support@quod.ai).
 	1. Connect to your cluster using `kubectl`
 	1. Install Quod AI hybrid on premise:

	Here is a sample command below to install Quod AI. The command will install MySQL, Elasticsearch and OpenVPN.

	```bash
	MYSQL_PASSWORD=changeme \
	MYSQL_CLUSTER_IP=10.0.0.1 \
	ES_CLUSTER_IP=10.0.0.2 \
	GIT_CLUSTER_IP=10.0.0.3 \
	OPENVPN_IP=34.87.100.0 \
	GIT_SERVER_URL=https://gitlab.mycompany.com \
	sh deploy.sh
	```
		
	**⚠️ Please keep track of the command you run. You will share it with us later. ⚠️**
		
	Where:
		
	* `MYSQL_PASSWORD`: new password that will be used when installing MySQL.
	* `MYSQL_CLUSTER_IP`: internal IP for MySQL.  Must be in the range of service subnet of your Kubernetes cluster. See below for instructions to get range of service subnet.
	* `ES_CLUSTER_IP`: internal IP for Elasticsearch.  Must be in the range of service subnet of your Kubernetes cluster .
	* `GIT_CLUSTER_IP`: internal IP for git server proxy.  Must be in the range of service subnet of your Kubernetes cluster.
	* `OPENVPN_IP`: external IP for OpenVPN server (reserved IP in [Prerequisites](#prerequisites)). We will connect to your cluster through this IP.
	* `GIT_SERVER_URL`: URL to your self-hosted git server e.g https://gitlab.mycompany.com.
		
	1. Check the deploy status: `kubectl get pods`
	1. Wait all pods to be successful:
		- Status: **Running** and **Ready** value is 1/1
		- OR Status: **Completed**

		See below for an example of successful state:
		
		```
		NAME                               READY   STATUS      RESTARTS   AGE
		es-79f6967fc9-8tlcr                1/1     Running     0          20h
		es-sequential-jobs-t6nhl           0/1     Completed   0          20h
		git-server-proxy-94cbdfdc8-tsnnd   1/1     Running     0          26h
		mysql-8496c7dbcb-m4tk5             1/1     Running     0          21h
		openvpn-server-6576ff9d8f-9z8kj    1/1     Running     0          26h
		sequential-jobs-5qjzp              0/1     Completed   0          21h
		```
	1. Generate the OpenVPN client credentials: `generate_vpn_client.sh`. A config file called `kubeVPN.ovpn` will be generated. **⚠️ Please keep track of kubeVPN.ovpn. You will share it with us later. ⚠️**.

## Create an application on your Git host

See instructions for [GitHub](https://docs.github.com/en/free-pro-team@latest/developers/apps/creating-an-oauth-app), [GitLab](https://docs.gitlab.com/ee/integration/oauth_provider.html#adding-an-application-through-the-profile). 

**⚠️ Please keep track of Application ID and Client Secret of your application. You will share it with us later. ⚠️**

When you create the application, you will be asked for:

1. **Authorization callback URL** or **Redirect URI**. See our original email for the information. This is customized to your organization.
2. 	**Permission scopes**
	- For **GitHub**: all scopes. Unfortunately GitHub offers only 1 scope to read code (all or nothing). You can see from [GitHub scope for apps](https://docs.github.com/en/free-pro-team@latest/developers/apps/scopes-for-oauth-apps#available-scopes) that that's the only scope to access source code, unfortunately. Someone has [complained about it on GitHub](https://github.com/jollygoodcode/jollygoodcode.github.io/issues/6) :-)

	- For **GitLab**: `api, read_user, read_repository, profile, email`

## Share your setup 

Submit the information you've tracked via our [Google Form](https://docs.google.com/forms/d/e/1FAIpQLSefjQ4DuRoch0WuU9QHK_nNuST58M6GkzewMBjU0MBNk7HBsg/viewform).

## FAQ

#### How do I get the service subnet range for Kubernetes? 
	
```
SVCRANGE=$(echo '{"apiVersion":"v1","kind":"Service","metadata":{"name":"tst"},"spec":{"clusterIP":"1.1.1.1","ports":[{"port":443}]}}' | kubectl apply -f - 2>&1 | sed 's/.*valid IPs is //')
echo $SVCRANGE
```

#### How do uninstall? 

If you need to reset or start over, run: `sh uninstall.sh`
