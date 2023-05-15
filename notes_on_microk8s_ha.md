## Test 1
On a cluster with 3 nodes in HA, v1.26.4, hard shutdown one. There is a replicaset of 4 nginx pods running on this cluster. 2 of the nodes went into `NotReady` state.  One being the shutdown node and the other being one of the running 2. The reason is that microk8s needs 3 voters for an HA setup and this situation wasn't clear. I force removed the unavailable node `microk8s remove-node --force`, and the second node became `Ready` again, but in the control plane acted as a `spare` node and no `voter` anymore, since with 2 HA is not possible.

Later ran the ansible `bootstrap_microk8s_cluster` playbook on the formatted node and joined it manually to the cluster. Worked fine.

## Test 2
On a cluste with 3 nodes in HA, v1.27.1, hard shutdown one. There is a replicaset of 4 nginx pods runing on this cluster. 1 of the nodes went into `NotReady` state (the shutdown one). 2 nginx pods (on the shutdown node) hang in `Terminating` state, and 2 new ones were setup on the remaining noed. The SD card of the shutdown node was formatted. `bootstrap_microk8s_cluster` playbook with `v1.26` was ran on it. `microk8s` was upgraded with `upgrade_microk8s_cluster` playbook. Upon joining the new node to the cluster, I had a failure because a node with the same name was still registered with the cluster in a `NotReady` status. I had to `microk8s remove-node --force` the node (like done in Test 1) and then join the node to the cluster.

**Important Note:** For HA, we need at least 4 nodes or better 5. Otherwise we aren't HA anymore with a single node failure.