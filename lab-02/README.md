# LAB-02 — File Integrity Monitoring & Threat Intelligence

Laboratório do SOC_LAB focado em **File Integrity Monitoring (FIM)** e enriquecimento de eventos via **Threat Intelligence**, utilizando Wazuh e integração com VirusTotal.

## Cenário

Simulação do download de um arquivo de teste a partir de uma máquina atacante para uma máquina vítima monitorada pelo Wazuh, observando a detecção do FIM e o enriquecimento do evento via VirusTotal.

## Ambiente

| Máquina      | Função                                          |
| ------------ | ----------------------------------------------- |
| **Wazuh**    | SIEM — processamento e visualização dos eventos |
| **Victim**   | Endpoint Linux monitorado pelo agente Wazuh     |
| **Attacker** | Origem do arquivo — servidor HTTP local         |

Rede privada dedicada, seguindo a estrutura utilizada no LAB-01.

## Ferramentas

- Wazuh + Wazuh Dashboard
- Linux Ubuntu/Debian
- Kali Linux
- Python HTTP Server
- wget
- VirusTotal

## Objetivos

- Configurar o Syscheck/FIM para monitoramento em tempo real
- Simular o download de um arquivo de teste (EICAR)
- Detectar a criação do arquivo via FIM
- Integrar o Wazuh ao VirusTotal
- Validar a consulta de hash e o alerta enriquecido
- Compreender a diferença entre **detecção de integridade** e **classificação de ameaça**

## Execução

1. **FIM** — Syscheck configurado no agente vítima para monitorar o diretório de downloads em tempo real
2. **Servidor do atacante** — `python3 -m http.server` disponibilizando o arquivo EICAR (arquivo de teste padrão, não é malware real)
3. **Download** — máquina vítima baixa o arquivo via `wget`
4. **Detecção FIM** — Syscheck reporta a criação do arquivo ao Wazuh Manager (detecção de integridade, sem classificação de ameaça)
5. **Integração VirusTotal** — Wazuh Manager consulta o hash do arquivo detectado
6. **Alerta enriquecido** — VirusTotal retorna detecção positiva → Wazuh gera alerta pela regra `87105 — VirusTotal: Alert - positives`, com os campos `virustotal.positives`, `virustotal.malicious` e `virustotal.permalink`

## Resultado

Laboratório concluído com sucesso: o FIM detectou a criação do arquivo em tempo real, o hash foi consultado no VirusTotal, e o Wazuh gerou o alerta enriquecido com severidade elevada — demonstrando a evolução do evento desde a telemetria do endpoint até o enriquecimento por Threat Intelligence.

## Principais aprendizados

- Diferença entre **detecção de alteração** (FIM) e **classificação de ameaça** (Threat Intelligence)
- Configuração e troubleshooting de integrações do Wazuh
- Uso de APIs de Threat Intelligence e análise de hashes
- Interpretação de alertas enriquecidos
- Relação entre telemetria de endpoint e contexto externo

## Evidências

Armazenadas em [`evidences/`](./evidences):

| #   | Arquivo                                 | Descrição                                    |
| --- | --------------------------------------- | -------------------------------------------- |
| 01  | `01-baseline-dashboard.png`             | Baseline do Wazuh Dashboard                  |
| 02  | `02-expanded_baseline-dashboard.png`    | Baseline do Dashboard (expandido)            |
| 03  | `03-baseline-process I network.png`     | Baseline de processos e rede                 |
| 04  | `04-etapa-entrega.png`                  | Etapa de entrega do arquivo pelo atacante    |
| 05  | `05-final_result_3vms.png`              | Visão do ambiente com as três VMs            |
| 06  | `06-expanded_wazuh-alert-dashboard.png` | Alerta do Wazuh (expandido)                  |
| 07  | `07-wazuh_alert_dashboard.png`          | Alerta do Wazuh no Dashboard                 |
| 08  | `08-wazuh_FIM_detection_dashboard.png`  | Detecção do FIM no Dashboard                 |
| 09  | `09-kali_linux_malware_test_file.png`   | Arquivo de teste (EICAR) na máquina atacante |

> Evidências revisadas previamente para garantir que não contenham credenciais ou outras informações sensíveis.

---

📄 O relatório detalhado do incidente (troubleshooting, análise passo a passo) está em [`incident-report.md`](./incident-report.md).
