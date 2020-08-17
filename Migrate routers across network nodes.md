# Migrate routers across network nodes


- Migrate routers across network nodes
- Here we describe the procedure to migrate a virtual router from a failed L3 agent (network node) to another one.

- To simulate the failure we remove the router from the first node with the command:
```
# ip netns
  qrouter-120c7e50-3afe-495b-b4fd-58a1b142d17f (id: 0)

  ip netns del qrouter-120c7e50-3afe-495b-b4fd-58a1b142d17f
```
The router disappears from the configuration, but we still manage to ping it (it is probably cached).

- We stop the neutron-l3-agent service:
```
# service neutron-l3-agent stop
```
Now the router is lost for real.

1. From an OpenStack client let’s look for the router ID:
```
$ neutron router-list
+--------------------------------------+--------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+-------+
| id                                   | name         | external_gateway_info                                                                                                                                                                      | distributed | ha    |
+--------------------------------------+--------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+-------+
| 120c7e50-3afe-495b-b4fd-58a1b142d17f | admin-router | {"network_id": "3daad902-45b3-4ded-b426-324741650637", "enable_snat": true, "external_fixed_ips": [{"subnet_id": "fe522fdf-608b-4471-a009-6db6a0c076f2", "ip_address": "90.147.189.209"}]} | False       | False |
+--------------------------------------+--------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+-------+
```
2. We get the node hosting the L3 agent with the following command:
```
$ neutron l3-agent-list-hosting-router $ROUTER_ID
+--------------------------------------+------------+----------------+-------+----------+
| id                                   | host       | admin_state_up | alive | ha_state |
+--------------------------------------+------------+----------------+-------+----------+
| 9f9a2fe0-561f-43e8-aa94-1f3224266f06 | pa1-r2-s09 | True           | xxx   |          |
+--------------------------------------+------------+----------------+-------+----------+
```
- The xxx means the host is down, as we know.

3. To get the list of all L3 agents:
```
$ neutron agent-list| grep L3
+--------------------------------------+----------------------+------------+-------------------+-------+----------------+---------------------------+
| id                                   | agent_type           | host       | availability_zone | alive | admin_state_up | binary                    |
+--------------------------------------+----------------------+------------+-------------------+-------+----------------+---------------------------+
| 775a4d38-d6b7-4f6d-a34b-e5327563e2e5 | L3 agent             | pa1-r2-s09 | nova              | xxx   | True           | neutron-l3-agent          |
| 9f9a2fe0-561f-43e8-aa94-1f3224266f06 | L3 agent             | pa1-r1-s11 | nova              | :-)   | True           | neutron-l3-agent          |
+--------------------------------------+----------------------+------------+-------------------+-------+----------------+---------------------------+
```

4. All what we have to do now is to remove our router from the first agent and add it to the second one:
```
neutron l3-agent-router-remove $L3_AGENT1_ID $ROUTER_ID
neutron l3-agent-router-add $L£_AGENT2_ID $ROUTER_ID
```
```
$ neutron l3-agent-list-hosting-router 120c7e50-3afe-495b-b4fd-58a1b142d17f
+--------------------------------------+------------+----------------+-------+----------+
| id                                   | host       | admin_state_up | alive | ha_state |
+--------------------------------------+------------+----------------+-------+----------+
| 9f9a2fe0-561f-43e8-aa94-1f3224266f06 | pa1-r1-s11 | True           | :-)   |          |
+--------------------------------------+------------+----------------+-------+----------+
```
... and in a few seconds we ping the router again!