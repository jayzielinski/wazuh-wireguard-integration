# wazuh-wireguard-integration
Wazuh Rules and Decoders for WireGuard Server (Linux)

#How to install in Wazuh?

1. Copy 130000-wireguard_decoders.xml to /var/ossec/etc/decoders/130000-wireguard_decoders.xml
2. Copy 130000-wireguard_rules.xml to /var/ossec/etc/rules/130000-wireguard_rules.xml
3. Set the correct ownership and permissions:
   sudo chown wazuh:wazuh /var/ossec/etc/rules/130000-wireguard_rules.xml
   sudo chmod 660 /var/ossec/etc/rules/130000-wireguard_rules.xml
4. Restart Wazuh Manager service.

Detailed installation instructions and tutorial how to enable WireGuard logs on Linux can be found at:
My blog:
My Medium:
