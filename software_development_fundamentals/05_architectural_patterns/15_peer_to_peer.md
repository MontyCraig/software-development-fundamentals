# Peer-to-Peer (P2P) Architecture

## Overview and Intent

Peer-to-Peer architecture is a decentralized model where every node in the
network acts as both a client and a server. There is no central authority
controlling the system; instead, peers collaborate directly to share resources,
data, and processing power. Each peer is equally privileged and can initiate
or respond to requests from any other peer.

The intent is to eliminate single points of failure and centralized bottlenecks.
By distributing both data and processing across all participants, P2P systems
can achieve extreme resilience, scalability, and resource efficiency. The system
grows stronger as more peers join, since each new peer contributes both demand
and capacity.

P2P architectures power some of the largest distributed systems in existence:
file sharing networks, blockchain systems, content delivery networks, and
decentralized communication platforms. The trade-off is complexity -- managing
consistency, discovery, and coordination without a central authority requires
sophisticated protocols.

## Structure and Components

### Component Diagram

```
         +--------+          +--------+
         | Peer A |<-------->| Peer B |
         |        |          |        |
         | +----+ |          | +----+ |
         | |Data| |          | |Data| |
         | +----+ |          | +----+ |
         +---+----+          +----+---+
             |   \              /  |
             |    \            /   |
             |     \          /    |
             |      \        /     |
         +---+----+  +------+---+  |
         | Peer C |  | Peer D   |  |
         |        |<>|          |<-+
         | +----+ |  | +----+  |
         | |Data| |  | |Data|  |
         | +----+ |  | +----+  |
         +---+----+  +----+---+
             |              |
             |   +--------+ |
             +-->| Peer E |<+
                 |        |
                 | +----+ |
                 | |Data| |
                 | +----+ |
                 +--------+

  Every node is both client AND server.
  No central authority. Direct peer connections.
  
  Discovery via:
  +-------------------+
  | DHT (Distributed  |   Peers find each other through
  | Hash Table)       |   distributed lookup protocols
  +-------------------+
```

### Key Components

1. **Peer / Node**: An individual participant that stores data, processes
   requests, and communicates with other peers. Each peer has equal
   capabilities and responsibilities.

2. **Overlay Network**: The logical network formed by peer connections on
   top of the physical network. Defines how peers find and communicate
   with each other.

3. **Distributed Hash Table (DHT)**: A decentralized lookup protocol that
   maps data keys to responsible peers, enabling efficient data location
   without a central directory.

4. **Gossip Protocol**: A communication pattern where peers periodically
   exchange state information with random neighbors, eventually
   propagating information to all peers.

5. **Data Replication Manager**: Ensures data durability by replicating
   chunks across multiple peers, with configurable redundancy levels.

## How It Works

```
  Peer A wants to find data "X"
       |
       v
  +----+------+
  | Peer A    |  1. Hash the key "X" to determine responsible peer
  | (Requester)|  2. Query DHT overlay network
  +----+------+
       |
       | DHT lookup (may hop through multiple peers)
       |
  +----+------+     +----------+     +----------+
  | Peer B    |---->| Peer D   |---->| Peer F   |
  | (partial  |     | (partial |     | (has data|
  |  routing) |     |  routing)|     |  for "X")|
  +----------+     +----------+     +----+------+
                                         |
                                         | 3. Direct transfer
                                         v
                                    +----+------+
                                    | Peer A    |  4. Receives data "X"
                                    | (Requester)|  5. Caches locally
                                    +----------+  6. Becomes another source

  Gossip Protocol (State Propagation):

  Round 1:  A tells B     (A knows, B knows)
  Round 2:  A tells D, B tells C  (A,B,C,D know)
  Round 3:  All tell random peers  (convergence)
```

## Pseudocode Example

