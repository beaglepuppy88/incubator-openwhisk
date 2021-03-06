# Mesos Support

The `MesosContainerFactory` enables launching action containers within a Mesos cluster. It does not affect the deployment of OpenWhisk components (invoker, controller).

## Enable

To enable MesosContainerFactory, use the following TypeSafe Config properties

| property | required | details | example |
| --- | --- | --- | --- |
| `whisk.spi.ContainerFactoryProvider` | required | enable the MesosContainerFactory | whisk.core.mesos.MesosContainerFactoryProvider |
| `whisk.mesos.master-url` | required | Mesos master HTTP endpoint to be accessed from the invoker for framework subscription | http://192.168.99.100:5050 |
| `whisk.mesos.master-url-public` | optional (default to whisk.mesos.master-url) | public facing Mesos master HTTP endpoint for exposing logs to cli users | http://192.168.99.100:5050 |
| `whisk.mesos.role` | optional (default *) | Mesos framework role| any string e.g. `openwhisk` |
| `whisk.mesos.failover-timeout-seconds` | optional (default 0) | how long to wait for the framework to reconnect with the same id before tasks are terminated  | see http://mesos.apache.org/documentation/latest/high-availability-framework-guide/ |
| `whisk.mesos.mesos-link-log-message` | optional (default true) | display a log message with a link to Mesos when using the default LogStore (or no log message) | Since logs are not available for invoker to collect from Mesos in general, you can either use an alternate LogStore or direct users to the Mesos ui |   |
| `whisk.mesos.constraints` | optional (default []) | placement constraint strings to use for managed containers | `["att1 LIKE v1", "att2 UNLIKE v2"]` |   |
| `whisk.mesos.blackbox-constraints` | optional (default []) | placement constraint strings to use for blackbox containers  | `["att1 LIKE v1", "att2 UNLIKE v2"]` |   |
| `whisk.mesos.constraint-delimiter` | optional (default " ") | delimiter used to parse constraints |  |   |
| `whisk.mesos.teardown-on-exit` | optional (default true) | set to true to disable the mesos framework on system exit; set for false for HA deployments |  |   |

To set these properties for your invoker, set the corresponding environment variables e.g.,
```properties
CONFIG_whisk_spi_ContainerFactoryProvider=whisk.core.mesos.MesosContainerFactoryProvider
CONFIG_whisk_mesos_masterUrl=http://192.168.99.100:5050
```

## Known Issues

* Logs are not collected from action containers.

  For now, the Mesos public URL will be included in the logs retrieved via the wsk CLI. Once log retrieval from external sources is enabled, logs from mesos containers would have to be routed to the external source, and then retrieved from that source.
 
* No HA or failover support (single invoker per cluster).
  
  Currently the version of `mesos-actor` in use does not support HA or failover. Failover support is planned to be provided by:
  
  * multiple invokers running in an Akka cluster
  * the Mesos framework actor is a singleton within the cluster
  * the Mesos framework actor is available from the other invoker nodes
  * if the node that contains the Mesos framework actor fails:
     * the actor will be recreated on a separate invoker node
     * the actor will resubscribe to mesos scheduler API with the same ID
     * the tasks that were previously launched by the actor will be reconciled
     * normal operation resumes
     
     
  



