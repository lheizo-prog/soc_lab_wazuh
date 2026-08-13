# Troubleshooting — LAB-02

Durante a preparação e execução do LAB-02, foram identificadas dificuldades relacionadas à configuração de rede das VMs, ao auditd, à integração do Wazuh com o VirusTotal, e à preparação dos artefatos utilizados nos cenários.

## 1. Configuração da rede das VMs

**Problema**

Inicialmente, as máquinas virtuais não conseguiam se comunicar pela rede Host-Only.

A VM do Wazuh apresentava `192.168.56.10/24`, porém não possuía uma rota default. A Victim também não conseguia realizar ping para o Wazuh.

Durante a investigação, o comando `ip neigh` apresentava estados como `FAILED` e `INCOMPLETE`, indicando que a resolução ARP entre as máquinas não estava funcionando.

**Causa**

A configuração das interfaces virtuais das VMs estava incorreta. A interface destinada à comunicação Host-Only não estava adequadamente configurada/ativa. Foi identificado que a configuração de rede possuía duas interfaces: uma utilizada para NAT e outra destinada à rede Host-Only.

**Solução**

Foi mantida a primeira interface da VM em NAT, permitindo acesso externo, enquanto a segunda interface foi utilizada para a comunicação entre as máquinas através da rede Host-Only `192.168.56.0/24`.

**Resultado**

A comunicação entre Wazuh (`192.168.56.10`) e Victim (`192.168.56.20`) foi estabelecida com sucesso.

---

## 2. Configuração incorreta do auditd

**Problema**

Durante a criação da regra de auditoria, foram inseridas regras incorretas:

```
-w /tmp/system_check.sh -p X -k soc_lab_exec
-w /tmp/syste_check.sh -p x -k soc_lab_exec
```

Os problemas eram: utilização de `X` em vez de `x`, e erro de digitação no nome do arquivo (`syste_check.sh`).

**Solução**

As regras incorretas foram removidas e substituídas pela regra correta:

```
-w /tmp/system_check.sh -p x -k soc_lab_exec
```

A configuração foi verificada com `sudo auditctl -l`.

**Resultado**

O auditd passou a monitorar corretamente o arquivo `/tmp/system_check.sh` utilizando a chave `soc_lab_exec`.

---

## 3. Erro no caminho do audit.log no Wazuh

**Problema**

Durante a configuração do Wazuh Agent, foi identificada uma referência incorreta ao arquivo de log: `/var/log/audit/audit.og`.

O `wazuh-logcollector` registrava:

```
ERROR: (1103): Could not open file '/var/log/audit/audit.og'
due to [(2)-(No such file or directory)]
```

**Solução**

A referência incorreta foi corrigida:

```xml
<localfile>
  <location>/var/log/audit/audit.log</location>
  <log_format>audit</log_format>
</localfile>
```

O agente foi reiniciado com `sudo systemctl restart wazuh-agent`.

**Resultado**

O Logcollector passou a analisar `/var/log/audit/audit.log` sem depender do arquivo inexistente.

---

## 4. Diferença entre evento coletado e alerta

**Problema**

A ausência inicial de notificações no Dashboard levou à hipótese de que o Wazuh não estava recebendo os eventos do auditd.

**Análise**

Foi identificado um evento com `location: /var/log/audit/audit.log`, `decoder: auditd` e a regra `80730` ("Auditd: SELinux permission check"), confirmando que a cadeia de coleta estava funcionando:

```
auditd → audit.log → Wazuh Agent → Wazuh Manager → decoder auditd → regra → alerta
```

O evento, entretanto, não estava relacionado ao `system_check.sh` — tratava-se de uma atividade do `apparmor_parser`.

**Conclusão**

Foi estabelecida uma distinção importante entre evento presente no `audit.log`, evento coletado pelo Wazuh, evento decodificado, e evento que efetivamente aciona uma regra de alerta.

---

## 5. Eventos administrativos aparecendo no Dashboard

**Problema**

Diversas atividades realizadas com `sudo` durante a configuração do ambiente apareceram no Dashboard — por exemplo, um comando `grep` no `ossec.conf`, acionando a regra `5402` ("Successful sudo to ROOT executed").

**Análise**

O alerta não representava a ação ofensiva do laboratório; foi provocado por uma ação administrativa legítima durante a configuração do ambiente.

**Conclusão**

Esse comportamento confirmou que o Wazuh monitorava corretamente as atividades administrativas do endpoint, mas evidenciou a necessidade de diferenciar atividade de configuração de atividade relacionada ao cenário ofensivo. Para a documentação final, esses eventos são tratados como ruído operacional do processo de montagem, não como evidência do ataque simulado.

---

## 6. Execução malsucedida do primeiro artefato

**Problema**

A primeira versão do arquivo `/tmp/system_check.sh` estava incorreta. O auditd registrou a tentativa de execução (`key="soc_lab_exec"`, `syscall=59` / execve), mas com `success=no`.

**Análise**

Isso demonstrou que a regra do auditd estava funcionando — a tentativa de execução foi registrada. O problema estava no artefato em si, não na infraestrutura de auditoria.

**Solução**

O arquivo incorreto foi descartado e recriado seguindo o fluxo: Attacker → criação do `system_check.sh` → servidor HTTP → Victim → download → execução → auditd → Wazuh.

**Resultado**

A tentativa malsucedida foi mantida como parte do histórico do laboratório, mas não utilizada como evidência principal da execução ofensiva.

---

## 7. Limitação para análise com grep

**Problema**

