# PostgreSQL DB with crunchy operator

## Install the DB cluster

After installing the [Crunchy Postgres Operator](https://github.com/mkoellges/postgres-operator-examples), create a postgres Cluster:

```sh
kubectl apply -k k8s/database
```

The Deployment is created via kustomize in a new created namespace `databases` and a 3 instance cluster `hippo` is created incl Backup to Disk.

## Add role for monitorings

```bash
kubectl exec -it hippo-instance1-45mt-0 -- patronictl list

kubectl exec -it [MASTER_INSTANCE] -- psql
```
Here create the monitoring user:

```sql
CREATE ROLE ccp_monitoring WITH LOGIN;
ALTER ROLE ccp_monitoring WITH PASSWORD '[DECODED PASSWORD OF SECRET hippo-monitoring]';
ALTER ROLE ccp_monitoring WITH SUPERUSER;
```

Now install the monitoring for postgres DBs in this namespace:

```bash
kubectl apply -k k8s/monitoring‚
```

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

If you need to rerun the backup at the same day, you need to use the `--overwrite` option

```bash
kubectl annotate -n databases postgrescluster hippo --overwrite \
  postgres-operator.crunchydata.com/pgbackrest-backup="$( date '+%F_%H:%M:%S' )"
```

Check the backups:

```bash
kubectl exec -it hippo-repo-host-0 -- pgbackrest info
```

## Upgrade postgres to higher version

Create a  `PG_UPDATE_DEPLOYMENT_NAME` yaml manifest `pg_update.yaml` and apply it in the same namespace where the database is running:

```yaml
apiVersion: postgres-operator.crunchydata.com/v1beta1
kind: PGUpgrade
metadata:
  name: [PG_UPDATE_DEPLOYMENT_NAME]
spec:
  image: registry.developers.crunchydata.com/crunchydata/crunchy-upgrade:ubi8-5.5.0-0
  postgresClusterName: [PG_CLUSTER_NAME]
  fromPostgresVersion: [FRON_VERSION]
  toPostgresVersion: [TO_VERSION]
```

After this manifest is deployed you can check with

```bash
kubectl -n [NAMESPACE] get pgupgrades
```

Now annotate the instance to fulfil the update

```bash
kubectl -n pgo annotate postgrescluster [POSTGRES_CLUISTER_NAME] postgres-operator.crunchydata.com/allow-upgrade="[PG_UPDATE_DEPLOYMENT_NAME]
```

Now shutdown the database by changing the notation for `shutdown` in the postgres-cluster yaml manifest

```yaml
...
  spec:
    shutdown: true
...
```

Now the instance will be shutted down and the database will be upgraded. For this a couple of jobs run - you see completed pods at the end.

Now update the version in your postgres cluster manifest to the desired version and set `spec.shutdown` to `false` again so that the instance starts again.
