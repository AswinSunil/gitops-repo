We must Install the following in our laptop:

1. kind
2. Docker
3. kubectl
4. Argocd
________________________________________________________________________________

Installing kind:

1. sudo apt update
2. install curl - sudo apt install curl
3. install kind cluster: curl -Lo ./kind "https://kind.sigs.k8s.io/dl/v0.23.0 kind-linux-amd64"
4. give executbale permisison to kind: chmod +x ./kind
5. set kind as env var: sudo mv ./kind /usr/local/bin/kind
6. verify kind installtion: kind version
________________________________________________________________________________

Installing Docker:

1. sudo apt update
2. sudo apt install ca-certificates curl apt-transport-https software-properties-common -y
3. sudo install -m 0755 -d /etc/apt/keyrings
4. sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
5. sudo chmod a+r /etc/apt/keyrings/docker.asc
6. echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
7. sudo apt update
8. sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
9. docker --version

note:

Your user account does not have the necessary permissions to communicate with the Docker daemon. The Docker daemon runs as a root process, and its socket (/var/run/docker.sock) is typically owned by root and accessible only by root or members of the docker group.

The Solution: Add your user to the docker group.

sudo usermod -aG docker $USER

close your terminal.Log out of your current session and log back in using quick toggles, then open a new terminal and execute the below command:

newgrp docker
________________________________________________________________________________

Installing kubectl:

1. curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
2. curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
3. chmod +x ./kubectl
4. sudo mv ./kubectl /usr/local/bin/kubectl
5. kubectl version --client
________________________________________________________________________________

Installing Argocd:

Installing Argo CD isn't quite like installing a standalone application on Ubuntu. Argo CD is designed to run inside a Kubernetes cluster. So, the process involves two main parts:

Step:1 - Installing the Argo CD CLI (Command Line Interface) on your Ubuntu machine: This allows you to interact with your Argo CD instance from your terminal.
Step:2 - Deploying Argo CD to your Kubernetes cluster: This is where the actual Argo CD server, controller, and other components run.
Step:3 - start argocd service
step:4 - Normally we use kubectl apply deployment, service, to start our application in our repo. This is a manual, imperative, non-gitops way to deploy resources directly to your cluster. Argo CD has no knowledge of this manual deployment.

Instead we must create an an Application Custom Resource within Argo CD itself that tells Argo CD to monitor our gitops-repo for your application's deployment, service YAML files and synchronize them to the cluster.

We can create the Application Custom Resource using two ways either by using a yaml file or by using the argocd web UI; Since yaml file is IAC we choose that method.

step:1

1. ARGOCD_VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')
echo "Latest Argo CD CLI version: $ARGOCD_VERSION"
2. curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/download/${ARGOCD_VERSION}/argocd-linux-amd64
3. chmod +x argocd-linux-amd64
4. sudo mv argocd-linux-amd64 /usr/local/bin/argocd
5. argocd version --client

step:2

1. create a kind cluster: kind create cluster --name task-cluster
2. inside the above cluster create new namespace for argocd: kubectl create namespace argocd
3. install argocd inside the above namespace as pods: kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
4. kubectl get pods -n argocd -w   Wait until all pods show Running and READY. Then Press Ctrl+C to exit the watch.

step:3

Open a new terminal window for this, as it will block the terminal while running.
kubectl port-forward svc/argocd-server -n argocd --address 0.0.0.0 8080:443

step:4

Access the argocd web ui using the below address in your browser. Note: this address will work only when the above service is running so do not close the above terminal.
https://localhost:8080

The default username of argocd is admin
TO know the password use the command: kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

use the username and password to login into argocd web ui. After logging in we can see that Application Custom Resource is not created; we can create it here or by using yaml file.

Application custom resource created using yaml file:

1. The argo-app.yaml file must be placed in the root directory of gitops-repo
2. apply the argo-app.yaml file in the argocd namespace: kubectl apply -f argo-app.yaml -n argocd
3. Now see the argocd web Ui an application custom resource would have been created.
4. If you click the application you can see the application custom resource made argocd to apply deployments, services, etc. (our application) in our gitops-repo automatically.
________________________________________________________________________________

Accessing our application:

In service.yaml file we configured that our application will be available at port 3000; for http://localhost:3000 to work, you must have the below command running in a separate terminal window:
kubectl port-forward service/task-manager 3000:80

Now access the application at http://localhost:3000 no task will be present because we still did not add any task to our task manager application.

run this command to add a task curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"title": "Finish resume project", "completed": false}' \
     http://localhost:3000/tasks
________________________________________________________________________________