Ao consultar `sudo grep -iE "audit|logcollector|error|warn" /var/ossec/logs/ossec.log`, o sistema retornou `grep: /var/ossec/logs/ossec.log: binary file matches`.

**Solução**

Utilizada a flag `-a`, forçando o tratamento do arquivo como texto:

```bash
sudo grep -aiE "audit|logcollector|error|warn" /var/ossec/logs/ossec.log
```

**Resultado**

Foi possível identificar diretamente os erros do `wazuh-logcollector`, incluindo o problema do caminho `audit.og`.

---

## 8. Tag `<integration>` posicionada incorretamente

**Problema**

Após configurar a integração do Wazuh Manager com o VirusTotal, o processo `wazuh-integratord` não aparecia ativo em `ps aux`. O log registrava:

```
wazuh-integratord: INFO: Remote integrations not configured. Clean exit.
```

**Causa**

A tag `<integration>` estava aninhada incorretamente **dentro** do bloco `<syscheck>`, em vez de estar em um nível direto de `<ossec_config>`.

**Solução**

O bloco `<integration>` foi movido para fora do `<syscheck>`:

```xml
<syscheck>
  ...
</syscheck>

<integration>
  <name>virustotal</name>
  <api_key>...</api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>
```

O manager foi reiniciado com `sudo systemctl restart wazuh-manager`.

**Resultado**

O processo `wazuh-integratord` passou a rodar corretamente, sem a mensagem de "Clean exit".

---

## 9. Erro "Check credentials" (HTTP 403) na integração VirusTotal

**Problema**

Após corrigir o posicionamento da tag, o primeiro alerta gerado pela integração retornou:

```
"description": "Error: Check credentials",
"error": "403"
```

**Análise**

O erro sugeria uma API key inválida. Para isolar a causa, a key foi testada manualmente contra a API pública do VirusTotal via `curl`.

**Resultado**

O teste manual confirmou que a API key era válida — o problema não estava na credencial em si, e sim em como o valor estava sendo digitado/comparado nos testes seguintes.

---

## 10. Erro "invalid resource" no teste manual da API

**Problema**

Ao testar a API do VirusTotal manualmente via `curl`, o retorno foi `invalid resource`, mesmo com a API key correta.

**Causa**

O hash utilizado como parâmetro `resource` estava sendo digitado manualmente (sem acesso a copiar/colar entre VMs), resultando em erro de digitação.

**Solução**

Em vez de digitar a API key e hashes manualmente, foi utilizada extração direta do valor salvo no arquivo de configuração, via variável de ambiente:

```bash
KEY=$(grep -oP '(?<=<api_key>).*(?=</api_key>)' /var/ossec/etc/ossec.conf)
curl -s "https://www.virustotal.com/vtapi/v2/file/report?apikey=$KEY&resource=<hash>"
```

**Resultado**

A consulta manual retornou o resultado correto, confirmando que tanto a API key quanto a integração estavam funcionais.

---

## 11. Repetição do evento de FIM

**Problema**

Após a primeira detecção, o FIM não gerava um novo evento para o mesmo arquivo sem alteração, impedindo repetir o teste da integração.

**Solução**

O arquivo foi removido e baixado novamente (`rm` seguido de novo `wget`), forçando um novo evento de criação no Syscheck.

**Resultado**

O novo evento disparou corretamente a consulta ao VirusTotal, retornando detecção positiva para o arquivo EICAR e gerando o alerta pela regra `87105` ("VirusTotal: Alert - positives").

---

## Síntese dos problemas encontrados

| Problema                                | Componente            | Situação                                |
| --------------------------------------- | --------------------- | --------------------------------------- |
| Comunicação entre VMs                   | VirtualBox/Rede       | Resolvido                               |
| Interface Host-Only não funcionando     | Rede                  | Resolvido                               |
| Regra auditd com `X` maiúsculo          | Auditd                | Resolvido                               |
| Caminho `syste_check.sh` incorreto      | Auditd                | Resolvido                               |
| Caminho `audit.og` incorreto            | Wazuh Agent           | Resolvido                               |
| Eventos administrativos no Dashboard    | Wazuh                 | Comportamento esperado                  |
| Eventos auditd chegando ao Wazuh        | Wazuh/Auditd          | Confirmado                              |
| Primeiro `system_check.sh` incorreto    | Artefato              | Identificado e corrigido                |
| Execução malsucedida do script          | Artefato              | Corrigido                               |
| Tag `<integration>` mal posicionada     | Wazuh Manager         | Resolvido                               |
| Erro 403 "Check credentials"            | Integração VirusTotal | Resolvido (falso positivo — key válida) |
| Erro "invalid resource" no teste manual | Integração VirusTotal | Resolvido                               |
| Repetição do evento de FIM              | Syscheck              | Resolvido                               |

---

## Estado atual do LAB-02

A infraestrutura principal está funcionando de ponta a ponta: comunicação entre VMs, coleta via auditd, integração com VirusTotal e geração de alertas enriquecidos foram validadas.

O cenário de download do arquivo de teste (EICAR) foi concluído com sucesso, com o Wazuh detectando a criação do arquivo via FIM, consultando o hash no VirusTotal e gerando o alerta correspondente.

O ponto que houve modificações, mas atualmente está concluído, é a recriação e execução bem-sucedida do `system_check.sh`. Depois de uma reavaliação, foi decidido criar um novo arquivo com um padrão de mercado de um Malware de teste para o cenário. Neste Lab, esse novo arquivo possui o nome de EICAR.tx e as documentações já estão atualizadas com esse novo nome, mas ainda há menções dessa antiga versão do Malware.