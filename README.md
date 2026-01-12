# Proyecto_Final_BigDataAplicado_GrupoAzul


## Roles por nodo

| Nodo | IP | Roles principales |
| :-- | :-- | :-- |
| ambari10 | 172.16.200.10 | NameNode HDFS, Kafka Broker, Grafana, WebHCat, Spark History, Zeppelin, Metrics Monitor |
| ambari11 | 172.16.200.11 | ResourceManager YARN, History Server MapReduce2, ZooKeeper Server, Metrics Monitor |
| ambari12 | 172.16.200.12 | HiveServer2, Hive Metastore, MySQL (Hive), ZooKeeper Server, Active HBase Master, Metrics Monitor |
| ambari13 | 172.16.200.13 | Metrics Collector (Ambari Metrics Master), DataNode, NodeManager, ZooKeeper Client, varios clientes |
| ambari15 | 172.16.200.15 | DataNode, NodeManager, Metrics Monitor |
| ambari9 | 172.16.200.9 | DataNode, NodeManager, Spark Thrift Server, RegionServer HBase, Metrics Monitor, varios clientes |
| ambari8 | 172.16.200.8 | DataNode, NodeManager, Metrics Monitor |
| ambari5 | 172.16.200.5 | DataNode, NodeManager, Metrics Monitor |
| ambari3 | 172.16.200.3 | DataNode, NodeManager, ZooKeeper Client, varios clientes |
| VM Redis | 172.16.200.23 | Servidor Redis ejecutándose en una VM sobre el PC físico que aloja ambari3 (servicio auxiliar externo al clúster Hadoop). |
| VM HA Debian | 172.30.101.216 | Debian con Home Assistant sobre Docker en el PC físico que aloja ambari5, para automatización doméstica y contenedores adicionales. |
​


## Relaciones entre nodos (visión lógica)

- **HDFS**
    - NameNode en `ambari10` coordina los **DataNode** en `ambari3`, `5`, `8`, `9`, `13`, `15` (y cualquiera adicional).
    - Los clientes HDFS de los demás nodos hablan con el NameNode para metadata y con los DataNode para bloques.
- **YARN / MapReduce2 / Spark**
    - ResourceManager en `ambari11` coordina los **NodeManager** de `ambari3`, `5`, `8`, `9`, `13`, `15`.
    - HistoryServer (MapReduce2) en `ambari11` recibe los logs de jobs.
    - Spark jobs usan YARN y HDFS: ejecutores en todos los nodos con NodeManager y datos en DataNodes.
- **Ambari Metrics**
    - **Metrics Collector** (master) está en `ambari13`.
    - Todos los **Metrics Monitor** (en `5`, `8`, `9`, `10`, `11`, `12`, `13`, `15`…) le envían métricas al collector.
    - Grafana en `ambari10` lee del Metrics Collector para las gráficas.
- **HBase**
    - Active HBase Master en `ambari12`.
    - RegionServer en `ambari9` (y cualquier otro que configures) guardan datos en los mismos DataNode de HDFS.
    - HBase Client en varios nodos (3, 9, 10, 11, 12…) se conectan al Master y a los RegionServer.
- **Hive**
    - HiveServer2 y Metastore en `ambari12`, con MySQL como backend de Metastore en el mismo nodo.
    - Los clientes Hive de otros nodos se conectan a HiveServer2, que a su vez usa HDFS (NameNode/DataNodes) y el Metastore/MySQL.
- **ZooKeeper**
    - ZooKeeper Servers en `ambari10`, `11`, `12` forman el ensemble.
    - Los ZooKeeper Clients (por ejemplo en `ambari3`, `13`, y otros componentes como Kafka, HBase, YARN) usan el ensemble para coordinación.
- **Kafka**
    - Broker Kafka en `ambari10`; productores y consumidores en el resto de nodos se conectan a este broker (y al ensemble de ZooKeeper).
