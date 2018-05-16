## ms202 migration operation
This is the precedure for the migration.

## Disable all monitoring system on all target hosts
1. The new servers set.

DBQ dm-ms202z: https://dbq-mysql.pro.jpe2.rpaas.net/instances/2375

DBQ ds-ms202-201z: https://dbq-mysql.pro.jpe2.rpaas.net/instances/2371

DBQ ds-ms202-199zz: https://dbq-mysql.pro.jpe2.rpaas.net/instances/2372

Almond dm-ms202z: https://almond.pro.jpe2.rpaas.net/instances/1896?alive=true

Almond ds-ms202-201z: https://almond.pro.jpe2.rpaas.net/instances/1890?alive=true

Almond ds-ms202-199z: https://almond.pro.jpe2.rpaas.net/instances/1891?alive=true

Pandora dm-ms202z: https://rmonitor.rakuten-it.com/monitor/PandoraSearch.cgi?search_module=dm-ms202z.prod.jp.local&search_group=&search_agent=&search_alert=ALL&search_status=ALL&pandora_select=&btnN=RMonitor%0D%0ASearch

Pandora ds-ms202-201z: https://almond.pro.jpe2.rpaas.net/instances/1891?alive=tru://rmonitor.rakuten-it.com/monitor/PandoraSearch.cgi?search_module=ds-ms202-201z.prod.jp.local&search_group=&search_agent=&search_alert=ALL&search_status=ALL&pandora_select=&btnN=RMonitor%0D%0ASearch 

Pandora ds-ms202-199z: https://rmonitor.rakuten-it.com/monitor/PandoraSearch.cgi?search_module=ds-ms202-199z.prod.jp.local&search_group=&search_agent=&search_alert=ALL&search_status=ALL&pandora_select=&btnN=RMonitor%0D%0ASearch

## Check for DML transaction on the new master


## Check for DML transaction on the new master

## Check for DML transaction on the new master

## Enable Read-only ( NO WRITE ) on the old master 

## Stop slave on the new master 

## Disable Read-only ( WRITE ) on the new  master 

## Revoke privileges on the old server set

## Check the status of the new server set

## In case of rollback: grant privileges on the old server set

## Enable all monitoring system on all target hosts

