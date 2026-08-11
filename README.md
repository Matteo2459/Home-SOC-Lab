# Home SOC Lab: Monitoraggio Rete e SIEM 
In questa repository ho documentato la progettazione, la configurazione e il testing di un ambiente di laboratorio SOC aziendale su scala ridotta. L'obiettivo del laboratorio è centralizzare la visibilità della sicurezza a livello perimetrale di rete (Firewall e IDS).

## Architettura di Rete del Laboratorio
L'infrastruttura è divisa in due componenti fondamentali che comunicano costantemente tra loro:

Centralizzatore dei Log (SIEM): Wazuh Server installato su una macchina virtuale dedicata con sistema operativo Debian 13.

Difesa Perimetrale e IDS: Firewall pfSense che gestisce il traffico di rete, con il sistema di rilevamento delle intrusioni Suricata attivo sulle interfacce di transito.

## Network Security e Log Management (pfSense e Suricata)
Il laboratorio si concentra sulla visibilità del traffico di rete e  sul fare in modo che i dispositivi di rete inviino i dati al SIEM in un formato comprensibile.

### Il Flusso Logico della Pipeline di Rete
Rilevamento sul Firewall: Il motore IDS di Suricata analizza i pacchetti in transito su pfSense eseguendo una Deep Packet Inspection (DPI).

Generazione del Log: Se un pacchetto viola una regola di sicurezza, Suricata genera un evento. Il sistema pfSense prende questo evento e lo scrive nel registro Syslog locale.

Spedizione Remota (Log Forwarding): pfSense è configurato per inoltrare immediatamente questi log attraverso la rete locale, sfruttando il protocollo standard Syslog sulla porta UDP 514, puntando direttamente all'IP di Wazuh Server.

Ricezione e Decodifica (Parsing): Il server Wazuh rimane in ascolto sulla porta UDP 514. Quando arriva il flusso di testo da pfSense, Wazuh utilizza i suoi decodificatori interni (Decoders) per separare i dati utili, come l'IP di origine, l'IP di destinazione e il tipo di minaccia, trasformando il testo grezzo in un evento leggibile.

## Test 1: Monitoraggio dei Tentativi di Accesso (Authentication Failure)
Ho simulato un tentativo di accesso non autorizzato eseguendo un login con credenziali errate sul pannello di controllo web di pfSense.

Il sistema ha risposto nel modo seguente:
pfSense ha registrato il fallimento e ha spedito il log a Wazuh. Il SIEM ha intercettato l'evento associandolo alla regola ID 2501 (syslog: User authentication failure). 


<img width="1905" height="1010" alt="Screenshot 2026-06-06 184457" src="https://github.com/user-attachments/assets/6c14c4ca-9fd9-4c30-a50d-ed098b0c3996" />
<br>
<br>

## Test 2: Rilevamento Anomalie di Rete con Suricata
Per testare la componente IDS, ho monitorato il traffico di rete standard evidenziando come Suricata intercetti i pacchetti TCP anomali, non validi o potenzialmente malevoli che tentano di superare il firewall.

Il sistema ha risposto nel modo seguente:
Wazuh ha raccolto gli allarmi generati da Suricata . L'indicizzazione di questi eventi permette  correlare i picchi di traffico anomalo con potenziali attività di scansione o ricognizione (Port Scanning) da parte di malintenzionati esterni.
<br>
<img width="1813" height="984" alt="Screenshot 2026-06-06 191606" src="https://github.com/user-attachments/assets/64222980-5cab-43fe-a156-3e5875274c7e" />