```
// === PEER NODE ===

STRUCTURE PeerNode:
    id: String
    address: NetworkAddress
    dataStore: LocalDataStore
    routingTable: RoutingTable
    connections: Map of String to PeerConnection
    replicationFactor: Integer
END STRUCTURE

FUNCTION PeerNode.Start(bootstrapPeers: List of NetworkAddress):
    // Join the network through known bootstrap peers
    FOR EACH address IN bootstrapPeers DO
        SET peer = this.Connect(address)
        IF peer IS NOT NULL THEN
            this.routingTable.Add(peer)
            // Request routing table entries from bootstrap peer
            SET neighbors = peer.GetNeighbors()
            FOR EACH neighbor IN neighbors DO
                this.routingTable.Add(neighbor)
            END FOR
            BREAK  // One successful bootstrap is enough
        END IF
    END FOR

    // Start background processes
    StartAsync(this.GossipLoop)
    StartAsync(this.HealthCheckLoop)
    StartAsync(this.ReplicationMaintenanceLoop)

    LOG "Peer " + this.id + " joined network with " +
        this.routingTable.Size() + " known peers"
END FUNCTION

// === DHT LOOKUP ===

FUNCTION PeerNode.FindValue(key: String) -> DataResult:
    // Check local store first
    SET localValue = this.dataStore.Get(key)
    IF localValue IS NOT NULL THEN
        RETURN DataResult.Found(localValue, source: this.id)
    END IF

    // Determine which peer should own this key
    SET targetId = HashToId(key)

    // Iterative lookup through the DHT
    SET closestPeers = this.routingTable.FindClosest(targetId, count: 3)
    SET queried = NEW Set

    LOOP
        SET improved = FALSE
        FOR EACH peer IN closestPeers DO
            IF queried.Contains(peer.id) THEN CONTINUE END IF
            queried.Add(peer.id)

            SET response = peer.Query(key)
            IF response.hasValue THEN
                // Cache locally for future requests
                this.dataStore.Put(key, response.value, ttl: 3600)
                RETURN DataResult.Found(response.value, source: peer.id)
            END IF

            // Response contains peers closer to the target
            FOR EACH closerPeer IN response.closerPeers DO
                IF NOT queried.Contains(closerPeer.id) THEN
                    Append(closestPeers, closerPeer)
                    SET improved = TRUE
                END IF
            END FOR
        END FOR

        IF NOT improved THEN BREAK END IF
    END LOOP

    RETURN DataResult.NotFound()
END FUNCTION

// === DATA STORAGE AND REPLICATION ===

FUNCTION PeerNode.StoreValue(key: String, value: Data) -> StoreResult:
    // Store locally
    this.dataStore.Put(key, value)

    // Replicate to peers closest to the key's hash
    SET targetId = HashToId(key)
    SET replicaPeers = this.routingTable.FindClosest(targetId, count: this.replicationFactor)

    SET successCount = 1  // Local store counts as one
    FOR EACH peer IN replicaPeers DO
        IF peer.id = this.id THEN CONTINUE END IF
        TRY
            peer.Replicate(key, value)
            SET successCount = successCount + 1
        CATCH error
            LOG "Replication to " + peer.id + " failed: " + error.message
        END TRY
    END FOR

    IF successCount >= (this.replicationFactor / 2) + 1 THEN
        RETURN StoreResult.Success(replicas: successCount)
    ELSE
        RETURN StoreResult.PartialFailure(replicas: successCount,
            required: (this.replicationFactor / 2) + 1)
    END IF
END FUNCTION

// === GOSSIP PROTOCOL ===

FUNCTION PeerNode.GossipLoop():
    LOOP FOREVER DO
        Sleep(Duration.Seconds(5))

        // Select random peers to gossip with
        SET targets = this.routingTable.SelectRandom(count: 3)

        SET myState = {
            peerId: this.id,
            timestamp: Now(),
            dataCount: this.dataStore.Size(),
            load: this.GetCurrentLoad(),
            version: this.stateVersion
        }

        FOR EACH target IN targets DO
            TRY
                SET theirState = target.ExchangeGossip(myState)

                // Merge their knowledge with ours
                this.MergeGossipState(theirState)

                // Update routing table
                IF NOT this.routingTable.Contains(target.id) THEN
                    this.routingTable.Add(target)
                END IF
                this.routingTable.UpdateLastSeen(target.id, Now())

            CATCH error
                this.routingTable.MarkUnresponsive(target.id)
            END TRY
        END FOR

        // Remove peers not seen recently
        this.routingTable.EvictStale(maxAge: Duration.Minutes(15))
    END LOOP
END FUNCTION

// === PEER DEPARTURE HANDLING ===

FUNCTION PeerNode.HandlePeerDeparture(departedPeerId: String):
    LOG "Peer " + departedPeerId + " departed, checking data responsibility"

    // Find data that the departed peer was responsible for
    SET affectedKeys = this.dataStore.GetKeysReplicatedTo(departedPeerId)

    FOR EACH key IN affectedKeys DO
        // Re-replicate to maintain replication factor
        SET currentReplicas = this.FindCurrentReplicas(key)
        IF Length(currentReplicas) < this.replicationFactor THEN
            SET targetId = HashToId(key)
            SET candidates = this.routingTable.FindClosest(targetId,
                count: this.replicationFactor + 2)

            FOR EACH candidate IN candidates DO
                IF NOT Contains(currentReplicas, candidate.id) THEN
                    SET value = this.dataStore.Get(key)
                    candidate.Replicate(key, value)
                    LOG "Re-replicated " + key + " to " + candidate.id
                    BREAK
                END IF
            END FOR
        END IF
    END FOR
END FUNCTION

// === CONTENT DISTRIBUTION (File Sharing Example) ===

FUNCTION PeerNode.DownloadFile(fileHash: String) -> File:
    // Find peers that have chunks of this file
    SET metadata = this.FindValue("meta:" + fileHash)
    IF metadata IS NULL THEN
        RAISE FileNotFound("No peers have file " + fileHash)
    END IF

    SET chunks = NEW Array(metadata.chunkCount)
    SET pendingChunks = Range(0, metadata.chunkCount - 1)

    WHILE Length(pendingChunks) > 0 DO
        FOR EACH chunkIndex IN pendingChunks DO
            SET chunkKey = fileHash + ":chunk:" + chunkIndex
            SET result = this.FindValue(chunkKey)

            IF result.isFound THEN
                SET chunks[chunkIndex] = result.value
                Remove(pendingChunks, chunkIndex)

                // Become a source for this chunk
                this.StoreValue(chunkKey, result.value)
            END IF
        END FOR
    END WHILE

    SET file = AssembleChunks(chunks)
    ASSERT VerifyHash(file) = fileHash
    RETURN file
END FUNCTION
```

