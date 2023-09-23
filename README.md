# PostgreSQL DB with crunchy operator

## Install the DB cluster

After installing the [Crunchy Postgres Operator](https://github.com/mkoellges/postgres-operator-examples), create a postgres Cluster:

```sh
kubectl apply -k k8s
```

The Deployment is created via kustomize in a new created namespace `databases` and a 3 instance cluster `hippo` is created incl Backup to Disk.

## Create a small table for testing

For testing purpose, create a small table:

```sql
create table test (id int, name char(32));

insert into test (id, name)
values
(1,'Manni'),
(2,'Henry'),
(3,'Florian'),
(4, 'Dirk'),
(5,'Markus'),
(6,'Björn');
```

## Create a backup of the DB

Backups can be done on demand:

```bash
kubectl annotate -n databases postgrescluster hippo \
  postgres-operator.crunchydata.com/pgbackrest-backup="$( date '+%F_%H:%M:%S' )"
```

Check the backups:

```bash
kubectl exec -it hippo-repo-host-0 -- pgbackrest info
```
