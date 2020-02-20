binary := azurekeyvault-flexvolume
REGISTRY_NAME ?= upstreamk8sci
REGISTRY ?= $(REGISTRY_NAME).azurecr.io
DOCKER_IMAGE ?= $(REGISTRY)/public/k8s/flexvolume/keyvault-flexvolume
VERSION          := v0.0.17

.PHONY: build
build: authors deps
	@echo "Building..."
	$Q GOOS=linux CGO_ENABLED=0 go build .
	$Q mv $(binary) ../deployment/flexvol-installer/

image: build
	@echo "Building docker image..."
	$Q docker build -t $(DOCKER_IMAGE):$(VERSION) ../deployment/flexvol-installer

.PHONY: clean deps

deps: setup
	@echo "Ensuring Dependencies..."
	$Q go env
	$Q dep ensure

clean:
	@echo "Clean..."
	$Q rm -rf $(binary)
	$Q rm -rf ../deployment/flexvol-installer/$(binary)

setup: clean
	@echo "Setup..."
	go get -u github.com/golang/dep/cmd/dep

authors:
	$Q git log --all --format='%aN <%cE>' | sort -u  | sed -n '/github/!p' > GITAUTHORS
	$Q cat AUTHORS GITAUTHORS  | sort -u > NEWAUTHORS
	$Q mv NEWAUTHORS AUTHORS
	$Q rm -f NEWAUTHORS
	$Q rm -f GITAUTHORS

KIND_VERSION ?= 0.5.1
KUBERNETES_VERSION ?= 1.15.3
E2E_IMAGE_VERSION = v0.0.16-e2e-$$(git rev-parse --short HEAD)

.PHONY: e2e-bootstrap
e2e-bootstrap:
	# Download and install kubectl
	curl -LO https://storage.googleapis.com/kubernetes-release/release/v${KUBERNETES_VERSION}/bin/linux/amd64/kubectl && chmod +x ./kubectl && sudo mv kubectl /usr/local/bin/
	# Download and install kind
	curl -L https://github.com/kubernetes-sigs/kind/releases/download/v${KIND_VERSION}/kind-linux-amd64 --output kind && chmod +x kind && sudo mv kind /usr/local/bin/
	# Create kind cluster
	kind create cluster --image kindest/node:v${KUBERNETES_VERSION}
	# Build image
	REGISTRY_NAME="e2e" IMAGE_VERSION=${E2E_IMAGE_VERSION} make image
	# Load image into kind cluster
	kind load docker-image --name kind e2e/azurekeyvault-flexvolume:${E2E_IMAGE_VERSION}
	# Set up tiller
	kubectl --namespace kube-system --output yaml create serviceaccount tiller --dry-run | kubectl --kubeconfig $$(kind get kubeconfig-path)  apply -f -
	kubectl create --output yaml clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller --dry-run | kubectl --kubeconfig $$(kind get kubeconfig-path) apply -f -

.PHONY: e2e-azure
e2e-azure:
	bats -t ../test/bats/azure.bats
