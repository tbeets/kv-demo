# KV Demo
Demo essential NATS KV features and extend demo to edge use-cases.

## Prerequisites
* NATS Server, JetStream-enabled
* NATS Account, JetStream-enabled
* NATS CLI tool
* Optional: NATS Cluster, NATS Supercluster, another NATS system configured as a Leafnode

### Create a KV Store
Let's create a KV Store (or bucket) called MENUBUCKET in which key values expire after 10 minutes and a history of up
to 5 values is kept for a key.

```text
nats kv add --help
nats kv add MENUBUCKET --history=5 --ttl=10m --replicas=3 --description="My cool menu bucket"
```
> Note: If you are not running a 3+ node NATS Cluster, set `--replicas=1` or omit.

We can delete a whole bucket easily, too.
```text
# Goodbye bucket!
nats kv del MENUBUCKET
```

### Put, get, and delete key-value pairs
Let's put some keys in our bucket and get them back. Then we'll "update" an existing key by putting another value.

```text
nats kv put --help
nats kv put MENUBUCKET sku1 "latte"

nats kv get --help
nats kv get MENUBUCKET sku1

nats kv put MENUBUCKET sku2 "bagel"

nats kv put MENUBUCKET sku1 "latte with oat milk"
nats kv get MENUBUCKET sku1

# Goodbye key sku2!
nats kv del MENUBUCKET sku2
```

### List, Info and history
Let's get information about our bucket, and also see the value history of one of the keys.

```text
# List the current keys in a bucket
nats kv ls --help
nats kv ls MENUBUCKET

nats kv info --help
nats kv info MENUBUCKET

nats kv history --help
nats kv history MENUBUCKET sku1
```

### Using history to revert to a key revision
Let's revert to the previous value of sku1 with revision number 1:

```text
nats kv revert MENUBUCKET sku1 1
```

### Update only if unmodified
In some cases, we might want to update the value of an existing key -- perhaps with a calculation or conditional update --
only if the key value we last read (and used) has not changed.

```text
nats kv update --help
nats kv history MENUBUCKET sku1

nats kv update MENUBUCKET sku1 "Fresh orange juice" 4
nats kv update MENUBUCKET sku1 "Fresh orange juice" 5
```
>Note: There is a similar cousin to Update: Create is like Put, but `nats kv create` will only "put" a key if the key does not
> already exist (or has been deleted).

### Create a watch on a KV Store or a particular key
KV watch allows our application to subscribe and see change events at the bucket level or the key level. 
Perfect for EDA!

```text
# Any change to the bucket
nats kv watch MENUBUCKET

# A specific key in the bucket
nats kv watch MENUBUCKET sku1
```
## Optional Demos

### Use a cloud KV from an edge leaf node
As a NATS connection at your edge location, access a KV Store in your "cloud" NATS System over leaf connection...

```text
nats kv history MENUBUCKET sku1 --js-domain=hub1
```

### Mirror a KV from cloud to edge
Create a continuously updated (and reliable), read-only copy of MENUBUCKET to an edge location. The bucket is then available to read at
the edge (low RTT) and is available even if the edge location is not currently connected with the origin bucket.

```text
nats kv add MENUBUCKET --history=5 --ttl=1d --description "mirror at my store" --mirror=MENUBUCKET --mirror-domain=hub1

# We don't specify `--js-domain` because we are referencing our local bucket now!
nats kv history MENUBUCKET sku1
```

> Note: Max history, TTL, storage type, and limits all may be different than the origin KV Store. This is useful if your
> edge location has different retention and versioning requirements than your origin location, etc.

## Life is but a stream (under the covers)
KV is a lightweight abstraction, a _materialized view_ over normal NATS JetStream, thus it inherits all the technology
and benefit of NATS streaming data services.

* KV Bucket is a stream
* KV Key is a message (or messages with history > 1) in the stream with unique key subject
* NATS Client Libraries provide a developer abstraction for KV -- the Synadia-supported language types all support KV
including .NET, Java, JavaScript/TypeScript, Python, Go, Rust, and C
* NATS CLI provides a command-line abstraction (using the NATS Client Library for Go)

### See the stream behind the bucket
To include KV streams in a list of streams in the account use the `--all` flag.

```text
nats stream list --all

# Look at the underlying stream info as `KV_<bucket name>`
nats stream info KV_MENUBUCKET

# Do all the cool JetStream things such as automatically move your Bucket from one cluster to another cluster in your
# supercluster!
nats stream edit KV_MENUBUCKET --cluster=cluster2
```
# See also
- [NATS.io Documentation](https://docs.nats.io/nats-concepts/jetstream/key-value-store)
- [tbeets/kv-demo](https://github.com/tbeets/kv-demo)


