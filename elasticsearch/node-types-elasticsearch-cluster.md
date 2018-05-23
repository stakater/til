# What are the node types in ElasticSearch cluster?

However there are only 4 node types - master, data, client and tribe;

- `Master` only nodes take place in updating cluster state as well as master elections. They should never handle query or index loads.
- `Data` only nodes store data that is indexed into Elasticsearch. These can also handle querying and indexing.
- `Client` only nodes are used as load balancers for indexing and searching.
- `Tribe` nodes are akin to cross-cluster client nodes, in that they can query more than one cluster. Think federated search.

Elasticsearch best-practices recommend to separate nodes in three roles:

- Master nodes - intended for clustering management only, no data, no HTTP API
- Client nodes - intended for client usage, no data, with HTTP API
- Data nodes - intended for storing and indexing data, no HTTP API