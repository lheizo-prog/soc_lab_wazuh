# SOC_LAB

## Sobre o projeto

O **SOC_LAB** é um laboratório prático desenvolvido para estudar conceitos de **Security Operations Center (SOC)**, monitoramento de endpoints, detecção de atividades suspeitas e investigação de incidentes em um ambiente controlado.

O projeto utiliza máquinas virtuais para representar diferentes componentes de uma infraestrutura monitorada, permitindo simular atividades ofensivas e observar como essas ações são registradas, detectadas e analisadas.

O principal objetivo é transformar conhecimentos teóricos de Cybersecurity em experiências práticas e documentadas.

---

## Objetivos

O projeto busca desenvolver conhecimentos práticos em:

- Monitoramento de endpoints;
- Análise de logs;
- SIEM;
- Detecção de atividades suspeitas;
- Investigação de alertas;
- File Integrity Monitoring (FIM);
- Threat Intelligence;
- Análise de autenticação;
- Reconhecimento de rede;
- Brute force em ambiente controlado;
- Correlação de eventos;
- Resposta a incidentes;
- Fundamentos de DFIR.

Além do resultado dos experimentos, o projeto também documenta dificuldades encontradas durante a configuração do ambiente, decisões tomadas e aprendizados obtidos durante cada laboratório.

---

## Ambiente

O laboratório é composto por três máquinas virtuais conectadas em uma rede privada criada especificamente para os experimentos.

| Máquina  | Endereço IP     | Função                            |
| -------- | --------------- | --------------------------------- |
| Wazuh    | `192.168.56.10` | Monitoramento e SIEM              |
| Victim   | `192.168.56.20` | Endpoint monitorado               |
| Attacker | `192.168.56.30` | Simulação de atividades ofensivas |

Os endereços utilizados pertencem ao ambiente isolado do laboratório.

A infraestrutura pode ser modificada conforme novos cenários forem adicionados.

---

## Tecnologias e ferramentas

### Monitoramento e análise

- Wazuh
- Wazuh Dashboard
- SIEM
- Logs de sistema
- Event monitoring
- File Integrity Monitoring (Syscheck)

### Threat Intelligence

- VirusTotal

### Segurança e simulação

- Kali Linux
- Nmap
- Hydra
- Python HTTP Server

### Sistema operacional e infraestrutura

- Linux
- Windows
- Máquinas virtuais
- Rede privada de laboratório

Novas ferramentas poderão ser incorporadas conforme a complexidade dos cenários aumentar.

---

## Organização do projeto

Cada laboratório possui uma finalidade específica e é documentado separadamente.

```text
SOC_LAB/
│
├── README.md
│
└── labs/
    │
    ├── 01-authentication-bruteforce/
    │   ├── README.md
    │   ├── evidences/
    │   ├── troubleshooting.md
    │   └── incident-report.md
    │
    └── 02-file-integrity-threat-intelligence/
        ├── README.md
        ├── troubleshooting.md
        ├── incident-report.md
        └── evidences/
```

A estrutura será expandida conforme novos laboratórios forem desenvolvidos.

---

## Laboratórios

| Laboratório | Tema                                            | Status    |
| ----------- | ----------------------------------------------- | --------- |
| LAB-01      | Authentication, Reconnaissance & Brute Force    | Concluído |
| LAB-02      | File Integrity Monitoring & Threat Intelligence | Concluído |

Os laboratórios serão adicionados progressivamente, sempre buscando introduzir uma nova capacidade de monitoramento ou investigação.

---

## Metodologia

Os experimentos seguem um processo prático de investigação:

**Preparação → Atividade controlada → Geração de eventos → Detecção → Investigação → Conclusão → Documentação**

A intenção não é apenas verificar se uma ferramenta gera um alerta. O objetivo é compreender o evento e determinar, a partir das evidências disponíveis, **o que aconteceu, quando aconteceu, qual sistema foi afetado e quais elementos permitem sustentar a conclusão**.

Quando uma atividade não gera o resultado esperado, o processo de troubleshooting também é documentado.

---

## LAB-01 — Authentication, Reconnaissance & Brute Force

O primeiro laboratório concentrou-se em atividades relacionadas a autenticação e reconhecimento.

Foram realizados:

- Tentativas manuais de autenticação com credenciais incorretas;
- Reconhecimento da máquina vítima utilizando Nmap;
- Criação de uma wordlist reduzida para o ambiente de teste;
- Tentativas controladas de brute force utilizando Hydra;
- Observação do sucesso da autenticação;
- Análise dos eventos e alertas gerados pelo Wazuh.

O laboratório permitiu estudar a relação entre uma atividade realizada pelo atacante, os eventos gerados no endpoint e a detecção realizada pelo SIEM.

---

## LAB-02 — File Integrity Monitoring & Threat Intelligence

O segundo laboratório amplia o escopo do projeto para **telemetria de arquivos e enriquecimento de eventos por meio de Threat Intelligence**.

O cenário simulou o download de um arquivo de teste (EICAR) a partir de uma máquina atacante para uma máquina vítima monitorada pelo Wazuh, observando:

- configuração do Syscheck/FIM para monitoramento em tempo real;
- detecção da criação do arquivo no endpoint;
- diferença entre detecção de integridade e classificação de ameaça;
- integração do Wazuh com o VirusTotal;
- consulta automática do hash do arquivo;
- geração de um alerta enriquecido a partir do resultado da integração.

O laboratório também documentou o processo de troubleshooting da integração (posicionamento incorreto do bloco `<integration>` no `ossec.conf`, validação de credenciais e reprodução do evento de FIM).

---

## Princípios do laboratório

Os cenários são executados em ambiente próprio e controlado.

As atividades ofensivas têm finalidade exclusivamente educacional e são realizadas contra máquinas pertencentes ao laboratório.

O projeto busca priorizar:

- aprendizado prático;
- documentação;
- reprodutibilidade;
- análise baseada em evidências;
- troubleshooting;
- progressão gradual de dificuldade.

---

## Segurança e privacidade

Os endereços IP utilizados no projeto pertencem à rede privada criada especificamente para o laboratório.

Informações sensíveis, credenciais, tokens, chaves privadas e outros segredos não devem ser armazenados no repositório.

Screenshots, logs e evidências publicadas devem ser revisados antes do envio ao GitHub para evitar exposição de informações não relacionadas ao ambiente de testes.

---

## Evolução planejada

A evolução do SOC_LAB será baseada na introdução gradual de novas categorias de eventos e investigação.

Entre os temas planejados estão:

- Process execution;
- PowerShell e command execution;
- User and privilege changes;
- Persistence;
- Suspicious network activity;
- Event correlation;
- Incident investigation;
- Incident response;
- Digital forensics.

A prioridade será aumentar a capacidade de **investigar e correlacionar evidências**, e não simplesmente adicionar novas ferramentas.

---

## Objetivo de aprendizado

Este projeto faz parte de uma formação prática em Cybersecurity, com foco especial em **SOC, Security Monitoring, Incident Investigation e, futuramente, DFIR**.

A proposta é construir uma sequência de experiências em que cada laboratório represente uma competência nova, permitindo acompanhar a evolução técnica por meio de resultados concretos, evidências e documentação.

---

## Status

**Em desenvolvimento.**

Novos laboratórios serão adicionados conforme o ambiente evoluir e novos conceitos forem estudados.
