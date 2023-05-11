---
description: Reference configuration example - Only for educational/collaboration purposes. Not to be used in production.  
last update: 04/13/2022
---

# Reference configuration example (JSON)

**WARNING - This example is in JSON format and it is intended for educational/collaboration purposes only. It is not intended for production use. The goal for production is to have the schema in [YANG](https://www.fir3net.com/Networking/Protocols/an-introduction-to-netconf-yang.html) format.**

```xml
{
    "qos-settings": [
        {
            "253de6f9-37bd-40ce-9cb2-9715915941d3": {
                "qos-id": "253de6f9-37bd-40ce-9cb2-9715915941d3",
                "bw": 100000000,
                "cps": 1000000,
                "flows": 500000
            }
        }
    ],

    "ha-groups": [
        {
            "group1": {
                "group-id": "group1",
                "status": "active",
                "v4": {
                    "bpg-community-id": 123, // also other bpg settings
                    "vip": "100.9.9.1"
                },
                "v6": {
                    "bpg-community-id": 123, // also other bpg settings
                    "vip": "2601:12:7a:88::1"
                }
            }
        },
        {
            "group2": {
                "group-id": "group2",
                "status": "standby",
                "v4": {
                    "bpg-community-id": 123, // also other bpg settings
                    "vip": "100.9.9.2"
                },
                "v6": {
                    "bpg-community-id": 123, // also other bpg settings
                    "vip": "2601:12:7a:88::2"
                }
            }
        }
    ],

    "enis": [
        {
            "F4939FEFC47E": {
                "eni-id": "497f23d7-f0ac-4c99-a98f-59b470e8c7bd",
                "mac": "F4939FEFC47E",
                "ha-group-id": "group1",
                "qos-id": "253de6f9-37bd-40ce-9cb2-9715915941d3",
                "vpcs": [
                    "559c6ce8-26ab-4193-b946-ccc6e8f930b2"
                ],
                "acls-v4-in": [
                    {
                        "acl-group-id": "0cf54937-efca-4481-9db3-49a642141bf4",
                        "stage": 1
                    },
                    {
                        "acl-group-id": "c7f8a564-f602-45b9-9969-68b1c9ee19fd",
                        "stage": 2
                    },
                    {
                        "acl-group-id": "c6b01c61-ed80-44f5-b2b3-e21b6ab06d1f",
                        "stage": 3
                    },
                    {
                        "acl-group-id": "", // pass-thru
                        "stage": 4
                    },
                    {
                        "acl-group-id": "", // pass-thru
                        "stage": 5
                    }
                ],
                "acls-v4-out": [
                    {
                        "acl-group-id": "0cf54937-efca-4481-9db3-49a642141bf4",
                        "stage": 1
                    },
                    {
                        "acl-group-id": "c7f8a564-f602-45b9-9969-68b1c9ee19fd",
                        "stage": 2
                    },
                    {
                        "acl-group-id": "c6b01c61-ed80-44f5-b2b3-e21b6ab06d1f",
                        "stage": 3
                    },
                    {
                        "acl-group-id": "", // pass-thru
                        "stage": 4
                    },
                    {
                        "acl-group-id": "", // pass-thru
                        "stage": 5
                    }
                ],
                "acls-v6-in": [ ],
                "acls-v6-out": [ ],
                "route-table-v4": "7c968a65-d892-405f-bee4-85af64a34ea9",
                "route-table-v6": "f8642d0c-80dd-4a66-a3ae-7e92abe69dae",
                "snat-policy-v4": "",
                "snat-policy-v6": "",
                "metering-policy-v4-in": "c5746b36-83eb-4a45-a6a2-e56484518d17",
                "metering-policy-v4-out": "b1e0d6de-28e6-4f03-9226-e3ae14493243",
                "metering-policy-v6-in": "",
                "metering-policy-v6-out": ""
            }
        }
    ],

    "vpc": [
        {
            "559c6ce8-26ab-4193-b946-ccc6e8f930b2": {
                "vpc-id": "559c6ce8-26ab-4193-b946-ccc6e8f930b2",
                "vni-key": 45654,
                "encap": "vxlan",
                "address_spaces": [
                    "10.1.0.0/16",
                    "10.2.0.0/16",
                    "2001:0123:1111:1000/64",
                    "2001:0123:1111:2000/64"
                ]
            }
        }
    ],

    //
    // active routing actions (active routing modules)
    // list of active modules impacts memory and needs to be configured as part of initial card config
    // Maximum 16 different routing actions (as each action ID will take 4 bits).
    //

    "routing-actions": [
        "none",        // 0x0
        "maprouting",  // 0x1
        "direct",      // 0x2
        "staticencap", // 0x3
        "appliance",   // 0x4
        "4to6",        // 0x5
        "mapdecap",    // 0x6
        "decap",       // 0x7
        "drop"         // 0x8
    ],

    //
    // Example memory how type of actions can be stored:
    //
    // -----------------------------------------------------------------------------------------------------------------------
    // | routing-type | order of actions | action-1 arguments | action-2 arguments | action-3 arguments | action-4 arguments |
    // -----------------------------------------------------------------------------------------------------------------------
    // Order of actions: 16-bit value that is split into 4x4 bits.
    // Example: 4to6 -> staticencap -> appliance -> none (will be represented as: 0x5340)
    // When packet hits the pipeline, it looks up LPM prefix, once the prefix is found, it gets routing type and persist arguments from LPM route
    // Then next stage takes routing type and looksup order ex. 0x5340 and persists all default arguments for stages
    // Then we go 4-times thru routing actions and apply applicable action to the packet based on carried arguments.
    // Important: "maprouting" has special implementation. If it is present (0x1000), then it will be only 1 action.
    // This action first does lookup to find the routing-type again based on prefix, and then performs the routing stages.
    //

    "routing-types-config": [
        {
            "direct": { 
                "routing-type": "direct",
                "actions": [ // up to 4 actions
                    {
                        "action-type": "direct"
                    }
                ]
            }
        },
        {
            "vpc": {
                "routing-type": "vpc",
                "actions": [ // ONLY 1 action here! More then 1 action is not allowed if action-type used is "maprouting"
                    {
                        "action-type": "maprouting" // NOTE: "maprouting" will be the ONLY action. It cannot be used with other actions
                    }
                ]
            }
        },
        {
            "vpc-direct": {
                "routing-type": "vpc-direct",
                "actions": [ // up to 3 actions
                    {
                        "action-type": "staticencap",
                        "encap-type": "vxlan"
                    }
                ]
            }
        },
        {
            "drop": {
                "routing-type": "drop",
                "actions": [ // up to 3 actions
                    {
                        "action-type": "drop"
                    }
                ]
            }
        },
        {
            "appliance": {
                "routing-type": "appliance",
                "actions": [ // up to 3 actions
                    {
                        "action-type": "appliance"
                    }
                ]
            }
        },
        {
            "privatelink": {
                "routing-type": "privatelink",
                "actions": [ // up to 3 actions
                    {
                        "action-type": "4to6"
                    },
                    {
                        "action-type": "staticencap",
                        "encap-type": "nvgre",
                        "vni-key": 100
                    }
                ]
            }
        },
        {
            "privatelinknsg": {
                "routing-type": "privatelinknsg",
                "actions": [ // up to 3 actions
                    {
                        "action-type": "4to6"
                    },
                    {
                        "action-type": "staticencap",
                        "encap-type": "nvgre",
                        "vni-key": 100
                    },
                    {
                        "action-type": "appliance",
                    }
                ]
            }
        },
        {
            "servicetunnel": {
                "routing-type": "servicetunnel",
                "actions": [ // up to 3 actions
                    {
                        "action-type": "4to6"
                    },
                    {
                        "action-type": "staticencap",
                        "encap-type": "nvgre",
                        "vni-key": 100
                    }
                ]
            }
        }
    ],

    //
    // active routing types for mappings (active routing types modules)
    // list of active modules impacts memory area reserved for mappings and needs to be configured as part of initial card config
    //

    "vpc-mappings-routing-types": [
        "vpc",
        "privatelink",
        "privatelinknsg"
    ],

    //
    // Example table in device memory (how mappings can be stored):
    // --------------------------------------------------------------------------------------------------------------------------------------
    // | overlay-ip-address | metering-bucket | routing-type | underlay-ip-address | mac | overlay-sip | overlay-dip | routing-appliance-id |
    // --------------------------------------------------------------------------------------------------------------------------------------
    //

    "vpc-mappings": [
        {
            "559c6ce8-26ab-4193-b946-ccc6e8f930b2": {
                "vpc-id": "559c6ce8-26ab-4193-b946-ccc6e8f930b2",
                "mappings" : [
                    {
                        "routing-type": "vpc-direct",
                        "overlay-ip-address": "10.0.0.5", // customer-address is unique per vpc
                        "underlay-ip-address": "100.1.2.3", // not unique
                        "mac": "F922839922A1", /// not unique, overlay destination mac
                        "metering-bucket": 0  // not unique
                    },
                    {
                        "routing-type": "vpc-direct",
                        "overlay-ip-address": "10.0.0.6",
                        "underlay-ip-address": "2601:12:7a:1::1234",
                        "mac": "F922839922A2",
                        "metering-bucket": 2223
                    },
                    {
                        "routing-type": "vpc-direct",
                        "overlay-ip-address": "2001:0123:1111:1000::5", // customer-address is unique per vpc
                        "underlay-ip-address": "2601:12:7a:1::1234", // not unique
                        "mac": "F922839922A2", // not unique, overlay destination mac
                        "metering-bucket": 2223 // not unique
                    },
                    {
                        "routing-type": "privatelink",
                        "overlay-ip-address": "10.1.0.8", // customer-address is unique per vpc
                        "underlay-ip-address": "50.1.2.3", // not unique
                        "mac": "F9A18A92207", // not unique, overlay destination mac
                        "overlay-sip": "fd40:108:0:d204:0:200::0", // not unique
                        "overlay-dip": "2603:10e1:100:2::3401:203", // not unique
                        "metering-bucket": 18872 // not unique
                    },
                    {
                        "routing-type": "privatelinknsg",
                        "overlay-ip-address": "10.1.0.9", // customer-address is unique per vpc
                        "underlay-ip-address": "50.1.2.8", // not unique
                        "mac": "F9A18A92209", // not unique, overlay destination mac
                        "overlay-sip": "fd40:108:0:d204:0:200::9", // not unique
                        "overlay-dip": "2603:10e1:100:2::3401:203", // not unique
                        "routing-appliance-id": 22,
                        "metering-bucket": 18872 // not unique
                    }
                ]
            }
        }
    ],

    "routing-appliances": [ // static tunnels to routing appliances
        {
            "7b0fa23b-f2dc-4574-bcee-d4e58aa20501": { // PL NSG SDN Appliance
                "appliance-id": "7b0fa23b-f2dc-4574-bcee-d4e58aa20501",
                "routing-appliance-id": 22,
                "routing-appliance-addresses": [ "100.8.1.2" ],
                "encap-type": "vxlan",
                "vni-key": 100
            },
            "225b6a80-b645-41c1-9a8d-486fcfc829a3": { // ExpressRoute Gateway
                "appliance-id": "225b6a80-b645-41c1-9a8d-486fcfc829a3",
                "routing-appliance-id": 28,
                "routing-appliance-addresses": [ // ECMP across those addresses
                    "10.8.8.1",
                    "10.8.8.2"
                ],
                "encap-type": "vxlan",
                "vni-key": 5555
            }
        }
    ],

    "metering-policies": [
        {
            "c5746b36-83eb-4a45-a6a2-e56484518d17": {
                "metering-policy-id": "c5746b36-83eb-4a45-a6a2-e56484518d17",
                "metering-bucket": 802, // default metering bucket if nothing is matched
                "ip-version": "IPv4",
                "rules": [
                    {
                        "prefixes": [
                            "20.0.0.0/24",
                            "25.0.0.0/16",
                            "31.0.0.0/22"
                        ],
                        "metering-bucket": 0 // 0 - means no metering needed
                    },
                    {
                        "prefixes": [
                            "50.0.0.0/24",
                            "55.0.0.0/16",
                            "51.0.0.0/22"
                        ],
                        "metering-bucket": 21 // this is metered
                    }
                ]
            }
        },
        {
            "b1e0d6de-28e6-4f03-9226-e3ae14493243": {
                "metering-policy-id": "b1e0d6de-28e6-4f03-9226-e3ae14493243",
                "metering-bucket": 802, // default metering bucket if nothing is matched
                "ip-version": "IPv6",
                "rules": [
                    {
                        "prefixes": [
                            "2601:1122::/48"
                        ],
                        "metering-bucket": 0 // 0 - means no metering needed
                    },
                    {
                        "prefixes": [
                            "2601:22:aabc::/64",
                            "2601:ff::/48"
                        ],
                        "metering-bucket": 21 // this is metered
                    }
                ]
            }
        }
    ],

    "route-tables": [
        {
            "7c968a65-d892-405f-bee4-85af64a34ea9": {
                "routetable-id": "7c968a65-d892-405f-bee4-85af64a34ea9",
                "ip-version": "IPv4",
                "routes": [
                    {
                        "ip-prefixes": [ "10.1.0.0/16" ],
                        "action" : {
                            "routing-type": "vpc", // vnet route - destination within vnet, needs mapping table lookup
                            "vpc-id": "559c6ce8-26ab-4193-b946-ccc6e8f930b2"
                        },
                        "metering-bucket": 7 // default metering bucket that will be used in case mapping has metering bucket set to 0
                    },
                    {
                        "ip-prefixes": [ "10.1.0.0/24" ], // intercept part of the traffic to go thru Firewall VM
                        "action" : {
                            "routing-type": "vpc-direct", // vnet route to single destination inside VPC (ex. forward traffic thru Firewall VM) - destination within vnet, needs mapping table lookup
                            "vpc-id": "559c6ce8-26ab-4193-b946-ccc6e8f930b2",
                            "customer-address": "10.0.0.6"
                        },
                        "metering-bucket": 7 // default metering bucket that will be used in case mapping has metering bucket set to 0
                    },
                    {
                        "ip-prefixes": [ "10.1.0.128/28" ], // exempt subset of traffic from going thru firewall and allow to go directly
                        "action" : {
                            "routing-type": "vpc", // vnet route - destination within vnet, needs mapping table lookup
                            "vpc-id": "559c6ce8-26ab-4193-b946-ccc6e8f930b2"
                        },
                        "metering-bucket": 7 // default metering bucket that will be used in case mapping has metering bucket set to 0
                    },
                    {
                        "ip-prefixes": [ "10.1.0.8/32" ], // direct path to VNET (in this case private endpoint)
                        "action" : {
                            "routing-type": "vpc", // vnet route - destination within vnet, needs mapping table lookup
                            "vpc-id": "559c6ce8-26ab-4193-b946-ccc6e8f930b2"
                        },
                        "metering-bucket": 0 // default metering bucket that will be used in case mapping has metering bucket set to 0
                    },
                    {
                        "ip-prefixes": [ "10.2.0.0/16" ],
                        "action" : {
                            "routing-type": "vpc", // vnet route - destination within vnet, needs mapping table lookup
                            "vpc-id": "559c6ce8-26ab-4193-b946-ccc6e8f930b2"
                        },
                        "metering-bucket": 71 // default metering bucket that will be used in case mapping has metering bucket set to 0
                    },
                    {
                        "ip-prefixes": [ "80.0.0.0/8" ],
                        "action" : {
                            "routing-type": "appliance", // static encap - traffic to on-premise via hardware appliances (ex. CISCO/ARISTA/etc)
                            "routing-appliance-id": 28
                        },
                        "metering-bucket": 71 // default metering bucket that will be used in case mapping has metering bucket set to 0
                    },
                    {
                        "ip-prefixes": [ "10.2.5.0/24" ],
                        "action" : {
                            "routing-type": "drop" // null route - drop all packets
                        }
                    },
                    {
                        "ip-prefixes": [ "50.1.2.3/32", "52.2.2.0/24" ],
                        "action": {
                            "routing-type": "ServiceTunnel",
                            "overlay-ip-address": "", // do not override the overlay-ip-address - use dest ip from packet
                            "underlay-ip-address": "", // do not override the overlay-ip-address - use dest ip from packet
                            "overlay-sip": "fd40:108:0:d204:0:200::0", // not unique
                            "overlay-dip": "2603:10e1:100:2::3401:203" // not unique       
                        },
                        "metering-bucket": 887
                    },
                    {
                        "ip-prefixes": [ "30.0.0.0/16" ],
                        "action" : {
                            "routing-type": "direct", // direct traffic - no encap or anything
                        },
                        "metering-bucket": 65535 // special metering bucket saying, metering policy will be evaluated to select metering bucket
                    },
                    {
                        "ip-prefixes": [ "0/0" ], // default route - there will ALWAYS be 1 default route
                        "action" : {
                            "routing-type": "vpc-direct", // vnet route to single destination inside VPC (ex. forward traffic thru Firewall VM) - destination within vnet, needs mapping table lookup
                            "vpc-id": "559c6ce8-26ab-4193-b946-ccc6e8f930b2",
                            "customer-address": "10.0.0.6"
                        },
                        "metering-bucket": 82 // default metering bucket that will be used in case mapping has metering bucket set to 0
                    }
                ]
            }
        },
        {
            "f8642d0c-80dd-4a66-a3ae-7e92abe69dae": {
                "routetable_id": "f8642d0c-80dd-4a66-a3ae-7e92abe69dae",
                "ip_version": "IPv6",
                "routes": [
                    // similar as for IPv4
                ]
            }
        }
    ],

    "prefix-tags": [
        {
            "b44c8dcb-a992-4bbf-a405-578d03b55cc5": {
                "prefix-tag-id": "b44c8dcb-a992-4bbf-a405-578d03b55cc5",
                "prefix-tag-number": 3998, // globally unique
                "ip-prefixes-ipv4": [
                    "8.8.8.8/32",
                    "10.0.0.0/8"
                ],
                "ip-prefixes-ipv6": [
                    "2001:4898:e0:3b8:1489:d7d6:1633:f246/128",
                    "::1/128",
                    "2001:4898:e0:3b8::/64"
                ]
            }
        }
    ],

    "acl-groups": [
        {
            "c7f8a564-f602-45b9-9969-68b1c9ee19fd": {
                "acl-group-id": "c7f8a564-f602-45b9-9969-68b1c9ee19fd",
                "rules": []
            }
        },
        {
            "c6b01c61-ed80-44f5-b2b3-e21b6ab06d1f": {
                "acl-group-id": "c6b01c61-ed80-44f5-b2b3-e21b6ab06d1f",
                "ip_version": "IPv4",
                "rules": [
                    {
                        "priority": 100,
                        "action": "allow",
                        "terminating": "false",
                        "protocols": [ 
                            6, // tcp + udp
                            17
                        ],
                        "src_addrs": [
                            "8.8.8.8/32",
                            "10.0.0.0/8"
                        ],
                        "src_ports": [
                            { "from": 0, "to": 1000 },
                            { "from": 1020, "to": 5000 }
                        ],
                        "dst_addrs": [
                            "8.8.8.8/32",
                            "10.0.0.0/8",
                        ],
                        "dst_ports": [
                            { "from": 80, "to": 80 },
                            { "from": 443, "to": 443 }
                        ]
                    },
                    {
                        "priority": 101,
                        "action": "allow",
                        "terminating": "false",
                        "protocols": [ 
                            6 // tcp only
                        ],
                        "src_addrs": [
                            "#3998" // uses prefix tag
                        ],
                        "src_ports": [
                            { "from": 0, "to": 1000 },
                            { "from": 1020, "to": 5000 }
                        ],
                        "dst_addrs": [
                            "#3998" // uses prefix tag
                        ],
                        "dst_ports": [
                            { "from": 80, "to": 80 },
                            { "from": 443, "to": 443 }
                        ]
                    },
                    {
                        "priority": 102,
                        "action": "allow",
                        "terminating": "true",
                        "protocols": [ 
                            1 // icmp
                        ],
                        "src_addrs": [
                            "0/0"
                        ],
                        "src_ports": [ ],
                        "dst_addrs": [
                            "0/0"
                        ],
                        "dst_ports": [ ]
                    },
                    {
                        "priority": 110,
                        "action": "deny",
                        "terminating": "true",
                        "protocols": [ 
                            6,
                            17
                        ],
                        "src_addrs": [
                            "10.0.0.15/32",
                        ],
                        "src_ports": [
                            { "from": 0, "to": 65535 }
                        ],
                        "dst_addrs": [
                            "10.0.0.0/8",
                        ],
                        "dst_ports": [
                            { "from": 80, "to": 80 },
                            { "from": 443, "to": 443 }
                        ]
                    }
                ]
            }
        },
        {
            "0cf54937-efca-4481-9db3-49a642141bf4": {
                "acl-group-id": "0cf54937-efca-4481-9db3-49a642141bf4",
                "ip_version": "IPv6",
                "rules": [
                    {
                        "priority": 100,
                        "action": "allow",
                        "terminating": "false",
                        "protocols": [ 
                            6, // tcp + udp
                            17
                        ],
                        "src_addrs": [
                            "2001:4898:e0:3b8:1489:d7d6:1633:f246/128",
                            "::1/128",
                            "2001:4898:e0:3b8::/64"
                        ],
                        "src_ports": [
                            { "from": 0, "to": 1000 },
                            { "from": 1020, "to": 5000 }
                        ],
                        "dst_addrs": [
                            "2001:4898:e0:3b8:1489:d7d6:1633:f246/128",
                            "::1/128",
                            "2001:4898:e0:3b8::/64"
                        ],
                        "dst_ports": [
                            { "from": 80, "to": 80 },
                            { "from": 443, "to": 443 }
                        ]
                    },
                    {
                        "priority": 101,
                        "action": "allow",
                        "terminating": "false",
                        "protocols": [ 
                            6 // tcp only
                        ],
                        "src_addrs": [
                            "#3998" // uses prefix tag
                        ],
                        "src_ports": [
                            { "from": 0, "to": 1000 },
                            { "from": 1020, "to": 5000 }
                        ],
                        "dst_addrs": [
                            "#3998" // uses prefix tag
                        ],
                        "dst_ports": [
                            { "from": 80, "to": 80 },
                            { "from": 443, "to": 443 }
                        ]
                    },
                    {
                        "priority": 102,
                        "action": "allow",
                        "terminating": "true",
                        "protocols": [ 
                            1 // icmp
                        ],
                        "src_addrs": [
                            "0/0"
                        ],
                        "src_ports": [ ],
                        "dst_addrs": [
                            "0/0"
                        ],
                        "dst_ports": [ ]
                    },
                    {
                        "priority": 110,
                        "action": "deny",
                        "terminating": "true",
                        "protocols": [ 
                            6,
                            17
                        ],
                        "src_addrs": [
                            "2001:4898:e0:3b8:1489:d7d6:1633:1/128"
                        ],
                        "src_ports": [
                            { "from": 0, "to": 65535 }
                        ],
                        "dst_addrs": [
                            "2001:4898:e0:3b8::/64"
                        ],
                        "dst_ports": [
                            { "from": 80, "to": 80 },
                            { "from": 443, "to": 443 }
                        ]
                    }
                ]
            }
        }
    ],

    "snat-policies": [
        // to be defined
    ]

}


```
