# anchore-engine
Deploying anchore-engine using Kubernetes

![K8s](https://img.shields.io/badge/-kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![postgres](https://img.shields.io/badge/-Postgres-005571?style=for-the-badge&logo=PostgreSQL&logoColor=white)

[Anchore](https://github.com/anchore/anchore-engine) is a service that analyzes docker images and applies user-defined acceptance policies to allow automated container image validation and certification 

#### :small_blue_diamond: Anchore engine architecture - from Sysdig [ kubernetes security guide](https://sysdig.com/blog/kubernetes-security-guide/):
Anchore Engine architecture is comprised of six components that can either be deployed ina single container or scaled out:
1. API Service: Central communication interface that can be accessed by code, using a REST API, or directly, using the command line.
2. Image Analyzer Service: Executed by the “worker”, these Anchore nodes perform the actual Docker image scanning.
3. Catalog Service: Internal database and system state service.
4. Queuing Service: Organizes, persists and schedules the engine tasks.
5. Policy Engine Service: Policy evaluation and vulnerabilities matching rules.
6. Kubernetes Webhook Service: Kubernetes-specific webhook service to validate images before they are spawned.

![Anchore-architecture](https://github.com/theJaxon/anchore-engine/blob/main/etc/anchore-architecture.jpg)

#### How image scanning tools work:
- Image scanning tools extract the image file then looks for all available `packages` and `libraries`.
- The version of these packages and libraries is compared against the vulnerability DB.
- If any package version matches with any of the CVE descriptions in the DB then a vulnerability within the image is reported.

---

### :small_blue_diamond: Deploy anchore engine:
```bash
# Clone the repo 
git clone https://github.com/theJaxon/anchore-engine.git

# Apply the defined yaml files 
k apply -f anchore-engine/anchore/
```

:red_circle: for the persistent volume i rely on dynamic provisioning provided by [local-path-provisioner](https://github.com/rancher/local-path-provisioner)
```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```


### :small_blue_diamond: Install [anchore-cli](https://github.com/anchore/anchore-cli):
```bash
apt-get install python3-pip
pip3 install anchorecli

# Make ~/.local/bin part of the PATH or export it using
export PATH="$HOME/.local/bin/:$PATH"
```

### :small_blue_diamond: Interact with anchore using the CLI:
1. Get the IP address of the `api` service 
```bash
api_ip=http://$(k get svc -l app=api -ojsonpath='{.items[0].spec.clusterIP}'):8228
```
2. Add an image to the engine using the pre-defined credentials
```bash
anchore-cli --u admin --p foobar --url $api_ip image add ubuntu

# Wait for analysis to start 
anchore-cli image wait ubuntu

# Get image overview
anchore-cli image get ubuntu
```
