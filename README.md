# Operator-101
A Kubernetes Operator tutorial based on Kubebuilder.

It will take you building to a full-featured cronjob controller step by step:
![](https://github.com/snlndod/mPOST/blob/master/Operator-101/00.png)

## Installation
There are some prerequisites that need to be installed in advance.

### Go
Extract the [archive](https://golang.org/dl/) we downloaded into /usr/local, creating a Go tree in /usr/local/go:
```
$ rm -rf /usr/local/go && tar -C /usr/local -xzf go1.15.9.linux-amd64.tar.gz
```

Add /usr/local/go/bin to the PATH environment variable in **/etc/profile**:
```
$ export PATH=$PATH:/usr/local/go/bin
$ export PATH=$PATH:$HOME/go/bin
$ export GO111MODULE=on
```

Verify that Go has installed successfully:
```
$ go version
go version go1.15.9 linux/amd64
```

### Docker
Install the docker package:
```
$ sudo zypper install docker
```

To automatically start the Docker service at boot time:
```
$ sudo systemctl enable docker.service
```

Start the Docker service:
```
$ sudo systemctl start docker.service
```

To allow a certain user to connect to the local Docker daemon, use the following command:
```
$ sudo /usr/sbin/usermod -aG docker i516697
```

### minikube
Download and install the latest minikube:
```
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Start our cluster:
```
$ minikube start --driver=docker --kubernetes-version=v1.17.0
```

Create a symbolic link to minikube's binary named _kubectl_:
```
$ ln -s $(which minikube) /usr/local/bin/kubectl
```

### kustomize
The following script detects our OS and downloads the appropriate kustomize binary to our current working directory:
```
$ curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```

### controller-gen
This will install controller-gen binary in **GOPATH/bin**:
```
$ go get sigs.k8s.io/controller-tools/cmd/controller-gen@v0.2.5
```

### Kubebuilder
Download kubebuilder and install locally:
```
$ os=$(go env GOOS)
$ arch=$(go env GOARCH)
$ curl -L https://go.kubebuilder.io/dl/2.3.2/${os}/${arch} | tar -xz -C /tmp/
$ sudo mv /tmp/kubebuilder_2.3.2_${os}_${arch}/* /usr/local/kubebuilder
$ export PATH=$PATH:/usr/local/kubebuilder/bin
```

## Usage
It covers initialization, development, deployment and testing.

### Initialization 
1. Tell Go and kubebuilder the base import path of our module:
    ```
    $ go mod init github.com/snlndod/Operator-101
    ```
2. Scaffold out a new project:
    ```
    $ kubebuilder init --domain github.com
    ```
3. Add a new API:
    ```
    $ kubebuilder create api --group batch --version v1 --kind CronJob
    ```

### Development
1. [Design CronJob API](https://github.com/snlndod/Operator-101/blob/main/api/v1/cronjob_types.go);
2. [Implement CronJob Controller](https://github.com/snlndod/Operator-101/blob/main/controllers/cronjob_controller.go);

### Deployment
1. Install CRDs and run the controller locally:
    ```
    $ make install
    $ make run
    ```
2. Install instances of CRs:
    ```
    $ kubectl apply -f config/samples/batch_v1_cronjob.yaml
    ```
3. Build/Push our image and deploy the controller to cluster:
    ```
    $ make docker-build docker-push IMG=snlndod/cronjob:latest
    $ make deploy IMG=snlndod/cronjob:latest
    ```

### Testing
1. [Setup Test Environment](https://github.com/snlndod/Operator-101/blob/main/controllers/suite_test.go);
2. [Test Controller's Behavior](https://github.com/snlndod/Operator-101/blob/main/controllers/suite_test.go);

## Miscellaneous
Execute integration tests and compile:
```
$ make test
$ make
```

Check cronjob's running/updating status:
```
$ kubectl get cronjob.batch.github.com cronjob-sample -o yaml
$ kubectl get job
```

Delete instances of CRs:
```
$ kubectl delete cronjob.batch.github.com cronjob-sample
```

Check/Delete namespace after deployment:
```
$ kubectl get namespace
$ kubectl delete namespace operator-101-system
```

Delete CRDs from a cluster:
```
$ make uninstall
```

## Contributing
We love contributions! Before submitting a Pull Request, it's always good to start with a new issue first.

## License
This repository is licensed under Apache 2.0. Full license text is available in [LICENSE](https://github.com/snlndod/Operator-101/blob/main/LICENSE).
