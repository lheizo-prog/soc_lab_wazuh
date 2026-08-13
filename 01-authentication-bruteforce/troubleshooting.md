# Troubleshooting — LAB-01

Durante a preparação e execução do **LAB-01 — SOC com Wazuh**, foram identificadas algumas dificuldades relacionadas à geração dos eventos ofensivos, à utilização das ferramentas de ataque e à interpretação dos alertas apresentados pelo Wazuh.

## 1. Geração de tentativas de autenticação

### Problema

Durante a construção do cenário, foi necessário gerar tentativas de autenticação malsucedidas para produzir os eventos que seriam posteriormente analisados pelo Wazuh.

A dificuldade principal foi garantir que as tentativas fossem efetivamente registradas pelo sistema e posteriormente disponibilizadas para o monitoramento.

### Solução

Foram realizadas tentativas de autenticação contra o serviço disponível na máquina Victim, verificando posteriormente os eventos registrados pelo sistema e pelo Wazuh.

### Resultado

As tentativas malsucedidas passaram a ser identificadas pelo Wazuh, permitindo utilizar esses eventos como parte da atividade suspeita do laboratório.

---

## 2. Configuração do brute force

### Problema

Para representar uma tentativa de ataque de força bruta, foi necessário utilizar uma wordlist adequada para o cenário.

Uma wordlist muito extensa aumentaria desnecessariamente o número de tentativas e dificultaria a análise do laboratório.

### Solução

Foi utilizada uma **wordlist reduzida**, contendo credenciais suficientes para representar o ataque sem gerar uma quantidade excessiva de eventos.

A ferramenta utilizada para a tentativa de brute force foi o **Hydra**.

### Resultado

O ataque produziu múltiplas tentativas de autenticação, permitindo observar o comportamento do Wazuh diante de uma sequência de falhas.

---

## 3. Identificação do ataque pelo Wazuh

### Problema

Uma sequência individual de tentativas de autenticação pode não representar necessariamente um ataque de força bruta.

Foi necessário analisar a quantidade e a repetição dos eventos para diferenciar uma falha isolada de um padrão suspeito.

### Solução

A sequência de tentativas produzida pelo Hydra foi analisada no Dashboard do Wazuh, observando:

- quantidade de eventos;
- origem das tentativas;
- usuário utilizado;
- serviço alvo;
- repetição das falhas;
- regras acionadas pelo Wazuh;
- nível dos alertas.

### Resultado

O Wazuh conseguiu identificar o padrão de tentativas como atividade compatível com **brute force**, permitindo sua utilização como evidência principal do cenário ofensivo.

---

## 4. Sucesso de autenticação após o brute force

### Problema

O laboratório precisava demonstrar não apenas as tentativas de força bruta, mas também o resultado do ataque.

Era necessário verificar se o sucesso posterior de autenticação seria registrado separadamente das tentativas malsucedidas.

### Solução

Após as tentativas de força bruta, foi realizada uma autenticação utilizando a credencial que havia sido encontrada durante o cenário.

O evento de autenticação bem-sucedida foi posteriormente analisado no Wazuh.

### Resultado

Foi possível estabelecer a sequência:

```text
Tentativas de autenticação
        ↓
Brute force
        ↓
Credencial encontrada
        ↓
Autenticação bem-sucedida
```
