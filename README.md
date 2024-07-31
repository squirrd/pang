# pang
PANG (Prometheus AlertManager NodeEporter Grafana) Stack

## Design Decisions
This is an example of an operator designed using the go operator SDK for deploying a simplified monitoring stack. [Design decisions were documented here](doc/design_decisions.md)

## Exercise Notes
[The development journey is documented here](doc/build_journey.md)

## Show my workings
Please see Exercise Notes for more details.  But the main piece of work was in architecting a metrics system and preparing the details.  This needs to be done before it can be wrapped in an operator.

This was done by building a file with all the required OCP objects in it.  This file can be viewed in [deploy_pang.yml](deploy_pang.yml).