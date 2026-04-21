# MonsterMQ Docker Swarm Deployment

A production-ready 3-node MonsterMQ cluster with HAProxy load balancing on Docker Swarm.

## Architecture

- **3x MonsterMQ** replicas with Hazelcast clustering (TCP-IP discovery via Swarm DNS)
- **HAProxy** load balancer for MQTT (1883) and GraphQL/Dashboard (4000)
- **Overlay network** for inter-node communication
- Rolling updates with automatic rollback

## Prerequisites

1. Docker Swarm initialized with 3+ manager nodes
2. An external `proxy` overlay network for ingress
3. PostgreSQL database accessible from the Swarm nodes

## Deployment

### 1. Create Docker configs

```bash
# MonsterMQ broker config (edit monster.yml for your DB, ports, etc.)
docker config create monstermq_config monster.yml

# Hazelcast cluster discovery (edit member hostnames to match your Swarm nodes)
docker config create hazelcast hazelcast.xml

# HAProxy load balancer (edit server hostnames to match your Swarm nodes)
docker config create haproxy haproxy.cfg
```

### 2. Deploy the stack

```bash
docker stack deploy -c docker-compose.yml monstermq
```

### 3. Verify

```bash
docker service ls                          # Check services are running
docker service logs monstermq_monstermq    # Broker logs
docker service logs monstermq_proxy        # HAProxy logs
```

## Customization

### Node hostnames

MonsterMQ replicas get hostnames in the format `<swarm-node-hostname>-monstermq` (configured via `{{.Node.Hostname}}-monstermq` in the compose file). Update these in:

- `hazelcast.xml` — `<member>` entries for cluster discovery
- `haproxy.cfg` — `server` entries for load balancing backends

### Ports

| Service | Port | Protocol |
|---------|------|----------|
| MQTT    | 1883 | TCP      |
| GraphQL + Dashboard | 4000 | HTTP |
| Hazelcast (internal) | 5701 | TCP |

### Scaling

The default setup runs 3 replicas with `max_replicas_per_node: 1`. To change the cluster size, update `replicas` in docker-compose.yml and add/remove corresponding entries in hazelcast.xml and haproxy.cfg.
