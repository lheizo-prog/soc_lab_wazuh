# Incident Report — LAB-02

**Título:** Download de Artefato Suspeito e Detecção via FIM + Threat Intelligence
**Laboratório:** SOC_LAB — LAB-02 (File Integrity Monitoring & Threat Intelligence)
**Classificação:** Simulação controlada (ambiente de laboratório)
**Status:** Encerrado — Detecção confirmada

---

## 1. Resumo executivo

Foi simulada, em ambiente controlado, a entrega de um arquivo suspeito de uma máquina atacante para uma máquina vítima monitorada pelo Wazuh. O objetivo foi validar a capacidade de detecção de criação de arquivos (File Integrity Monitoring) e o enriquecimento desse evento por meio de uma integração de Threat Intelligence com o VirusTotal.

O incidente simulado foi detectado com sucesso em duas camadas: primeiro pelo FIM (Syscheck), que identificou a criação do arquivo no endpoint, e em seguida pela integração com o VirusTotal, que classificou o arquivo como malicioso e elevou a severidade do alerta.

## 2. Escopo e ambiente

| Ativo    | IP              | Papel no incidente                             |
| -------- | --------------- | ---------------------------------------------- |
| Wazuh    | `192.168.56.10` | SIEM — coleta, correlação e geração de alertas |
| Victim   | `192.168.56.20` | Endpoint afetado — download do arquivo         |
| Attacker | `192.168.56.30` | Origem do artefato — servidor HTTP local       |

Ambiente isolado em rede privada (Host-Only), sem exposição externa.

## 3. Artefato utilizado

- **Nome:** `eicar.txt`
- **Tipo:** EICAR Standard Anti-Virus Test File
- **Natureza:** Arquivo de teste padrão da indústria, utilizado para validar mecanismos de antivírus. Não é malware funcional, mas é reconhecido como malicioso por praticamente todos os engines de detecção — inclusive os agregados pelo VirusTotal.

## 4. Linha do tempo

| Etapa | Evento                                                                                                                      |
| ----- | --------------------------------------------------------------------------------------------------------------------------- |
| 1     | Servidor HTTP local iniciado na máquina Attacker (`python3 -m http.server`), disponibilizando o arquivo `eicar.txt`         |
| 2     | Máquina Victim realiza o download do arquivo via `wget`, simulando a ação de um usuário                                     |
| 3     | Syscheck (FIM), monitorando o diretório de downloads em tempo real, detecta a criação do arquivo e reporta ao Wazuh Manager |
| 4     | Evento de FIM aciona a integração configurada com o VirusTotal, associada ao grupo `syscheck`                               |
| 5     | Wazuh Manager envia o hash do arquivo para consulta na API do VirusTotal                                                    |
| 6     | VirusTotal retorna detecção positiva (arquivo reconhecido como malicioso)                                                   |
| 7     | Wazuh gera alerta enriquecido pela regra `87105` — _VirusTotal: Alert - positives_, com severidade elevada                  |

## 5. Detecção

### 5.1 Camada 1 — File Integrity Monitoring

O Syscheck, configurado com monitoramento em tempo real (`realtime="yes"`) no diretório de downloads da Victim, identificou a criação do arquivo imediatamente após o `wget`. Nesse estágio, o alerta gerado indicava apenas a alteração do sistema de arquivos ("file added"), sem qualquer classificação de ameaça — evidenciando a limitação do FIM isolado: ele detecta _que algo mudou_, não _se é malicioso_.

### 5.2 Camada 2 — Threat Intelligence (VirusTotal)

A integração configurada no Wazuh Manager consultou automaticamente o hash do arquivo detectado pelo FIM contra a base do VirusTotal. O retorno positivo resultou em um alerta enriquecido, contendo:

- `virustotal.positives` — número de engines que sinalizaram o arquivo como malicioso
- `virustotal.malicious` — indicador de classificação positiva
- `virustotal.permalink` — link para o relatório completo no VirusTotal

Esse alerta foi classificado pela regra `87105`, com severidade significativamente mais alta que o evento inicial de FIM.

## 6. Dificuldades no processo de detecção

Durante a configuração da integração, foram identificados e corrigidos os seguintes obstáculos (detalhados em [`troubleshooting.md`](./troubleshooting.md)):

- Bloco `<integration>` posicionado incorretamente dentro do `<syscheck>` no `ossec.conf`, impedindo a inicialização do processo `wazuh-integratord`
- Erro `403 — Check credentials` no primeiro alerta gerado, posteriormente identificado como falso positivo (a API key estava correta)
- Erro `invalid resource` em teste manual, causado por digitação incorreta de hash/API key devido à ausência de clipboard compartilhado entre as VMs
- Necessidade de forçar um novo evento de FIM (remoção e novo download do arquivo) para repetir o teste, já que o Syscheck não reemite alerta para um arquivo já indexado sem alteração

Nenhum desses obstáculos esteve relacionado à infraestrutura de detecção em si (Wazuh, Syscheck ou VirusTotal), e sim à configuração inicial e ao processo de validação manual.

## 7. Indicadores observados (IOCs do laboratório)

| Tipo                         | Valor                                   |
| ---------------------------- | --------------------------------------- |
| Nome do arquivo              | `eicar.txt`                             |
| Regra de FIM                 | Syscheck — evento de criação de arquivo |
| Regra de alerta (VirusTotal) | `87105` — VirusTotal: Alert - positives |
| Diretório monitorado         | `/home/usuario/Downloads` (Victim)      |

## 8. Análise e conclusão

O cenário demonstrou, de forma prática, a diferença entre **detecção de integridade** e **classificação de ameaça**:

- O FIM, isoladamente, é capaz de identificar _que_ um arquivo foi criado, mas não tem contexto para avaliar se ele representa risco.
- A integração com uma fonte externa de Threat Intelligence (VirusTotal) complementa essa lacuna, fornecendo uma classificação de reputação baseada em múltiplos engines de análise.

A combinação das duas camadas resultou em uma cadeia de detecção completa e coerente: **download → FIM detecta a criação → hash é consultado externamente → veredicto de ameaça → alerta de alta severidade**, validando a arquitetura de detecção proposta para o laboratório.

## 9. Evidências

Screenshots e registros do Dashboard estão disponíveis em [`evidences/`](./evidences), referenciados no [`README.md`](./README.md) do laboratório.

## 10. Recomendações (para ambientes reais)

Ainda que este seja um cenário de laboratório, as seguintes práticas seriam recomendadas em um ambiente de produção:

- Restringir o diretório de downloads monitorado a extensões/tipos de arquivo relevantes, reduzindo ruído no FIM
- Definir regras de resposta automática (active response) para isolar ou quarentenar arquivos com detecção positiva no VirusTotal
- Monitorar o limite de requisições da API (contas gratuitas do VirusTotal têm cota reduzida) para evitar lacunas de detecção em ambientes com maior volume de eventos
- Complementar a análise de reputação de hash com análise comportamental (sandboxing) para arquivos não previamente catalogados
