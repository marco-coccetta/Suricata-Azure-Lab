#Suricata NIDS Lab - Azure Network Monitoring

[![Suricata](https://img.shields.io/badge/Suricata-7.x-orange?style=flat&logo=suricata)](https://suricata.io/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20LTS-E95420?style=flat&logo=ubuntu)](https://ubuntu.com/)
[![Azure](https://img.shields.io/badge/Azure-VM-B1s-0078D4?style=flat&logo=microsoftazure)](https://azure.microsoft.com/)

##Obiettivo del Laboratorio
Implementazione pratica di **Suricata IDS** su VM Azure per:
- Network Security Monitoring (NSM)
- Analisi traffico (HTTP/DNS/ICMP)
- Threat Hunting su log JSON (`eve.json`)
- Comprendere "rumore di fondo" Azure

Azure (West Europe)
├── RG: RG-CyberLab
├── VM: Ubuntu 22.04 LTS (B1s, IP pubblico)
│   ├── Suricata IDS (eth0 → eve.json)
│   └── jq (JSON parsing)
└── NSG: SSH solo mio IP



## Guida Riproducibile (da Zero)

1) Azure VM Deployment

Portale Azure → VM → Ubuntu 22.04 → B1s → SSH key → NSG: SSH solo mio IP

2) Accesso e Preparazione

ssh -i azure_key.pem azureuser@IP_VM
sudo apt update && sudo apt upgrade -y

3) Suricata Installation

sudo apt install suricata jq -y
sudo suricata-update  # Aggiorna regole
sudo systemctl enable --now suricata

4) Traffic Generation

curl -I http://testmyids.com      # HTTP
dig @8.8.8.8 google.com          # DNS  
ping -c 5 8.8.8.8                # ICMP

5) Log Analysis (jq)


# HTTP
tail /var/log/suricata/eve.json | jq 'select(.event_type=="http")'

# DNS
tail /var/log/suricata/eve.json | jq 'select(.event_type=="dns") | .dns.rrname'

# ICMP (nuovo!)
tail /var/log/suricata/eve.json | jq 'select(.proto=="ICMP")'

# Top IP
tail -200 /var/log/suricata/eve.json | jq -r '.src_ip // .dest_ip' | sort | uniq -c | sort -nr

Esempio Log ICMP (Ping Reale)

text
{
  "timestamp": "2026-01-14T16:38:00",
  "proto": "ICMP",
  "icmp": { "type": 8, "code": 0 },  // Echo Request (ping)
  "src_ip": "10.0.0.4",
  "dest_ip": "8.8.8.8"
}

Risultati

    5970+ pacchetti processati

    Azure Agent (168.63.129.16) identificato 

    ICMP, HTTP, DNS catturati 

    JQ queries pronte per SOC 

Hardening Applicato

    NSG: SSH limitato al mio IP

    Costi: ~€0.01/ora (B1s)

    Cleanup: Delete Resource Group

Skills Demonstrate

Junior SOC Analyst:

    Azure VM provisioning & NSG

    Suricata IDS deployment

    Log parsing (JSON/jq)

    Network protocol analysis (ICMP/DNS/HTTP)

    Documentation & reproducibility

