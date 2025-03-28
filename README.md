# Self-Destructing USB – Log-Based Intrusion Detection & Forensic Analysis  

## Description  

This project explores a proactive cybersecurity solution: a **Self-Destructing USB Process** that logs unauthorized access attempts and executes a secure data wipe if connected to an untrusted device. By integrating log-based intrusion detection and forensic analysis, the project enhances USB security, mitigating the risk of unauthorized data extraction and cyber threats.  

## Core Cybersecurity Principles  

- **Confidentiality** – Ensuring sensitive data on the USB remains inaccessible to unauthorized users.  
- **Integrity** – Preventing unauthorized modifications to stored data and logs.  
- **Availability** – Implementing controlled access mechanisms to ensure legitimate users can utilize the USB securely.  

## Technical Background: Why This Topic Was Selected  

USB devices are one of the most common attack vectors in cybersecurity. While they offer convenience and portability, they also introduce significant security risks, making them an attractive target for cyber threats. This project focuses on addressing the following key concerns:  

- **Unauthorized Access & Data Theft** – Unprotected USBs can be easily accessed by malicious actors, leading to data breaches and loss of sensitive information.  
- **Malware & Exploitation** – Attack techniques such as **BadUSB**, **Rubber Ducky**, and **Stuxnet** demonstrate how USBs can be weaponized to inject malware, execute unauthorized commands, or compromise entire systems.  
- **Lack of Monitoring & Security Controls** – Many organizations do not have proper security controls in place to monitor USB activity, making it difficult to detect unauthorized access or malicious use.  

By designing a **Self-Destructing USB**, this project aims to mitigate these risks by implementing log-based intrusion detection and an automated self-destruction mechanism. This solution provides a proactive approach to securing USB devices and preventing data theft in untrusted environments.  


## Research Steps  

To develop the **Self-Destructing USB – Log-Based Intrusion Detection & Forensic Analysis** project, the following research steps were undertaken:  

- **Selected Linux-Based OS (Ubuntu)** – Chosen for its flexibility, scriptability, and hardware control capabilities.  
- **Developed Automation Scripts** – Used **Bash** and **Python** to monitor and trigger USB events dynamically.  
- **Explored Log Analysis Tools** – Investigated **Splunk** for real-time event tracking and threat detection as an optional security measure (Optional).  
- **Tested Secure Data Wiping Commands** – Experimented with commands like **diskutil eraseDisk** to simulate USB self-destruction upon detecting unauthorized access.  
- **Used Wireshark for Monitoring** – Implemented optional **USB-level activity monitoring** to detect potential data exfiltration (Optional).  
- **Evaluated Remote Logging & Alerting** – Configured **Splunk, Logwatch, and smart home notifications** to enable proactive security responses (Optional).  

These steps contributed to the overall security architecture, ensuring the USB device effectively mitigates unauthorized access attempts while providing forensic analysis capabilities.  


## Live Demonstration  

The project included a live demonstration showcasing the detection and self-destruction process in action. The key focus areas are:  

- Identifying unauthorized system access.  
- Logging details of an untrusted environment.  
- Executing data-wiping procedures securely.  

<img width="915" alt="Screen Shot 2025-03-28 at 5 34 27 PM" src="https://github.com/user-attachments/assets/a90e9831-9f64-4f8d-9595-d92b19c0ca4c" />
<img width="916" alt="Screen Shot 2025-03-28 at 5 34 00 PM" src="https://github.com/user-attachments/assets/79570ac3-3b89-452b-afa2-36a6d562a576" />
<img width="918" alt="Screen Shot 2025-03-28 at 5 37 30 PM" src="https://github.com/user-attachments/assets/034b7789-8da8-4049-85a4-dc23c549a000" />
<img width="806" alt="Screen Shot 2025-03-28 at 5 38 05 PM" src="https://github.com/user-attachments/assets/f805d679-104a-483c-8cf1-1f6ab773d84d" />
<img width="810" alt="Screen Shot 2025-03-20 at 3 55 51 PM" src="https://github.com/user-attachments/assets/51fbe48f-cc4e-4ef8-9ce9-c43f9231250e" />
<img width="810" alt="Screen Shot 2025-03-20 at 3 54 50 PM" src="https://github.com/user-attachments/assets/1be3c616-d387-48f2-80a4-8f9bc46d3ff0" />
<img width="808" alt="Screen Shot 2025-03-28 at 5 38 41 PM" src="https://github.com/user-attachments/assets/b4617e5e-513a-4728-8bf2-0f9af331df08" />


