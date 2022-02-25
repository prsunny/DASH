# DASH Sonic Orchestration HLD
## High Level Design Document
### Rev 0.1

# Table of Contents

  * [Revision](#revision)

  * [About this Manual](#about-this-manual)

  * [Definitions/Abbreviation](#definitionsabbreviation)
 
  * [1 Requirements Overview](#1-requirements-overview)
    * [1.1 Functional requirements](#11-functional-requirements)
    * [1.2 CLI requirements](#12-cli-requirements)
    * [1.3 Warm Restart requirements ](#13-warm-restart-requirements)
    * [1.4 Scaling requirements ](#14-scaling-requirements)
  * [2 Packet Flows](#2-packet-flows)
  * [3 Modules Design](#3-modules-design)
    * [3.1 Config DB](#31-config-db)
    * [3.2 App DB](#32-app-db)
    * [3.3 Module Interaction](#33-module-interaction)
    * [3.4 CLI](#34-cli)
    * [3.5 Test Plan](#35-test-plan)

###### Revision

| Rev |     Date    |       Author       | Change Description                |
|:---:|:-----------:|:------------------:|-----------------------------------|
| 0.1 | 02/01/2022  |     Prince Sunny   | Initial version                   |
| 0.1 | 02/23/2022  |     Prince Sunny   | Packet Flows                      |

# About this Manual
This document provides more detailed design of Dash APIs, Dash orchestration agent and the Schemas. General Dash HLD can be found at <TBD> 

# Definitions/Abbreviation
###### Table 1: Abbreviations
|                          |                                |
|--------------------------|--------------------------------|
| DASH                     | Disaggregated APIs for Sonic Hosts |
| VNI                      | Vxlan Network Identifier       |
| VTEP                     | Vxlan Tunnel End Point         |
| VNET                     | Virtual Network                |
| ENI                      | Elastic Network Interface      |
| gNMI                     | gRPC Network Management Interface |
| vPORT                    | VM's NIC. Eni, Vnic, VPort are used interchangeably |

# 1 Requirements Overview

## 1.1 Functional requirements

At a high level the following should be supported:
  
- Bringup Sonic image for DEVICE_METADATA subtype - `appliance`
- Bringup Swss/Syncd containers for switch_type - `dpu`
- Able to program DASH objects configured via gRPC client to appliance card via SAI DASH API

  Phase #1
    - Vnet-Vnet scenario
    - Route/LPM support
    - Underlay IPv4 and IPv6
    - Stateful ACL support
    - TCP state tracking on flows
    - Telemetry and Monitoring

  Phase #2
    - Private Link
    - Overlay IPv6 
    - Rest of features
  

## 1.2 CLI requirements
- User should be able to show the DASH configured objects

## 1.3 Warm Restart requirements
No special handling for Warm restart support.

## 1.4 Scaling requirements
TBD  
| Item                     | Expected value              |
|--------------------------|-----------------------------|
| VNETs         | 1024                         |

# 2 Packet Flows

# 3 Modules Design

The following are the schema changes. 

## 3.1 Config DB

### 3.1.1 DEVICE Metadata Table

```
"DEVICE_METADATA": {
    "localhost": {
        "subtype": "appliance",
        "type": "sonichost",
        "switch_type": "dpu",
        "sub_role": "None"
     }
}
```

### 3.1.2 VXLAN Table
```
VXLAN_TUNNEL|{{tunnel_name}} 
    "src_ip": {{ip_address}} 
    "dst_ip": {{ip_address}} (OPTIONAL)
```
### 3.1.3 VNET/Interface Table
  
```
VNET|{{vnet_name}} 
    "vxlan_tunnel": {{tunnel_name}}
    "vni": {{vni}} 
    "scope": {{"default"}} (OPTIONAL)
    "peer_list": {{vnet_name_list}} (OPTIONAL)
    "advertise_prefix": {{false}} (OPTIONAL)
```

## 3.2 APP DB

### VNET

The following are the changes for Vnet Route table

``` 
VNET_ROUTE_TUNNEL_TABLE:{{vnet_name}}:{{prefix}} 
    "endpoint": {{ip_address}} 
    "mac_address":{{mac_address}} (OPTIONAL) 
    "vni": {{vni}}(OPTIONAL) 
```
  
```
key                      = VNET_ROUTE_TUNNEL_TABLE:vnet_name:prefix ; Vnet route tunnel table with prefix 
; field                  = value 
ENDPOINT                 = list of ipv4 addresses    ; comma separated list of endpoints
ENDPOINT_MONITOR         = list of ipv4 addresses    ; comma separated list of endpoints, space for empty/no monitoring
MAC_ADDRESS              = 12HEXDIG                  ; Inner dst mac in encapsulated packet 
VNI                      = DIGITS                    ; VNI value in encapsulated packet 
WEIGHT                   = DIGITS                    ; Weights for the nexthops, comma separated (Optional) 
PROFILE                  = STRING                    ; profile name to be applied for this route, for community  
                                                       string etc (Optional) 
```
## 3.3 Module Interaction

A high-level module interaction is captured in the following diagram.

  ![dash-high-level-diagram](https://github.com/prsunny/DASH/blob/main/Assets/dash-high-level-design.svg)
  
### 3.3.1 Sonic host containers

The following containers shall be enabled for sonichost and part of the image. Switch specific containers shall be disabled for the image built for the appliance card.
  
| Container/Feature Name   | Is Enabled?     |
|--------------------------|-----------------|
|	SNMP | Yes |
|	Telemetry	| Yes |
|	LLDP | Yes |
|	Syncd |	Yes |
|	Swss | Yes |
|	Database | Yes |
|	BGP | Yes |
|	Teamd	| No |
|	Pmon | Yes |
|	Nat | No |
|	Sflow | No |
|	DHCP Relay | No |
|	Radv | No |
|	Macsec | No |
|	Resttapi | No |
|	gNMI | Yes |

### 3.3.2 DashOrch (Overlay)
A new orchestration agent "dashorch" shall be implemented that subscribes to DASH CONFIG_DB/APP_DB objects and programs the ASIC_DB via the SAI DASH API. Dashorch shall have sub-orchestrations to handle ACLs, Routes, CA-PA mappings. DASH orchestration agent shall write the state of each tables to STATEDB that applications shall utilize to fetch the programmed status of configured objects.
  
DASH APIs shall be exposed as gNMI interface and part of the Sonic gNMI container. Clients shall configure the Sonic via gRPC get/set calls. gNMI container has the config backend to translate/write  DASH objects to CONFIG_DB and/or APP_DB.

### 3.3.3 SWSS Lite (Underlay)
Sonic for DASH shall have a lite swss initialization without the heavy-lift of existing switch based orchestration agents that Sonic currently have. The initialization shall be based on switch_type "dpu". For the underlay support, the following SAI APIs are expected to be supported:
  
| Component                | SAI attribute                                         |
|--------------------------|-------------------------------------------------------|
| Host Interface           | SAI_HOSTIF_ATTR_NAME |
|                          | SAI_HOSTIF_ATTR_OBJ_ID |
|                          | SAI_HOSTIF_ATTR_TYPE |
|                          | SAI_HOSTIF_ATTR_OPER_STATUS |
|                          | SAI_HOSTIF_TABLE_ENTRY_ATTR_CHANNEL_TYPE |
|                          | SAI_HOSTIF_TABLE_ENTRY_ATTR_HOST_IF |
|                          | SAI_HOSTIF_TABLE_ENTRY_ATTR_TRAP_ID |
|                          | SAI_HOSTIF_TABLE_ENTRY_ATTR_TYPE |
|                          | SAI_HOSTIF_TRAP_ATTR_PACKET_ACTION |
|                          | SAI_HOSTIF_TRAP_ATTR_TRAP_GROUP |
|                          | SAI_HOSTIF_TRAP_ATTR_TRAP_PRIORITY |
|                          | SAI_HOSTIF_TRAP_ATTR_TRAP_TYPE |
|                          | SAI_HOSTIF_TRAP_GROUP_ATTR_POLICER |
|                          | SAI_HOSTIF_TRAP_GROUP_ATTR_QUEUE | 
| Neighbor                 | SAI_NEIGHBOR_ENTRY_ATTR_DST_MAC_ADDRESS |
| Nexthop                  | SAI_NEXT_HOP_ATTR_IP  | 
|                          | SAI_NEXT_HOP_ATTR_ROUTER_INTERFACE_ID  | 
|                          | SAI_NEXT_HOP_ATTR_TYPE  |
| Packet                   | SAI_PACKET_ACTION_FORWARD  | 
|                          | SAI_PACKET_ACTION_TRAP  | 
|                          | SAI_PACKET_ACTION_DROP  | 
| Policer                  | SAI_POLICER_ATTR_CBS |
|                          | SAI_POLICER_ATTR_CIR |
|                          | SAI_POLICER_ATTR_COLOR_SOURCE |
|                          | SAI_POLICER_ATTR_GREEN_PACKET_ACTION |
|                          | SAI_POLICER_ATTR_METER_TYPE |
|                          | SAI_POLICER_ATTR_MODE |
|                          | SAI_POLICER_ATTR_PBS  |      
|                          | SAI_POLICER_ATTR_PIR  |  
|                          | SAI_POLICER_ATTR_RED_PACKET_ACTION  |  
|                          | SAI_POLICER_ATTR_YELLOW_PACKET_ACTION  |  
| Port                     | SAI_PORT_ATTR_ADMIN_STATE  |  
|                          | SAI_PORT_ATTR_ADVERTISED_AUTO_NEG_MODE  |  
|                          | SAI_PORT_ATTR_ADVERTISED_FEC_MODE  |  
|                          | SAI_PORT_ATTR_ADVERTISED_INTERFACE_TYPE  |  
|                          | SAI_PORT_ATTR_ADVERTISED_MEDIA_TYPE  |  
|                          | SAI_PORT_ATTR_ADVERTISED_SPEED
|                          | SAI_PORT_ATTR_AUTO_NEG_MODE  |  
|                          | SAI_PORT_ATTR_FEC_MODE  |  
|                          | SAI_PORT_ATTR_HW_LANE_LIST  |  
|                          | SAI_PORT_ATTR_INTERFACE_TYPE  |  
|                          | SAI_PORT_ATTR_MTU  |  
|                          | SAI_PORT_ATTR_OPER_SPEED  |  
|                          | SAI_PORT_ATTR_OPER_STATUS  |  
|                          | SAI_PORT_ATTR_SPEED  |  
| RIF                      | SAI_ROUTER_INTERFACE_ATTR_ADMIN_V4_STATE  |  
|                          | SAI_ROUTER_INTERFACE_ATTR_ADMIN_V6_STATE  |  
|                          | SAI_ROUTER_INTERFACE_ATTR_MTU  |
|                          | SAI_ROUTER_INTERFACE_ATTR_PORT_ID  |
|                          | SAI_ROUTER_INTERFACE_ATTR_SRC_MAC_ADDRESS |
|                          | SAI_ROUTER_INTERFACE_ATTR_TYPE  |  
|                          | SAI_ROUTER_INTERFACE_ATTR_VIRTUAL_ROUTER_ID  |  
| Route                    | SAI_ROUTE_ENTRY_ATTR_NEXT_HOP_ID  |  
|                          | SAI_ROUTE_ENTRY_ATTR_PACKET_ACTION  |  
| Switch                   | SAI_SWITCH_ATTR_CPU_PORT  |  
|                          | SAI_SWITCH_ATTR_DEFAULT_TRAP_GROUP  |  
|                          | SAI_SWITCH_ATTR_DEFAULT_VIRTUAL_ROUTER_ID  |
|                          | SAI_SWITCH_ATTR_DEFAULT_VLAN_ID  |
|                          | SAI_SWITCH_ATTR_ECMP_DEFAULT_HASH_SEED |
|                          | SAI_SWITCH_ATTR_INIT_SWITCH  |  
|                          | SAI_SWITCH_ATTR_PORT_LIST  |  
|                          | SAI_SWITCH_ATTR_PORT_NUMBER  |  
|                          | SAI_SWITCH_ATTR_PORT_STATE_CHANGE_NOTIFY  |  
|                          | SAI_SWITCH_ATTR_SHUTDOWN_REQUEST_NOTIFY  |  
|                          | SAI_SWITCH_ATTR_SRC_MAC_ADDRESS |
|                          | SAI_SWITCH_ATTR_SWITCH_ID   |  
|                          | SAI_SWITCH_ATTR_TYPE  |  
|                          | SAI_SWITCH_ATTR_VXLAN_DEFAULT_PORT  |  
|                          | SAI_SWITCH_ATTR_VXLAN_DEFAULT_ROUTER_MAC |  

## 3.4 CLI

The following commands shall be modified/added :

```
	- show dash routes all
	- show dash acls
```

## 3.5 Test Plan
