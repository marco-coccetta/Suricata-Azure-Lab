#Suricata NIDS Lab - Azure Security Monitoring

[![Suricata](https://img.shields.io/badge/Suricata-7.0.3-orange?style=flat&logo=suricata)](https://suricata.io/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20LTS-E95420?style=flat&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Azure](https://img.shields.io/badge/Azure-Cloud-0078D4?style=flat&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/)

##Obiettivo
Configurazione pratica di **Suricata IDS (Intrusion Detection System)** su una Virtual Machine Azure per implementare Network Security Monitoring (NSM), analizzare traffico reale e simulare attività di Threat Hunting tramite log JSON.

##Architettura
*   **Infrastructure**: Azure VM (Standard_B1s)
*   **OS**: Ubuntu 22.04 LTS
*   **IDS Engine**: Suricata 7.x (monitoring su interfaccia `eth0`)
*   **Logging**: EVE JSON format (analizzato tramite `jq`)

##Installazione & Setup
Comandi principali utilizzati per il deployment:

```bash
# Aggiornamento sistema e installazione pacchetti
sudo apt update && sudo apt install suricata jq -y

# Verifica versione e build info
suricata --build-info

# Avvio motore IDS su interfaccia specifica
sudo suricata -c /etc/suricata/suricata.yaml -i eth0



#  Analisi del Traffico (Log Reali)
Esempio di traffico intercettato e analizzato dai log eve.json:

| Evento            | Source IP     | Dest IP       | Protocollo | Dettagli                         |
| ----------------- | ------------- | ------------- | ---------- | -------------------------------- |
| HTTP Health Check | 168.63.129.16 | 10.0.0.4      | TCP/80     | /HealthService (WALinuxAgent)    |
| Azure Status      | 10.0.0.4      | 168.63.129.16 | TCP/32526  | PUT /status (Host communication) |
| DNS Resolution    | 10.0.0.4      | 168.63.129.16 | UDP/53     | Internal hostname query          |


# Comandi di Threat Hunting (JQ)
Esempi di query utilizzate per estrarre intelligence dai log grezzi:

# Filtrare solo traffico DNS
sudo jq 'select(.event_type=="dns")' /var/log/suricata/eve.json

# Filtrare traffico HTTP
sudo jq 'select(.event_type=="http")' /var/log/suricata/eve.json

# Statistiche: Top Talkers (Conteggio connessioni per IP)
sudo jq -r '.src_ip // .dest_ip' /var/log/suricata/eve.json | sort | uniq -c | sort -nr

#Risultati Ottenuti
5970+ pacchetti analizzati correttamente.

Identificato traffico di servizio Azure Guest Agent (non malevolo, ma rumoroso).

324 transazioni HTTP e 28 query DNS tracciate.

#Competenze Acquisite
Cloud Security: Deployment e hardening base di Azure VM.

Network Forensics: Analisi pacchetti e flussi di rete.

Log Analysis: Parsing di log JSON complessi con strumenti CLI (jq).

IDS Tuning: Comprensione del "rumore" di fondo in ambiente cloud.


