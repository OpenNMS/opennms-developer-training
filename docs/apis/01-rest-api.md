# REST API

## Goal 

Understand how to use OpenNMS' REST APIs.

For example, you should know how to:
* Create a requisition
* Trigger an import for a requisition
* Add meta-data
* Retrieve nodes
* Update alarms
* Create a graph

## History

Original API available is called v1

v2 added FIQL support

New endpoints are now only added to v2

## v1

## v2

FIQL Queries:
```
% curl -u admin:admin -H "Accept: application/json" 'http://localhost:8980/opennms/api/v2/nodes?limit=3&_s=node.label==Node*' | jq
{
  "id": "30",
  "label": "Node9"
}
{
  "id": "29",
  "label": "Node8"
}
{
  "id": "28",
  "label": "Node7"
}
```

Need lots of examples of FIQL
Use integration tests

TODO: Graph API

## Lab

Write a REST client in Java
