# Home SOC Lab — Wazuh SIEM

Ambiente de laboratório pessoal para estudo prático de monitoramento de segurança (SOC),
usando Wazuh como SIEM para detecção de eventos em tempo real.

## Objetivo

Simular um ambiente básico de SOC para praticar:

- Coleta e correlação de logs
- Detecção de tentativas de acesso não autorizado (SSH brute-force)
- Análise de alertas de segurança

## Arquitetura

- **VM 1 — Wazuh Server** (Ubuntu Server 22.04 LTS): Indexer + Manager + Dashboard
- **VM 2 — Vítima/Endpoint** (Ubuntu Server): Wazuh Agent monitorando eventos locais
- Rede interna isolada (VirtualBox Host-only, 192.168.56.0/24)

## Ferramentas utilizadas

- VirtualBox
- Wazuh 4.9.2 (Indexer, Server, Dashboard, Agent)
- Ubuntu Server

## O que foi feito

1. Provisionamento de duas VMs em rede isolada
2. Instalação e configuração do Wazuh Server (Indexer + Manager + Dashboard)
3. Instalação do Wazuh Agent no endpoint monitorado
4. Registro e conexão do agente ao servidor
5. Simulação de ataque de força bruta via SSH (tentativas de login falhadas)
6. Verificação da detecção do evento no Dashboard do Wazuh

## Evidências

### Agente conectado e ativo

![Agent Status](screenshots/03-dashboard-agents-active.png)

### Alerta de tentativa de SSH brute-force detectado

![SSH Alert](screenshots/05-alert-ssh-detected.png)

## Desafios técnicos enfrentados e resolvidos

Durante a montagem do ambiente, enfrentei e resolvi problemas reais de infraestrutura:

- Instabilidade de CPU (soft lockup) no Wazuh Indexer, resolvida ajustando a interface
  de paravirtualização do VirtualBox
- Incompatibilidade de versão entre agente e servidor (agente instalado via repositório
  trouxe versão mais recente que o manager), resolvida fixando a versão exata do pacote
- Configuração de rede interna (Host-only) entre as VMs via netplan
- Ajuste de `vm.max_map_count` exigido pelo OpenSearch/Indexer

## Próximos passos

- [ ] Criar regras de detecção customizadas
- [ ] Adicionar VM atacante (Kali) para gerar tráfego de reconhecimento (Nmap)
- [ ] Documentar relatório de incidente estruturado
- [ ] Testar detecção de alteração de arquivos (FIM)
