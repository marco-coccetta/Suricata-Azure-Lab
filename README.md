# Suricata NIDS Lab - Azure Security Monitoring

![Suricata](https://suricata.io/)
![Ubuntu](https://ubuntu.com/)
![Azure](https://azure.microsoft.com/)

## Obiettivo
Configurazione Suricata IDS su Azure VM per Network Security Monitoring e Threat Hunting.

## Architettura
Azure VM Ubuntu 22.04 LTS (Standard_B1s)
└── Suricata 7.0.3 → eve.json → jq analysis


## Installazione
```bash
sudo apt update && sudo apt install suricata jq -y
sudo suricata -c /etc/suricata/suricata.yaml -i eth0


## Traffico Catturato (Log Reali)

| Evento | Source IP | Dest IP | Protocollo | Dettagli
|-------- |----------- |--------- |------------|----------
| HTTP Health Check | 168.63.129.16 | 10.0.0.4 | TCP/80 | `/HealthService` WALinuxAgent
| Azure Status | 10.0.0.4 | 168.63.129.16 | TCP/32526 | `PUT /status`
| DNS Resolution | 10.0.0.4 | 168.63.129.16 | UDP/53 | VM hostname query

# DNS Analysis
sudo jq 'select(.event_type=="dns")' /var/log/suricata/eve.json

# HTTP Traffic
sudo jq 'select(.event_type=="http")' /var/log/suricata/eve.json

# IP Connections Count
sudo jq -r '.src_ip // .dest_ip' /var/log/suricata/eve.json | sort | uniq -c | sort -nr

#Risultati

5970 pacchetti analizzati

Azure Guest Agent (168.63.129.16) identificato

324 HTTP transactions, 28 DNS queries

#Competenze

Azure VM deployment & networking

Suricata IDS configuration

JSON log analysis (Eve format)

Threat hunting con jq