## Scripts  

### usb_monitor.sh  

This script continuously monitors for new USB devices, logs relevant system information, and provides a security response. If an unauthorized USB is detected, it prompts the user with an option to initiate self-destruction.  

#### **Location**  
`/Users/jhmcalpine/usb-security/usb_monitor.sh`  

#### **Code**  

```bash
#!/bin/bash

LOGFILE="/Users/jhmcalpine/usb-security/usbsecurity.log"
LAST_STATE_FILE="/Users/jhmcalpine/usb-security/usb_last_state"
AUTHORIZED_USB="TrustedUSB"  # Replace with actual authorized USB name

# Get current mounted volumes
CURRENT_STATE=$(ls /Volumes)

# Get last known state
if [ -f "$LAST_STATE_FILE" ]; then
    LAST_STATE=$(cat "$LAST_STATE_FILE")
else
    LAST_STATE=""
fi

# Find new USB devices by comparing the states
NEW_USB=$(comm -13 <(echo "$LAST_STATE") <(echo "$CURRENT_STATE"))

# If no new USB is found, exit the script (prevents running on ejection)
if [ -z "$NEW_USB" ]; then
    exit 0
fi

# Update last known state
echo "$CURRENT_STATE" > "$LAST_STATE_FILE"

# Capture system details
TIMESTAMP=$(date)
HOSTNAME=$(hostname)
MAC_ADDRESS=$(ifconfig en0 | awk '/ether/{print $2}')
ACTIVE_USER=$(whoami)
TOP_PROCESSES=$(ps aux | sort -nrk 3,3 | awk 'NR<=5 {printf " %s%% %s\n", $3, $11}')

# Log event
echo "-----------------------------" >> "$LOGFILE"
echo "Timestamp: $TIMESTAMP" >> "$LOGFILE"
echo "Hostname: $HOSTNAME" >> "$LOGFILE"
echo "MAC Address: $MAC_ADDRESS" >> "$LOGFILE"
echo "Active User: $ACTIVE_USER" >> "$LOGFILE"
echo "Top 5 Running Processes (CPU Usage % | Command):" >> "$LOGFILE"

if [ -n "$TOP_PROCESSES" ]; then
    echo "$TOP_PROCESSES" >> "$LOGFILE"
else
    echo "No process data available." >> "$LOGFILE"
fi

echo "USB inserted: $NEW_USB at $(date)" >> "$LOGFILE"
echo "Mounted Volumes: $CURRENT_STATE" >> "$LOGFILE"
echo "-----------------------------" >> "$LOGFILE"

# Check for authorized or unauthorized USB
if echo "$NEW_USB" | grep -q "$AUTHORIZED_USB"; then
    osascript -e 'display notification "Authorized USB detected. Log generated." with title "USB Monitor"'
    echo "Authorized USB detected. Log generated." >> "$LOGFILE"
else
    RESPONSE=$(osascript -e 'display dialog "Unauthorized USB detected! 
Initiate self-destruct?" buttons {"Yes", "No"} default button "No"')

    if [[ "$RESPONSE" == *"Yes"* ]]; then
        echo "Self-destruct initiated!" >> "$LOGFILE"
        sudo diskutil eraseDisk HFS+ USB /dev/disk2  # Modify this to target the correct disk
    else
        echo "Self-destruct canceled by user." >> "$LOGFILE"
    fi
fi

---

## Impacts & Security Enhancements  

By implementing this **Self-Destructing USB**, organizations and individuals can:  

- **Prevent Data Theft** – Attackers cannot extract sensitive data even if they gain physical access to the USB.  
- **Detect Unauthorized Use** – Logs provide forensic evidence for incident response.  
- **Enhance Endpoint Security** – Organizations can integrate similar mechanisms into enterprise USB security policies.  

## Mitigation: Securing USB Devices  

To prevent USB-based attacks, it is essential to:  

- Disable auto-run functionality on endpoints.  
- Use encrypted USBs with authentication mechanisms.  
- Regularly monitor USB access logs for anomalies.  
- Implement strict access control policies for removable devices.  

---


## Final Thoughts 

This project demonstrates a novel approach to USB security by integrating **log-based intrusion detection and automated self-destruction**. By applying these techniques, users can significantly reduce the risk of unauthorized access and data theft.  
