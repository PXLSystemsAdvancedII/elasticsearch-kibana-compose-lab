# Elasticsearch Cluster Lab

## What is Elasticsearch?

Elasticsearch is a distributed, open-source search and analytics engine built on Apache Lucene. It enables:
- **Full-text search**: Fast, scalable search across large datasets
- **Real-time analytics**: Aggregate and analyze data in real-time
- **Distributed architecture**: Automatically distributes data across multiple nodes
- **RESTful APIs**: JSON-based API for all operations

Key components:
- **Indices**: Collections of related documents (like database tables)
- **Shards**: Pieces of an index distributed across nodes
- **Replicas**: Copies of shards for fault tolerance

## Lab Architecture

This lab sets up a 3-node Elasticsearch cluster with Kibana for visualization:

```
                     ┌─────────────┐
                     │   Kibana    │ (Port 5601)
                     │ Dashboard   │
                     └─────┬───────┘
                           │
         ┌─────────────────┴─────────────────┐
         │                                   │
    ┌────▼────┐          ┌─────────┐    ┌────▼────┐
    │  es01   │◄────────▶│  es02   │◄──▶│  es03   │
    │ Master  │          │ Master  │    │ Master  │
    │ Data    │          │ Data    │    │ Data    │
    └─────────┘          └─────────┘    └─────────┘
     Port 9200           Port 9201      Port 9202
```

Each Elasticsearch node:
- Acts as both master-eligible and data node
- Can handle HTTP requests
- Communicates with other nodes for clustering
- Stores data in mounted volumes

This lab demonstrates Elasticsearch cluster concepts including quorum, distributed search, and high availability.

## Lab Setup

### Starting the Environment

```bash
docker-compose up -d
```

### Verifying the Cluster

Check cluster status:
```bash
curl http://localhost:9200/_cluster/health?pretty
```

## Lab Exercises

### 1. Understanding Quorum

**Learn about quorum and master node election**

Check cluster state:
```bash
curl http://localhost:9200/_cluster/state?pretty
```

Identify the master node:
```bash
curl http://localhost:9200/_cluster/state/master_node?pretty
```

**Simulate node failure:**
```bash
docker-compose stop es03
```

Observe cluster behavior:
```bash
curl http://localhost:9200/_cluster/health?pretty
```

**Test quorum:**
```bash
docker-compose stop es02
```

Observe how the cluster handles losing quorum (majority of nodes).

Restart nodes:
```bash
docker-compose start es02 es03
```

### 2. Data Distribution

Create an index with multiple shards:
```bash
curl -X PUT "localhost:9200/distributed-index?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}'
```

View shard allocation:
```bash
curl http://localhost:9200/_cat/shards/distributed-index?v
```

### 3. Kibana Dashboard - Monitoring Cluster and Quorum

**Monitor cluster status:**
1. Open Kibana at http://localhost:5601
2. Navigate to **Stack Monitoring** (or click the monitoring icon ⚡)
3. Click on **Nodes** to see all cluster nodes
4. Observe node states (green = healthy, yellow = warning, red = critical)

**Visualize quorum effects:**
1. Open Stack Monitoring → Cluster Overview
2. Stop a node: `docker-compose stop es03`
3. Watch the dashboard show:
   - Node count changes
   - Cluster health transitions from green to yellow
   - Master node election (if master node stopped)
4. Stop another node: `docker-compose stop es02`
5. Observe:
   - Cluster status turns red
   - "No master" message appears
   - API requests fail due to lack of quorum

**Create real-time monitoring:**
1. Navigate to Stack Management → Saved Objects
2. Import cluster health visualizations
3. Create a dashboard showing:
   - Node status
   - Master node identity
   - Cluster health timeline

### 4. Advanced Concepts

**Rolling restarts:**
```bash
docker-compose restart es03
# Wait for green status
docker-compose restart es02
# Wait for green status
docker-compose restart es01
```

**Cluster settings:**
```bash
curl http://localhost:9200/_cluster/settings?pretty
```

**Node stats:**
```bash
curl http://localhost:9200/_nodes/stats?pretty
```

## Key Learning Points

1. **Quorum**: Elasticsearch requires a majority of master-eligible nodes to elect a master
2. **Fault Tolerance**: The cluster can survive the loss of one node
3. **Data Replication**: Data is replicated across nodes for availability
4. **Shard Distribution**: Elasticsearch automatically distributes shards across nodes

## Cleanup

```bash
docker-compose down -v
```

## Troubleshooting

If containers fail to start, check:
- Docker system resources
- Port conflicts (9200-9202, 5601)
- Memory constraints

Check container logs:
```bash
docker-compose logs -f es01
```