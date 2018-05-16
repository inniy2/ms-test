## ms202 migration operation
This is the precedure for the migration.

## Disable all monitoring system on all target hosts
1. The new servers set.
DBQ dm-ms202z: https://dbq-mysql.pro.jpe2.rpaas.net/instances/2375
DBQ dm-ms202z: https://dbq-mysql.pro.jpe2.rpaas.net/instances/2375

## Check for DML transaction on the new master

## Enable Read-only ( NO WRITE ) on the old master 

## Stop slave on the new master 

## Disable Read-only ( WRITE ) on the new  master 

## Revoke privileges on the old server set

## Check the status of the new server set

## In case of rollback: grant privileges on the old server set

## Enable all monitoring system on all target hosts

