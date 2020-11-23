# Links

## Goal

Understand how OpenNMS learns links and how they are modeled.
Understand the Graph APIs.

## Lecture

## Hands On

```
admin@opennms> feature:install opennms-enlinkd-shell
admin@opennms> opennms:generate-topology --protocol cdp
```

Enumerate the available graphs:
```
curl -X GET -u admin:admin http://localhost:8980/opennms/api/v2/graphs
```

Show the default vertices on the graph:
```
curl -X GET -u admin:admin http://localhost:8980/opennms/api/v2/graphs/nodes/nodes
```

Show connections around node with id=21:
```
curl -X POST -u admin:admin -H "Content-Type: application/json" -d '{ "semanticZoomLevel": 1, "verticesInFocus": ["21"] }' http://localhost:8980/opennms/api/v2/graphs/nodes/nodes
```