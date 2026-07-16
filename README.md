# This is a Zabbix template for Lenovo server monitoring via SNMP v3.

### Based on [VSKUMBHANI/zabbix-templates repository](https://github.com/VSKUMBHANI/zabbix-templates/blob/main/Lenovo%20XCC%20SNMPv3.yaml).

No idea what the original template source is.

---

Changes are based on what is included in the Lenovo [MIB files](https://datacentersupport.lenovo.com/gb/en/products/servers/thinksystem/sr550/7x04/solutions/ht511161). The MIB files seem to be the same across different Lenovo server models, so it does not matter for which you download them.
Storages and NAS are probably different.

100% compatible with Lenovo SR550, unknown for others. Feel free to test.

---

## Added additional functionalities:
- Overall system health status: using `systemHealthStat, monitors.4.1`. Single health enum (normal/nonCritical/critical/nonRecoverable), with three triggers (critical/warning/disaster). Corresponds with what the XCC UI says.
- System health summary discovery: using `systemHealthSummaryTable`. Pairs with the item above. This table only populates while there is an active issue, so it gives you the specific reason behind a bad overall health status. Uses loose text matching for severity since the MIB defines it as a free-form `OCTET STRING`, not an enum. May be inaccurate accross different servers.
- Total power available, CPU power consumption, memory power consumption: similar to the pre-existing `Total power in use` under `fuelGaugeInformation`, same table, just unused fields.
- RAID volume discover: uses `raidVolumeTable`. Monitors status/capacity/bootable per logical array. The previous RAID drive discovery was monitoring single disks, but not the array itself.
- RAID card battery status: it has no trigger, since the MIB does not enumerate valid values. A comment is left to set `{$RAID_BATTERY_OK_STATUS}` if you want to check what your RAID card reports and set it manually.
- PSU discover: now also pulls data for the part number and serial number.
- Network card discovery: now also pulls MAC address and port maximum speed.
- CPU discovery: now also pulls CPU clockspeed and thread count.
- Fan discovery: now discovers properly all installed fans, even if not all slots are populated.
- Properly tagged components

## Fixes:
- Fixed issue with the Disk discovery, having wrong starting index.

## Tested on the following setup:
Server: Lenovo SR5550 7X04 \
CPU: 1x Intel(R) Xeon(R) Silver 4208 \
RAM: 3x Kingston (part number: LV32D4R2D8HD-161R), 1x SK Hynix (part number: HMA82GR7DJR8N-XN) \
RAID: ThinkSystem RAID 5350-8i PCIe 12Gb Adapter (part number: SR17A85787) \
HDD: 2x MG08SDA400N (part number: SHD7A23126) in RAID1 \
HDD: 2x ST32000645NS (part number: ST32000645NS) in RAID1 \
PSU: 2x ACBE (part number: SP57A96630) \
Fans: 3/4 installed and active \
Backplane: LNVO (part number: SC57A01988) \
Ethernet: Intel X722 LOM



## How to use:
1. Follow the instructions for installing generic MIBs: https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/snmp/mibs
2. Make sure your `/etc/snmp/snmp.conf` file looks like this `mibdirs /usr/share/snmp/mibs:/usr/share/snmp/mibs/iana:/usr/share/snmp/mibs/ietf:/usr/local/share/snmp/mibs/`. You need to have the `mibdirs` to include the default SNMP MIBs folder as well.
3. Download the template file.
4. The template was made for Zabbix Server 7.0.
5. If your version differs, edit the YAML and change line 2 `version: '7.0'` to whatever you are using.
6. Go to Zabbix Server UI -> Data Collection -> Templates -> Import. Requires to have the `Linux by SNMP` template, which should be installed if you followed steps 1-2.
7. Download the [Lenovo XCC MIB files](https://datacentersupport.lenovo.com/gb/en/products/servers/thinksystem/sr550/7x04/solutions/ht511161).
8. Extract the `.mib` files into `/usr/local/share/snmp/mibs`.
9. Restart the Zabbix Server service.
10. Go to the Lenovo Server XCC UI -> BMC Configuration -> User/LDAP -> Create.
11. Set a username and password of your choice.
12. Set `Authority Level` to `Read only`.
13. Enable toggle for `SNMP Settings`.
14. Set `Privacy password` of your choice.
15. Set `Authentication protocol` to `HMAC-SHA`.
16. Set `Privacy protocol` to `AES`.
17. Apply the changes
18. Add a Lenovo server as a Zabbix Host with `SNMP` interface.
19. Set `SNMP version` to `SNMPv3`.
20. Leave `Context name` empty.
21. Set `Security name` as the username of the newly created user.
22. Set `Security level` to `authPriv`.
23. Set `Authentication protocol` to `SHA1`.
24. Set `Authentication passphrase` to `{$SNMP.USER.PASSWORD}`.
25. Set `Privacy protocol` to `AES128`.
26. Set `Privacy passphrase to `{$SNMP.USER.PRIVACY}`.
27. Set Macros for the `{$SNMP.USER.PASSWORD}` and `{$SNMP.USER.PRIVACY}` using `Secret Text` option and set the values to the passwords you have set in steps 11 and 14.


## Known issues:
- Network card discovery -> Item Prototype for `Network Adapter Link speed` is outputting 0Gbps. Needs to be debugged and fixed.
- Network card discovery -> Item prototype for `Network Adapter: {#NETNAME}` uses `adapterstatus` key. Needs to be thought out what we may need here.


---
Changes co-authored with Claude. I was too lazy to deep dive into the MIBs.