## When to Use

- **File sharing and content distribution** where decentralization avoids
  single-point bandwidth bottlenecks.
- **Decentralized systems** requiring censorship resistance or no central
  authority (blockchain, distributed ledgers).
- **Edge computing** where processing should happen at the network edge
  rather than in centralized data centers.
- **Collaborative systems** where peers contribute resources (storage,
  compute, bandwidth) proportional to their consumption.
- **Resilient communication** that must survive network partitions and
  node failures without centralized coordination.

## When NOT to Use

- **Systems requiring strong consistency** where all nodes must agree on
  state at all times. P2P favors eventual consistency.
- **Low-trust environments** without cryptographic verification, where
  malicious peers could corrupt data or disrupt the network.
- **Simple client-server interactions** where centralization is simpler
  and adequate for the scale.
- **Real-time coordination** requiring low-latency consensus across all
  participants.
- **Regulated environments** where centralized audit and control are
  legally required.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| File sharing | Distributed file network | Decentralized storage, no single point of failure |
| Blockchain | Distributed ledger | Trustless consensus, censorship resistance |
| Communication | Decentralized messaging | Privacy, no central server to compromise |
| CDN | Peer-assisted content delivery | Edge caching, bandwidth distribution |
| Scientific computing | Distributed computation | Aggregate idle compute resources across many machines |

## Trade-offs

### Advantages

- **No single point of failure**: System survives any individual node loss.
- **Scalability**: Capacity grows with the number of participants.
- **Resource efficiency**: Leverages distributed resources of all peers.
- **Censorship resistance**: No central authority to shut down.
- **Cost distribution**: Infrastructure cost shared across participants.

### Disadvantages

- **Complexity**: Discovery, routing, consistency, and security are hard.
- **Eventual consistency**: Data may be temporarily inconsistent across peers.
- **Free-rider problem**: Some peers consume without contributing.
- **Security challenges**: Malicious peers, Sybil attacks, data poisoning.
- **Unpredictable performance**: Depends on peer availability and bandwidth.

## Comparison with Related Patterns

| Dimension | Peer-to-Peer | Client-Server | Broker |
|-----------|-------------|--------------|--------|
| Topology | Fully decentralized | Centralized | Centralized routing |
| Failure resilience | High | Low (server is SPOF) | Medium |
| Consistency | Eventual | Strong | Configurable |
| Discovery | DHT, gossip | Known server address | Service registry |
| Trust model | Cryptographic verification | Server authority | Broker authority |

## Evolution and Variations

- **Pure P2P**: Every peer is identical. No special roles. Maximum
  decentralization but harder to bootstrap.
- **Hybrid P2P**: Some peers serve special roles (super-nodes, trackers,
  bootstrap nodes) while most peers are equal. Improves discovery.
- **Structured P2P**: Peers are organized in a specific overlay topology
  (ring, tree, hypercube) with deterministic routing via DHT.
- **Unstructured P2P**: Peers connect randomly. Discovery relies on
  flooding or random walks. Simpler but less efficient.
- **Federated Model**: Multiple independent servers that communicate
  with each other using a P2P protocol. Users connect to their chosen
  server but messages flow between servers.

## Key Takeaways

1. P2P eliminates the central server but replaces it with distributed
   protocols that are significantly more complex to implement correctly.
2. DHT is the key enabling technology for structured P2P networks. It
   provides logarithmic lookup time for finding data across the network.
3. Replication is essential for durability. Peers join and leave at any
   time; data must survive individual node departures.
4. Security in P2P requires cryptographic verification at every step.
   Without a central authority, every message and every piece of data
   must be independently verifiable.
5. The gossip protocol is the P2P equivalent of "word of mouth." It
   provides eventual consistency with probabilistic guarantees and is
   remarkably effective for state dissemination.
