# wazuh-wireguard-integration
Wazuh Rules and Decoders for WireGuard Server (Linux)
![Post logo](/images/posts/wazuh-wireguard-integration/post-logo.png "Wazuh & WireGuard Logs Monitoring on a Linux Host")
## Wazuh & WireGuard Logs Monitoring on a Linux Host

WireGuard is an extremely simple yet fast and modern VPN that utilizes state-of-the-art cryptography. While its simplicity and performance are major advantages, one of its design choices is a minimalistic approach to logging. By default, WireGuard does not write detailed events to a dedicated log file, which can be a challenge for security monitoring.

This guide provides a practical solution for monitoring of a Linux-based WireGuard server and integration with Wazuh. We will leverage rsyslog to capture kernel-level WireGuard events and forward them to a custom log file. I have also prepared a ready-to-use set of custom Wazuh decoders and rules to parse these logs and generate meaningful security alerts.

## Event Monitoring Capabilities

This integration allows you to monitor for events such as:

- **130001**: Successful handshake initiations
- **130002**: Invalid handshake initiations (potential attacks or misconfigurations)
- **130003**: Standard keepalive packets
- **130004**: Packets sent from an unallowed source IP (policy violation, potential attack)

## Requirements

- A WireGuard server installed on a Linux host
- A Wazuh agent installed on the same Linux host
- rsyslog service (installed by default on most Linux distributions)

## Part 1: Enabling Log Capture via rsyslog on the WireGuard Server

First, we need to instruct the rsyslog service to listen for kernel messages containing the word "wireguard" and redirect them to a dedicated log file.

### Step 1: Create a new rsyslog configuration file

```bash
sudo nano /etc/rsyslog.d/20-wireguard.conf
```

### Step 2: Add the following rules to the file

This rule finds any message with "wireguard:", writes it to `/var/log/wireguard.log`, and then stops processing it further to avoid duplication.

```bash
# Rule for WireGuard logs
:msg, contains, "wireguard:" /var/log/wireguard.log
& stop
```
![rsyslog configuration](/images/posts/wazuh-wireguard-integration/edit_rsyslog_conf.png "Edit rsyslog rules")
### Step 3: Restart the rsyslog service

Save the file and restart the rsyslog service to apply the changes.

```bash
sudo systemctl restart rsyslog
```

You should now see WireGuard events appearing in `/var/log/wireguard.log`.
You can check events with following command:
```bash
sudo tail -f /var/log/wireguard.log
```
![rsyslog output](/images/posts/wazuh-wireguard-integration/wireguard_tail_command.png "WireGuard logs in rsyslog")
## Part 2: Configuring the Wazuh Agent to Monitor the Log File

Next, we configure the Wazuh agent to read our new log file and forward the events to the Wazuh Manager. We will also add a custom prefix to each log line, which will help our custom decoders identify the logs.

### Step 1: Open the Wazuh agent configuration file

```bash
sudo nano /var/ossec/etc/ossec.conf
```

### Step 2: Add the localfile block

Add the following `<localfile>` block within the `<ossec_config>` section.

```xml
<localfile>
  <location>/var/log/wireguard.log</location>
  <log_format>syslog</log_format>
  <out_format>wireguard-log: $(log)</out_format>
</localfile>
```

**Note**: The `<out_format>` tag is crucial. It prepends `wireguard-log:` to every log entry before sending it to the manager. Our parent decoder will use this unique string to match the incoming events.
![ossec.conf edit](/images/posts/wazuh-wireguard-integration/edit_ossec_conf.png "Edit ossec.conf")
### Step 3: Restart the Wazuh agent service

Save the file and restart the Wazuh agent service.

```bash
sudo systemctl restart wazuh-agent
```

## Part 3: Importing Custom Rules and Decoders to the Wazuh Manager

Now, on your Wazuh Manager server, we will download and install the custom rule and decoder files.

### Step 1: Clone the GitHub repository

Clone the GitHub repository containing the integration files.

```bash
git clone https://github.com/jayzielinski/wazuh-wireguard-integration.git
```

### Step 2: Copy the decoder and rule files

Copy the decoder and rule files to the correct Wazuh directories.

```bash
sudo cp wazuh-wireguard-integration/130000-wireguard_decoders.xml /var/ossec/etc/decoders/
sudo cp wazuh-wireguard-integration/130000-wireguard_rules.xml /var/ossec/etc/rules/
```

### Step 3: Set the correct ownership and permissions

Set the correct ownership and permissions for the new files. This is essential for the Wazuh manager to be able to read them.

```bash
sudo chown wazuh:wazuh /var/ossec/etc/rules/130000-wireguard_rules.xml
sudo chmod 660 /var/ossec/etc/rules/130000-wireguard_rules.xml
```

### Step 4: Restart the Wazuh Manager service

Restart the Wazuh Manager service to load the new decoders and rules.

```bash
sudo systemctl restart wazuh-manager
```

## Part 4: Testing the Integration

You can now test the rules by triggering specific events on a WireGuard client.

### Test Case 1: Triggering Rule 130004 (Unallowed IP)

This rule detects when a legitimate peer sends traffic from an IP address that is not listed in its AllowedIPs configuration on the server.

To trigger this, I modified my WireGuard client configuration, changing the client's Address to an IP not covered by the server's AllowedIPs for that peer, and then tried to send traffic.

**Result on WireGuard Server** (`/var/log/wireguard.log`):
```
[Log entry showing unallowed IP packet]
```

**Result on Wazuh Dashboard**: An alert with description `WireGuard: Peer [...] sent a packet with an unallowed source IP [...]` was generated.

### Test Case 2: Triggering Rule 130002 (Invalid Handshake)

This rule detects a failed handshake attempt, which can occur due to a mismatched private/public key pair. This is a critical security event to monitor.

To trigger this, I changed the PrivateKey on my WireGuard client configuration to an incorrect value and attempted to connect to the server.

**Result on Wazuh Dashboard**: An alert with description `WireGuard: Invalid handshake initiation from [...]` was generated.
![Results in Wazuh Dashboard](/images/posts/wazuh-wireguard-integration/wazuh_wireguard_dashboard_logs.png "Results in Wazuh Dashboard")
![Results on WireGuard Server](/images/posts/wazuh-wireguard-integration/wazuh_wireguard_logs.png "Logs in Rsyslog")
## Conclusion

Your integration is now complete and actively monitoring your WireGuard server! This setup enables security monitoring of WireGuard VPN connections, helping you detect potential attacks, misconfigurations, and policy violations in real-time.

## Repository

For the complete integration files (decoders and rules), visit:
[https://github.com/jayzielinski/wazuh-wireguard-integration](https://github.com/jayzielinski/wazuh-wireguard-integration) 

## Would you like me to create other interesting content related to Wazuh or cybersecurity? 

Contact me on one of the platforms listed at:
[https://www.cylenth.blog/about](https://www.cylenth.blog/about)

---
