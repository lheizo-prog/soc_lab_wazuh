# Relatório de Incidente — Tentativa de Acesso não Autorizado via SSH

## Resumo

Durante testes no ambiente de laboratório SOC-Lab-Wazuh, foi simulada uma tentativa
de acesso não autorizado via SSH ao host `victmlab` (192.168.56.20), com o objetivo
de validar a capacidade de detecção do SIEM.

## Linha do tempo

| Horário (UTC) | Evento                                                  |
| ------------- | ------------------------------------------------------- |
| 18:21:XX      | Tentativa de conexão SSH com usuário inválido (`jorge`) |
| 18:21:XX      | Conexão recusada por falha de autenticação de chave     |
| 18:21:XX      | Novas tentativas com usuário `usuario_invalido`         |
| 18:21:XX      | 3 tentativas consecutivas de senha incorreta            |
| 18:21:XX      | Wazuh Agent reporta os eventos ao Manager               |
| 18:21:XX      | Alerta gerado no Dashboard (Threat Hunting)             |

_(ajuste os horários exatos com base nos logs reais que você tem)_

## Detecção

O Wazuh Agent instalado no host vítima monitorou os logs de autenticação (`/var/log/auth.log`
via `wazuh-logcollector`) e reportou as tentativas de login falhadas ao Wazuh Manager.
O evento foi classificado pela regra correspondente a falhas de autenticação SSH, gerando
um alerta visível no Dashboard.

**Evidência:**

![Alerta no Dashboard](../screenshots/05-alert-ssh-detected.png)

![Detalhe do alerta](../screenshots/06-alert-detail-expanded.png)

## Análise

- **Origem:** localhost (simulação interna, sem origem externa real)
- **Usuários testados:** `jorge`, `usuario_invalido` (nenhum existente no sistema)
- **Resultado:** todas as tentativas falharam (comportamento esperado)
- **Severidade classificada pelo Wazuh:** _(preencher com o nível que apareceu, ex: level 5)_
- **Regra disparada:** _(preencher com o rule.id / rule.description que apareceu no alerta expandido)_

## Ação de resposta recomendada

Em um ambiente de produção, a resposta a esse tipo de evento normalmente incluiria:

1. Verificar se as tentativas persistem (possível indicativo de brute-force automatizado)
2. Bloquear temporariamente o IP de origem via firewall/fail2ban após N tentativas
3. Revisar se o usuário-alvo existe e notificar o responsável, caso positivo
4. Monitorar por escalonamento (tentativas em outras contas/portas)

## Conclusão

O teste validou que o pipeline de detecção do ambiente está funcional: geração do evento
na origem → coleta pelo agente → processamento pelo servidor → alerta visível no dashboard.
Isso confirma a viabilidade do ambiente como ferramenta de estudo prático para SOC.
