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

Dopo l’incolla:
1. Premi **Commit changes**.
2. Vai in tab **Preview**: vedrai solo titolo, badge, sezioni pulite e codice formattato, senza il testo “Ecco la versione corretta…”.

[1](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/108580053/725668ff-579a-47b7-82ff-1aef351b8fc4/Schermata-del-2026-01-14-19-06-20.jpg?AWSAccessKeyId=ASIA2F3EMEYEZ4JDXLDL&Signature=YybORzCOLDX0iyZ%2F8q1uxQsiiZo%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEFoaCXVzLWVhc3QtMSJHMEUCIQDTTCbWoVqsXxI7XaK%2F%2BFGtpIBPEklgdScrTIB7f6UC8wIgVFSbwo3RUMOVW0qXVRZKUjS3Ms9fZ9tXJ%2BdY1NO0lnQq8wQIIxABGgw2OTk3NTMzMDk3MDUiDD%2FOro1B3pzM3f4XVirQBPZ0F61iqT5KoVWF3JEsDk1UKODbWzgE2AhLyps5O24MkLtU8V9fWiC3TqEr5iWgUokPTfQCoEznvfQrLONqwgEGPG9SbCqg0wFELK0yIZdXLiZDESKfButdd4qyiEFbo1mLAjKwFLbktv9T1C9mE0%2B2CeLOg1xy4WOcXjBW2Q7GI7eqj5Wy%2Fr%2BHFLXv7L94TaN1FHWt%2BF7HLzz29kye8E7oay%2F%2F0XybMsnOTIZEk5xMCbNVWcKmAeNBaO5pxhfV6zJeBiGz4a85Q4FR2SPcqygIFcp0eyPV94o9LzosyLYNOLOavqRdPHS7R4tCasak6uuz4fZov6W%2FppxfODjYdoeh%2Bm6Ga1jLvAXaRtEs6trEGj0reAGyfSqpc95%2FclN92llMo7MeDyZ16UNz7wgCDf977Nsw9sgyWXH1YiOCATb3FoavoeHc%2BxyKFATtJDxEtGKlj6Y7oxjxeKwY9XMTkbIrHp6Tr27hIOVouffk6u8fRYY7viAbdNaQJyMeVSiAD2J2PfOw77j3ppkT1SOl7bJOX8gentPAP%2FedXb1wiMK45t0xiEG0M%2BlFOGPOTHAO1SaftcB0RmrkeYMUoWRDrWrZjlq06Ax%2F73otOQS2SiM6C1D2aem6yf4Cr8XXO6Gz7tqm8zXBBxTadWGDYGlV2tVe8MppuQJeFmmiElE7HZQEFwSfXyBKAbNweNB14Ms%2F0GZOm0u%2FWntGemdNTc8KNseA3Z5LnG4HnagUpak7cLHdC1PigsipUA0Ug2f98iYQqjPYF8gE%2BslJI%2Fnw46GGn%2BIwkaSfywY6mAGKCUPwn3jLskl%2FMSi8Q327O5FD%2BtKS0PIlOJNcQmSM2hpfrVl4HanM8YjvaKllK4rk%2FAWtdq0%2BKApvtYoqh6eFiJQHbQ3VwDmr0PTEkcGfs5sS60tvzox8GeRYk3fDrF3IswAH3Pnu1UJuMcDZiGJYVm%2BZ0qyZcNA7OTCK%2FeRZOGGdSmpXECNtrnjBmrCtmZAPUiZpxDAyfA%3D%3D&Expires=1768414638)
[2](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/108580053/81b61d8c-4c42-4329-a72e-12e0c8bd86e7/CV.pdf)
[3](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/108580053/0c3831da-80c4-4b7e-bdc7-65c973eca44f/image.jpg)
[4](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/108580053/acb1501d-8e84-4b1e-bdfc-2ef0664a4130/image.jpg)
[5](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/108580053/da150abc-c968-43a5-a79c-feadf88dbe56/image.jpg)
[6](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/108580053/d24c2c1d-a989-4824-a852-f56dc970f2a0/Screenshot-from-2026-01-13-16-18-36.jpg)
[7](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/108580053/4c367647-1712-4eb7-8127-9db3156cac05/image.jpg)
[8](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/108580053/869fcbd4-b0ed-455b-9864-89d1cbd5428d/image.jpg)
