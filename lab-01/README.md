# LAB-01 — Authentication, Reconnaissance & Brute Force

## Sobre o laboratório

Este foi o primeiro laboratório desenvolvido no **SOC_LAB**, com foco em monitoramento de autenticação, reconhecimento de rede e detecção de tentativas de brute force.

O cenário foi construído em um ambiente controlado utilizando três máquinas virtuais: uma máquina com Wazuh responsável pelo monitoramento, uma máquina vítima e uma máquina atacante.

O objetivo foi observar como diferentes atividades realizadas contra o endpoint são registradas e posteriormente identificadas pelo Wazuh.

---

## Objetivos

- Validar o monitoramento da máquina vítima pelo Wazuh;
- Observar eventos de autenticação;
- Realizar reconhecimento controlado do endpoint;
- Simular tentativas de brute force;
- Analisar os eventos gerados durante a atividade;
- Identificar os alertas produzidos pelo Wazuh;
- Relacionar a atividade executada pelo atacante aos eventos observados na vítima.

---

## Ambiente

| Máquina  | Endereço IP     | Função                          |
| -------- | --------------- | ------------------------------- |
| Wazuh    | `192.168.56.10` | SIEM e monitoramento            |
| Victim   | `192.168.56.20` | Endpoint monitorado             |
| Attacker | `192.168.56.30` | Simulação de atividade ofensiva |

O laboratório utiliza uma rede privada criada especificamente para os experimentos.

---

## Ferramentas utilizadas

- Wazuh;
- Kali Linux;
- Nmap;
- Hydra;
- Wordlist personalizada;
- Máquinas virtuais.

---

## Cenário

O laboratório foi desenvolvido em etapas.

Primeiramente, foram realizadas tentativas manuais de autenticação com credenciais incorretas na máquina vítima. Essa etapa foi utilizada para observar os eventos relacionados a falhas de autenticação e verificar sua disponibilidade no Wazuh.

Em seguida, a máquina atacante realizou um reconhecimento controlado da vítima utilizando o **Nmap**, buscando identificar portas e serviços disponíveis no ambiente.

Após o reconhecimento, foi criada uma wordlist reduzida especificamente para o laboratório. Essa wordlist foi utilizada pelo **Hydra** para realizar tentativas controladas de brute force contra o serviço de autenticação identificado.

O cenário foi concluído com uma autenticação bem-sucedida, permitindo observar também os eventos relacionados ao acesso obtido.

---

## Etapa 1 — Tentativas de autenticação

Antes de utilizar a máquina atacante, foram realizadas tentativas manuais de autenticação utilizando credenciais incorretas.

O objetivo era verificar:

- se os eventos de falha de autenticação eram registrados;
- quais informações eram disponibilizadas pelo sistema;
- como esses eventos apareciam no Wazuh;
- se as regras de detecção existentes eram acionadas.

Essa etapa também serviu como uma validação inicial do funcionamento do monitoramento.

---

## Etapa 2 — Reconhecimento com Nmap

A partir da máquina atacante, foi realizado um reconhecimento controlado contra o endereço IP da vítima:

```text
192.168.56.20
```

O Nmap foi utilizado para identificar portas e serviços expostos no ambiente de teste.

O resultado do reconhecimento foi utilizado para determinar quais serviços poderiam ser analisados nas etapas seguintes.

A atividade foi realizada exclusivamente contra a máquina pertencente ao laboratório.

---

## Etapa 3 — Brute Force controlado

Após identificar o serviço de autenticação a ser testado, foi criada uma wordlist reduzida para o cenário.

A redução da wordlist teve como objetivo manter o teste controlado e adequado ao ambiente virtual utilizado.

O Hydra foi então utilizado para realizar tentativas automatizadas de autenticação contra a vítima.

O objetivo não era apenas descobrir a credencial, mas observar como a sequência de tentativas apareceria nos eventos coletados pelo Wazuh.

---

## Etapa 4 — Autenticação bem-sucedida

Durante o teste, uma das tentativas resultou em uma autenticação bem-sucedida.

Esse momento foi importante para comparar os diferentes eventos produzidos durante o cenário:

- falhas de autenticação;
- múltiplas tentativas;
- autenticação bem-sucedida;
- atividade originada pela máquina atacante.

A sequência permitiu analisar o incidente como um conjunto de eventos, em vez de observar cada alerta de maneira isolada.

---

## Monitoramento e detecção com Wazuh

Após a execução das atividades, os eventos foram analisados através da interface do Wazuh.

Foram observadas informações relacionadas à atividade de autenticação, incluindo dados como:

- endereço IP;
- horário;
- usuário;
- tipo de evento;
- resultado da autenticação;
- regra acionada;
- nível de severidade, quando disponível.

O objetivo foi relacionar os eventos registrados pelo endpoint com as ações executadas a partir da máquina atacante.

---

## Investigação

A análise do cenário foi realizada a partir da sequência dos eventos.

A investigação buscou responder:

1. Qual máquina originou as tentativas?
2. Qual máquina foi alvo?
3. Qual usuário estava sendo testado?
4. Quantas tentativas ocorreram?
5. Em que período ocorreram?
6. Houve uma autenticação bem-sucedida?
7. Quais alertas foram gerados pelo Wazuh?
8. É possível relacionar os diferentes eventos ao mesmo cenário?

Essa abordagem permitiu começar a desenvolver uma visão de investigação baseada em evidências, em vez de apenas observar individualmente cada alerta.

---

## Resultado

O laboratório foi concluído com sucesso.

Foi possível:

- gerar eventos reais de autenticação no ambiente de teste;
- realizar reconhecimento controlado da máquina vítima;
- executar um cenário de brute force com uma wordlist personalizada;
- obter uma autenticação bem-sucedida;
- visualizar os eventos relacionados no Wazuh;
- analisar os alertas produzidos durante o cenário.

O laboratório demonstrou, na prática, a relação entre **atividade ofensiva, geração de eventos no endpoint e detecção por meio de um SIEM**.

---

## Principais aprendizados

O principal aprendizado deste laboratório foi compreender que uma atividade de segurança não deve ser analisada apenas pelo alerta final.

Uma tentativa de brute force, por exemplo, pode ser observada como uma sequência de acontecimentos que envolve reconhecimento, múltiplas tentativas de autenticação e, eventualmente, um acesso bem-sucedido.

Isso reforçou a importância de analisar:

- contexto;
- sequência temporal;
- origem dos eventos;
- sistema afetado;
- usuário envolvido;
- relação entre diferentes registros.

Também foi possível compreender melhor o papel de um SIEM no processo de monitoramento e investigação.

---

## Evidências

As evidências relacionadas ao laboratório estão armazenadas no diretório `evidence/`.

Entre os registros podem ser incluídos:

- screenshots do Wazuh;
- resultados do reconhecimento;
- eventos de autenticação;
- alertas relacionados ao brute force;
- demais evidências relevantes para a análise.

As evidências publicadas devem ser previamente revisadas para garantir que não contenham credenciais ou outras informações sensíveis.

---

## Conclusão

O LAB-01 estabeleceu a base prática do SOC_LAB para monitoramento e investigação de atividades relacionadas a autenticação.

A experiência permitiu observar todo o processo, desde a atividade executada pela máquina atacante até a geração e análise dos eventos no Wazuh.

Os próximos laboratórios irão ampliar esse conhecimento para outras categorias de telemetria, especialmente atividades executadas diretamente no endpoint, permitindo evoluir de uma análise centrada em autenticação para uma investigação mais ampla do comportamento de sistemas monitorados.
