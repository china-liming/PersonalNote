## 注释
Functions for acquiring data
获取数据的功能

## moloch_packet_init
```c
void moloch_packet_init()
{
    pcapFileHeader.magic = 0xa1b2c3d4;
    pcapFileHeader.version_major = 2;
    pcapFileHeader.version_minor = 4;

    pcapFileHeader.thiszone = 0;
    pcapFileHeader.sigfigs = 0;

    char filename[PATH_MAX];
    snprintf(filename, sizeof(filename), "/tmp/%s.tcp.drops.4", config.nodeName);
    moloch_drophash_init(&packetDrop4, filename, 4);

    snprintf(filename, sizeof(filename), "/tmp/%s.tcp.drops.6", config.nodeName);
    moloch_drophash_init(&packetDrop6, filename, 16);

    snprintf(filename, sizeof(filename), "/tmp/%s.tcp.drops.4S", config.nodeName);
    moloch_drophash_init(&packetDrop4S, filename, 12);

    snprintf(filename, sizeof(filename), "/tmp/%s.tcp.drops.6S", config.nodeName);
    moloch_drophash_init(&packetDrop6S, filename, 36);

    g_timeout_add_seconds(10, moloch_packet_save_drophash, 0);

    mac1Field = moloch_field_define("general", "lotermfield",
        "mac.src", "Src MAC", "source.mac",
        "Source ethernet mac addresses set for session",
        MOLOCH_FIELD_TYPE_STR_HASH,  MOLOCH_FIELD_FLAG_ECS_CNT | MOLOCH_FIELD_FLAG_LINKED_SESSIONS | MOLOCH_FIELD_FLAG_NOSAVE,
        "transform", "dash2Colon",
        "fieldECS", "source.mac",
        (char *)NULL);

    mac2Field = moloch_field_define("general", "lotermfield",
        "mac.dst", "Dst MAC", "destination.mac",
        "Destination ethernet mac addresses set for session",
        MOLOCH_FIELD_TYPE_STR_HASH,  MOLOCH_FIELD_FLAG_ECS_CNT | MOLOCH_FIELD_FLAG_LINKED_SESSIONS | MOLOCH_FIELD_FLAG_NOSAVE,
        "transform", "dash2Colon",
        "fieldECS", "destination.mac",
        (char *)NULL);

    dscpField[0] = moloch_field_define("general", "integer",
        "dscp.src", "Src DSCP", "srcDscp",
        "Source non zero differentiated services class selector set for session",
        MOLOCH_FIELD_TYPE_INT_GHASH,  MOLOCH_FIELD_FLAG_CNT,
        (char *)NULL);

    dscpField[1] = moloch_field_define("general", "integer",
        "dscp.dst", "Dst DSCP", "dstDscp",
        "Destination non zero differentiated services class selector set for session",
        MOLOCH_FIELD_TYPE_INT_GHASH,  MOLOCH_FIELD_FLAG_CNT,
        (char *)NULL);

    moloch_field_define("general", "lotermfield",
        "mac", "Src or Dst MAC", "macall",
        "Shorthand for mac.src or mac.dst",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        "regex", "^mac\\\\.(?:(?!\\\\.cnt$).)*$",
        "transform", "dash2Colon",
        (char *)NULL);

    oui1Field = moloch_field_define("general", "termfield",
        "oui.src", "Src OUI", "srcOui",
        "Source ethernet oui set for session",
        MOLOCH_FIELD_TYPE_STR_HASH,  MOLOCH_FIELD_FLAG_CNT | MOLOCH_FIELD_FLAG_LINKED_SESSIONS,
        (char *)NULL);

    oui2Field = moloch_field_define("general", "termfield",
        "oui.dst", "Dst OUI", "dstOui",
        "Destination ethernet oui set for session",
        MOLOCH_FIELD_TYPE_STR_HASH,  MOLOCH_FIELD_FLAG_CNT | MOLOCH_FIELD_FLAG_LINKED_SESSIONS,
        (char *)NULL);

    vlanField = moloch_field_define("general", "integer",
        "vlan", "VLan", "network.vlan.id",
        "vlan value",
        MOLOCH_FIELD_TYPE_INT_GHASH,  MOLOCH_FIELD_FLAG_ECS_CNT | MOLOCH_FIELD_FLAG_LINKED_SESSIONS | MOLOCH_FIELD_FLAG_NOSAVE,
        (char *)NULL);

    greIpField = moloch_field_define("general", "ip",
        "gre.ip", "GRE IP", "greIp",
        "GRE ip addresses for session",
        MOLOCH_FIELD_TYPE_IP_GHASH,  MOLOCH_FIELD_FLAG_CNT | MOLOCH_FIELD_FLAG_LINKED_SESSIONS,
        (char *)NULL);

    moloch_field_define("general", "integer",
        "tcpflags.syn", "TCP Flag SYN", "tcpflags.syn",
        "Count of packets with SYN and no ACK flag set",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        (char *)NULL);

    moloch_field_define("general", "integer",
        "tcpflags.syn-ack", "TCP Flag SYN-ACK", "tcpflags.syn-ack",
        "Count of packets with SYN and ACK flag set",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        (char *)NULL);

    moloch_field_define("general", "integer",
        "tcpflags.ack", "TCP Flag ACK", "tcpflags.ack",
        "Count of packets with only the ACK flag set",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        (char *)NULL);

    moloch_field_define("general", "integer",
        "tcpflags.psh", "TCP Flag PSH", "tcpflags.psh",
        "Count of packets with PSH flag set",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        (char *)NULL);

    moloch_field_define("general", "integer",
        "tcpflags.fin", "TCP Flag FIN", "tcpflags.fin",
        "Count of packets with FIN flag set",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        (char *)NULL);

    moloch_field_define("general", "integer",
        "tcpflags.rst", "TCP Flag RST", "tcpflags.rst",
        "Count of packets with RST flag set",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        (char *)NULL);

    moloch_field_define("general", "integer",
        "tcpflags.urg", "TCP Flag URG", "tcpflags.urg",
        "Count of packets with URG flag set",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        (char *)NULL);

    moloch_field_define("general", "integer",
        "packets.src", "Src Packets", "srcPackets",
        "Total number of packets sent by source in a session",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        "fieldECS", "source.packets",
        (char *)NULL);

    moloch_field_define("general", "integer",
        "packets.dst", "Dst Packets", "dstPackets",
        "Total number of packets sent by destination in a session",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        "fieldECS", "destination.packets",
        (char *)NULL);

    moloch_field_define("general", "integer",
        "initRTT", "Initial RTT", "initRTT",
        "Initial round trip time, difference between SYN and ACK timestamp divided by 2 in ms",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        (char *)NULL);

    moloch_field_define("general", "termfield",
        "communityId", "Community Id", "communityId",
        "Community id flow hash",
        0,  MOLOCH_FIELD_FLAG_FAKE,
        "fieldECS", "network.community_id",
        (char *)NULL);

    int t;
    for (t = 0; t < config.packetThreads; t++) {
        char name[100];
        DLL_INIT(packet_, &packetQ[t]);
        MOLOCH_LOCK_INIT(packetQ[t].lock);
        MOLOCH_COND_INIT(packetQ[t].lock);
        snprintf(name, sizeof(name), "moloch-pkt%d", t);
#ifndef FUZZLOCH
        g_thread_unref(g_thread_new(name, &moloch_packet_thread, (gpointer)(long)t));
#endif
    }

    HASH_INIT(fragh_, fragsHash, moloch_packet_frag_hash, (HASH_CMP_FUNC)moloch_packet_frag_cmp);
    DLL_INIT(fragl_, &fragsList);

    moloch_add_can_quit(moloch_packet_outstanding, "packet outstanding");
    moloch_add_can_quit(moloch_packet_frags_outstanding, "packet frags outstanding");


    moloch_packet_set_ethernet_cb(MOLOCH_ETHERTYPE_ETHER, moloch_packet_ether);
    moloch_packet_set_ethernet_cb(MOLOCH_ETHERTYPE_TEB, moloch_packet_ether); // ETH_P_TEB - Trans Ether Bridging
    moloch_packet_set_ethernet_cb(MOLOCH_ETHERTYPE_RAWFR, moloch_packet_frame_relay);
    moloch_packet_set_ethernet_cb(ETHERTYPE_IP, moloch_packet_ip4);
    moloch_packet_set_ethernet_cb(ETHERTYPE_IPV6, moloch_packet_ip6);
}

```