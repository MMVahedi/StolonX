# Master eligibility selector

You can restrict which keepers are allowed to be elected master by setting a `masterEligibilitySelector` in the [cluster specification](cluster_spec.md).

**Behaviour**

- If `--can-be-master` is explicitly set to `false`, it always takes precedence over `masterEligibilitySelector` (the keeper can never become a master).
- Label matching is **exact** (both key and value must match).
- Only `matchLabels` is supported.
- This configuration can be updated **dynamically** using `stolonctl update` without restarting keepers.
- Keeper labels are **static** — they are defined at keeper startup with the `--labels` flag. Changing labels requires restarting the keeper.
- Labels are reported by keepers to the sentinel and stored in the cluster state.

## Example: Restrict master to a specific zone

To ensure that the master is always located in a specific zone:

1. Start keepers with zone labels:

```bash
# Keeper in zone A
stolon-keeper --labels="topology.kubernetes.io/zone=zone-a" ...

# Keeper in zone B
stolon-keeper --labels="topology.kubernetes.io/zone=zone-b" ...

# Keeper in zone C
stolon-keeper --labels="topology.kubernetes.io/zone=zone-c" ...
```

2. Do not explicitly set `--can-be-master` to `false` on these keepers, unless you explicitly don't want them to become masters.

3. Apply selectors:

```bash
stolonctl update --patch '{
  "masterEligibilitySelector": {
    "matchLabels": {
      "topology.kubernetes.io/zone": "zone-a"
    }
  }
}'
```

**Result**

Only keepers in `zone-a` *whose `--can-be-master`* is not explicitly set to `false` can become master. Failover will not promote replicas from other zones.

> **Note**: *If no eligible keepers are available (e.g., zone failure), the cluster will not elect a new master until the configuration is updated or more keepers with matching labels become available.*
>
> *This feature does not guarantee availability across zones. It enforces a policy constraint on master election and may prevent failover if no eligible keepers are available.*
