output:
```
2015/11/04 23:10:36 Request: {"method":"list_dbs","params":[],"id":1}
2015/11/04 23:10:36 Response: {"id":1,"error":null,"result":["Open_vSwitch"]}
2015/11/04 23:10:36 Request: {"method":"transact","params":["Open_vSwitch",{"op":"select","table":"Interface","where":[["name","==","patch-tun_ptn101a"]]}],"id":2}
2015/11/04 23:10:36 Response: {"id":2,"error":null,"result":[{"rows":[{"link_speed":["set",[]],"duplex":["set",[]],"link_resets":["set",[]],"status":["map",[]],"admin_state":["set",[]],"other_config":["map",[]],"_version":["uuid","8606aa97-743b-4a6c-a7bc-b6a9a9f0ee43"],"ofport":-1,"name":"patch-tun_ptn101a","link_state":["set",[]],"type":"patch","mtu":["set",[]],"mac":["set",[]],"cfm_mpid":["set",[]],"external_ids":["map",[]],"cfm_fault":["set",[]],"_uuid":["uuid","9a5b9407-27c7-41bb-944a-f071d8aad155"],"ingress_policing_rate":0,"options":["map",[["peer","patch-int_l2sw101a"]]],"statistics":["map",[]],"cfm_remote_mpids":["set",[]],"ingress_policing_burst":0,"lacp_current":["set",[]]}]}]}
```