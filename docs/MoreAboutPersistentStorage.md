# Persistent Stores used in Open Cluster Management Observability

Open Cluster Management Observability is a stateful application. It creates following persistent volumes (the number of copies depend on replication factor set).

### List of Persistent Volumes

| Name | Purpose |
| ----------- | ----------- |
| alertmanager | Alertmanager stores the nflog data and silenced alerts in its storage (nflog is append-only log of active/resolved notifications along with the notified receiver, and a hash digest of the notification's identifying contents) .|
| thanos-compact | The compactor needs local disk space to store intermediate data for its processing, as well as bucket state cache. The required space depends on the size of the underlying blocks. The compactor must have enough space to download all of the source blocks, then build the compacted blocks on the disk. On-disk data is safe to delete between restarts and should be the first attempt to get crash-looping compactors unstuck. However, it is recommended to give the compactor persistent disks in order to effectively use bucket state cache in between restarts. |
| thanos-rule |The thanos ruler evaluates Prometheus recording and alerting rules against a chosen query API by issuing queries at a fixed interval. Rule results are written back to the disk in the Prometheus 2.0 storage format. Rule results are written back to disk in the Prometheus 2.0 storage format. The amount of hours or days of data retained in this stateful set was fixed in API Version `observability.open-cluster-management.io/v1beta1`. It has been exposed as an API parameter in `observability.open-cluster-management.io/v1beta2`: _RetentionInLocal_ |
| thanos-receive-default | Thanos receiver accepts incoming data (Prometheus remote-write requests) and writes these into a local instance of the Prometheus TSDB. Periodically (every 2 hours), TSDB blocks are uploaded to the object storage for long term storage and compaction. The amount of hours or days of data retained in this stateful set, which acts a local cache was fixed in API Version `observability.open-cluster-management.io/v1beta`. It has been exposed as an API parameter in `observability.open-cluster-management.io/v1beta2`: _RetentionInLocal_ |
| thanos-store-shard| It acts primarily as an API gateway and therefore does not need significant amounts of local disk space. It joins a Thanos cluster on startup and advertises the data it can access. It keeps a small amount of information about all remote blocks on local disk and keeps it in sync with the bucket. This data is generally safe to delete across restarts at the cost of increased startup times. |





### Configuring the Stateful sets

In Mutlicluster Observability Operator API: `observability.open-cluster-management.io/v1beta1` following was exposed:

```
    //defaults shown below
    statefulSetSize: 10Gi
    statefulSetStorageClass: gp2
```
Taking into consideration this can result in wasted space, we have allowed settings to be individually tunable in the next revision:  `observability.open-cluster-management.io/v1beta2`

```
    //defaults shown below
    StorageClass: gp2
    AlertmanagerStorageSize: 1Gi 
    RuleStorageSize: 1Gi
    CompactStorageSize: 100 Gi
    ReceiveStorageSize: 100 Gi
    StoreStorageSize: 10 Gi

```
**Note**: The default storage class, as configured in the system, is used for configuring the persistent volumes automatically unless a different storage class is specified in the custom resource specification. If no storage class exists, for example in default OpenShift bare metal installations, a storage class must be created and specified or the installation of observability fails.
<br>
<br>

## Object Store
In addition to the persistent volumes previously mentioned, the time series historical data is stored in object stores. Thanos uses object storage as the primary storage for metrics and metadata related to them. Details about the object storage and downsampling will be provided in another document.