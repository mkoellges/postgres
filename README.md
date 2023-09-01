# PostgreSQL DB with crunchy operator

After installing the [Crunchy Postgres Operator](https://github.com/mkoellges/postgres-operator-examples), create a postgres Cluster:

```sh
kubectl apply -k k8s
```

The Deployment is created via kustomize in a new created namespace `databases` and a 3 instance cluster `test` is created incl Backup to Disk.
