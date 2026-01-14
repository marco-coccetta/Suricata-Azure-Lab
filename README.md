# Suricata NIDS Lab - Azure Network Monitoring

[![Suricata](https://img.shields.io/badge/Suricata-7.x-orange?style=flat&logo=suricata)](https://suricata.io/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20LTS-E95420?style=flat&logo=ubuntu)](https://ubuntu.com/)
[![Azure](https://img.shields.io/badge/Azure-VM-B1s-0078D4?style=flat&logo=microsoftazure)](https://azure.microsoft.com/)

## Obiettivo del laboratorio
Implementazione pratica di **Suricata IDS** su VM Azure per:
- Network Security Monitoring (NSM)
- Analisi traffico (HTTP/DNS/ICMP)
- Threat hunting su log JSON (`eve.json`)
- Comprendere il “rumore di fondo” tipico di Azure

## Architettura
```text
Azure (West Europe)
├── RG: RG-CyberLab
├── VM: Ubuntu 22.04 LTS (B1s, IP pubblico)
│   ├── Suricata IDS (eth0 -> eve.json)
│   └── jq (JSON parsing)
└── NSG: SSH consentito solo dal mio IP
```

## Guida riproducibile

### 1) Azure VM deployment
Portale Azure → Virtual Machines → Create → Ubuntu 22.04 → B1s → SSH key → NSG: SSH consentito solo dal mio IP.

### 2) Accesso e preparazione
```bash
ssh -i azure_key.pem azureuser@IP_VM
sudo apt update && sudo apt upgrade -y
```

### 3) Installazione Suricata
```bash
sudo apt install suricata jq -y
sudo suricata-update          # Aggiorna regole
sudo systemctl enable --now suricata
```

### 4) Generazione traffico
```bash
# HTTP
curl -I http://testmyids.com

# DNS
dig @8.8.8.8 google.com

# ICMP
ping -c 5 8.8.8.8
```

### 5) Log analysis (jq)
```bash
# HTTP
tail /var/log/suricata/eve.json | jq 'select(.event_type=="http")'

# DNS
tail /var/log/suricata/eve.json | jq 'select(.event_type=="dns") | .dns.rrname'

# ICMP
tail /var/log/suricata/eve.json | jq 'select(.proto=="ICMP")'

# Top IP
tail -200 /var/log/suricata/eve.json | jq -r '.src_ip // .dest_ip' | sort | uniq -c | sort -nr
```

## Risultati tipici (dal mio lab)

### Top IP osservati
```text
  324 168.63.129.16  # Azure Guest Agent (normale)
   28 8.8.8.8        # Google DNS
    5 10.0.0.4       # La VM stessa
```

### Esempio log ICMP (ping reale)
```json
{
  "timestamp": "2026-01-14T16:38:00",
  "proto": "ICMP",
  "icmp": { "type": 8, "code": 0 },
  "src_ip": "10.0.0.4",
  "dest_ip": "8.8.8.8"
}
```

### Sintesi risultati
- 5970+ pacchetti processati
- Azure Agent (`168.63.129.16`) identificato come traffico legittimo
- ICMP, HTTP e DNS catturati correttamente
- Query jq pronte per uso in contesto SOC

## Hardening applicato
- NSG: SSH limitato al mio IP
- Costi: circa €0,01/ora (B1s)
- Cleanup: eliminazione del Resource Group a fine test

## Skills dimostrate

**Ruolo target: Junior SOC Analyst**

- Azure VM provisioning e configurazione NSG
- Deploy e gestione di Suricata IDS
- Log parsing (JSON con jq)
- Analisi protocolli di rete (ICMP, DNS, HTTP)
- Documentazione tecnica e riproducibilità del lab
```


