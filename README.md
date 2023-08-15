# PostgreSQL DB with crunchy operator

## Install on Kubernetes

Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.

```sh
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.25.0/install.sh | bash -s v0.25.0
```

## Install the operator by running the following command:
What happens when I execute this command?

```sh
kubectl create -f https://operatorhub.io/install/postgresql.yaml
```

This Operator will be installed in the "operators" namespace and will be usable from all namespaces in the cluster.

## After install, watch your operator come up using next command.

```sh
kubectl get csv -n operators
```

To use it, checkout the custom resource definitions (CRDs) introduced by this operator to start using it.

## Install Postgres datbaase

Install the database using crunchy

```sh
kubectl apply -k k8s/
```