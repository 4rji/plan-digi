![Starlink-Primary Resilient Gateway Reference Lab architecture](assets/lab-diagram-tabloid-EN.png)

# Starlink-Primary Resilient Gateway Reference Lab

A reference lab for resilient remote sites using **Starlink as the primary WAN**, **5G as backup**, and a **Digi IX40-05** as the edge router. Digi Remote Manager provides fleet configuration and monitoring, while XBee and ConnectCore MP255 add site telemetry and embedded edge processing.

## About the project

This project represents a remote or industrial site that must remain connected when its primary satellite link is degraded or unavailable. The Digi IX40-05 manages the Starlink and 5G connections, while Digi Remote Manager provides centralized visibility and device management.

The lab also integrates XBee sensors and a ConnectCore MP255 embedded device to collect site data and continue operating during connectivity interruptions. It compares native SureLink failover with Digi WAN Bonding and can include an independent out-of-band management path for remote recovery.

## Objective

The objective is to validate a practical and reusable architecture for resilient remote connectivity. The project demonstrates how satellite, cellular backup, centralized management, and edge telemetry can work together to keep a remote site observable and manageable during network failures.

This repository currently contains the architecture, requirements, build plan, test strategy, and risk register. Start with the [master plan](01-plan-maestro.md) or the short [executive pitch](08-executive-pitch-EN.md).
