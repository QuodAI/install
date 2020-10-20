### Prerequisites
Reserve an external IP for VPN connection.

Install command line tools

- envsubst
- Helm 3
- kubectl

Install a Kubernetes cluster on your cloud provider, connect to your cluster to through `kubectl`

### Kubernetes minimum hardware requirements
- 5 vCPUs
- 10 GB Memory
- At least 2 nodes with more than 2 vCPUs and 4 GB Memory

### Install Kubernetes

| Options          | Description                                                                                          |
|------------------|------------------------------------------------------------------------------------------------------|
| MYSQL_PASSWORD   | Password for MySQL user                                                                              |
| MYSQL_CLUSTER_IP | Internal IP for MySQL.  Must be in the range of service subnet of your Kubernetes cluster            |
| ES_CLUSTER_IP    | Internal IP for Elasticsearch.  Must be in the range of service subnet of your Kubernetes cluster    |
| GIT_CLUSTER_IP   | Internal IP for git server proxy.  Must be in the range of service subnet of your Kubernetes cluster |
| OPENVPN_IP       | External IP for OpenVPN server (reserved IP in [Prerequisites](#prerequisites)). We will connect to your cluster through this IP                     |
| GIT_SERVER_URL   | URL to your self-hosted git server e.g https://gitlab.mycompany.com                                            |

How to get service subnet range of Kubernetes

```
SVCRANGE=$(echo '{"apiVersion":"v1","kind":"Service","metadata":{"name":"tst"},"spec":{"clusterIP":"1.1.1.1","ports":[{"port":443}]}}' | kubectl apply -f - 2>&1 | sed 's/.*valid IPs is //')
echo $SVCRANGE
```

We will give you a key file call `hybrid-key.json` to access our Docker container, please put this key file in this directory

Example command
```shellscript
MYSQL_PASSWORD=changeme \
MYSQL_CLUSTER_IP=10.0.0.1 \
ES_CLUSTER_IP=10.0.0.2 \
GIT_CLUSTER_IP=10.0.0.3 \
OPENVPN_IP=34.87.100.0 \
GIT_SERVER_URL=https://gitlab.mycompany.com \
sh deploy.sh
```

After running deploy script, checking the deploy status with command

```
kubectl get pods
```

Wait all pods to have:

- Status: Running and READY value is 1/1
- Status: Completed

Example successful state

```
NAME                               READY   STATUS      RESTARTS   AGE
es-79f6967fc9-8tlcr                1/1     Running     0          20h
es-sequential-jobs-t6nhl           0/1     Completed   0          20h
git-server-proxy-94cbdfdc8-tsnnd   1/1     Running     0          26h
mysql-8496c7dbcb-m4tk5             1/1     Running     0          21h
openvpn-server-6576ff9d8f-9z8kj    1/1     Running     0          26h
sequential-jobs-5qjzp              0/1     Completed   0          21h
```

Execute script `generate_vpn_client.sh` to generate the OpenVPN client credentials. 
A config file called `kubeVPN.ovpn` will be generated after you run this script successfully.

### Create Gitlab application
A Gitlab application with following permissions is needed, `Redirect URI` will be given by us. 
The admin of Gitlab should create this application

- api
- read_user
- read_repository
- profile
- email

### Share your setup with us
Finally, email your setup information to support@quod.ai:
- All values in [Install Kubernetes options](#install-kubernetes)
- `kubeVPN.ovpn` file
- Application ID and Client Secret of Gitlab application

### Uninstall all deployments and disks

```
sh uninstall.sh
```
