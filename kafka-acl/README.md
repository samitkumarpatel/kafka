# Kafka RBAC — ACL Setup

Role-based access control for Kafka topics, producers, and consumer groups. Default deny — every permission must be explicitly granted.

---

## Scenario

| Principal | Role | Allowed topic | Consumer group |
|---|---|---|---|
| `User:order-producer` | Producer | `orders` | — |
| `User:payment-producer` | Producer | `payments` | — |
| `User:order-consumer` | Consumer | `orders` | `order-cg` |
| `User:payment-consumer` | Consumer | `payments` | `payment-cg` |
| `User:kafka-admin` | Admin | all (cluster-level) | — |

---

## Broker config

Enable ACLs and set default deny in `server.properties`:

```properties
authorizer.class.name=kafka.security.authorizer.AclAuthorizer
super.users=User:kafka-admin
allow.everyone.if.no.acl.found=false
```

---

## Producer ACLs

A producer needs `Write` + `Describe` on its own topic only.

```bash
# order-producer → orders
kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:order-producer \
  --operation Write    --topic orders

kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:order-producer \
  --operation Describe --topic orders

# payment-producer → payments
kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:payment-producer \
  --operation Write    --topic payments

kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:payment-producer \
  --operation Describe --topic payments
```

Attempting to write to a topic with no ACL throws `TopicAuthorizationException` immediately.

---

## Consumer ACLs

A consumer needs **two** grants: one on the topic and one on its consumer group. The Group ACL is mandatory — `subscribe()` fails without it even if the Topic ACL exists.

```bash
# order-consumer → orders topic + order-cg group
kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:order-consumer \
  --operation Read     --topic orders

kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:order-consumer \
  --operation Describe --topic orders

kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:order-consumer \
  --operation Read     --group order-cg

# payment-consumer → payments topic + payment-cg group
kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:payment-consumer \
  --operation Read     --topic payments

kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:payment-consumer \
  --operation Describe --topic payments

kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:payment-consumer \
  --operation Read     --group payment-cg
```

`order-consumer` joining `payment-cg` (or reading `payments`) throws `GroupAuthorizationException`.

---

## Admin ACLs

Admin operations are scoped to the `Cluster` resource — not a wildcard on topics.

```bash
kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:kafka-admin \
  --operation Create   --cluster

kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:kafka-admin \
  --operation Alter    --cluster

kafka-acls.sh --bootstrap-server broker:9092 \
  --add --allow-principal User:kafka-admin \
  --operation Describe --cluster
```

---

## Verify ACLs

```bash
# All ACLs on a topic
kafka-acls.sh --bootstrap-server broker:9092 \
  --list --topic orders

# All ACLs for a principal
kafka-acls.sh --bootstrap-server broker:9092 \
  --list --allow-principal User:order-consumer

# All ACLs in the cluster
kafka-acls.sh --bootstrap-server broker:9092 --list
```

---

## Revoke an ACL

Replace `--add` with `--remove`:

```bash
kafka-acls.sh --bootstrap-server broker:9092 \
  --remove --allow-principal User:order-producer \
  --operation Write --topic orders
```

---

## ACL matrix

| Principal | Topic: orders | Topic: payments | Group: order-cg | Group: payment-cg |
|---|---|---|---|---|
| `order-producer` | WRITE, DESCRIBE | ✕ | — | — |
| `payment-producer` | ✕ | WRITE, DESCRIBE | — | — |
| `order-consumer` | READ, DESCRIBE | ✕ | READ | ✕ |
| `payment-consumer` | ✕ | READ, DESCRIBE | ✕ | READ |
| `kafka-admin` | via Cluster ACL | via Cluster ACL | — | — |

---

## Common errors

| Error | Cause | Fix |
|---|---|---|
| `TopicAuthorizationException` | No `Write` ACL on topic | Add `--operation Write --topic <name>` |
| `GroupAuthorizationException` | No `Read` ACL on group | Add `--operation Read --group <name>` |
| ACLs have no effect | `allow.everyone.if.no.acl.found=true` | Set to `false` in server.properties |
| `DescribeTopicException` on produce | Missing `Describe` | Add `--operation Describe --topic <name>` |
