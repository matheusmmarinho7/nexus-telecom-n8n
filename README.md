# Nexus Telecom — Triagem Automática de E-mails com IA

Sistema de triagem inteligente de e-mails de suporte construído em n8n, com classificação por GPT-4o-mini, roteamento automático para canais Slack e registro em Google Sheets.

---

## Visão geral

Provedoras de internet e empresas de suporte recebem dezenas de e-mails por dia de clientes com problemas técnicos, dúvidas financeiras e solicitações de cancelamento. Sem triagem, tudo cai na mesma caixa e o tempo de resposta aumenta.

Este workflow resolve isso automaticamente:

1. Detecta novos e-mails na caixa do Gmail
2. Filtra e-mails inválidos antes de processar
3. Classifica o conteúdo com GPT-4o-mini
4. Gera um ticket com ID único e prazo de resposta
5. Registra no Google Sheets para controle
6. Notifica o canal Slack correto por categoria
7. Responde automaticamente ao cliente no Gmail

---

## Fluxo completo

```
Gmail Trigger
    ↓
Filtro - Email Válido (IF)
    ├── inválido → Sheets - Descartados
    └── válido
         ↓
    OpenAI - Classificador (GPT-4o-mini)
         ↓
    Normalizar Dados (Code JS)
         ↓
    Filtro - Sucesso IA (IF)
         ├── sucesso: false → Slack - Erro IA
         └── sucesso: true
              ↓
         Sheets - Tickets
              ↓
         Switch (categoria)
              ├── tecnico      → Slack - Técnico      → Gmail - Resposta Auto
              ├── financeiro   → Slack - Financeiro   → Gmail - Resposta Auto
              ├── cancelamento → Slack - Cancelamento → Gmail - Resposta Auto
              ├── elogio       → Slack - Elogio       → Gmail - Resposta Auto
              └── outro        → Ignorar - Outro
```

---

## Tratamento de erros — 5 camadas

### Camada 1 — Filtro pré-AI
Bloqueia e-mails inválidos antes de consumir tokens da OpenAI.

Condições verificadas:
- Corpo do e-mail não está vazio
- Remetente não é automático (`noreply`, `no-reply`, `mailer-daemon`, `automated`, `donotreply`)
- Assunto não está vazio

E-mails bloqueados são registrados na aba `DESCARTADOS` do Sheets com data, remetente, assunto e motivo.

### Camada 2 — Retry automático no OpenAI
Configurado com 3 tentativas e 2 segundos de intervalo entre elas. Se a API da OpenAI retornar timeout, erro 429 ou qualquer falha, o n8n tenta automaticamente antes de desistir. Após 3 falhas consecutivas, o erro sobe para o Error Trigger global.

### Camada 3 — Try-catch no Code JS
Toda a lógica de normalização está dentro de um bloco `try-catch`. Se a IA retornar JSON malformado ou campos inesperados, o `catch` retorna um objeto padrão com `sucesso: false` em vez de quebrar o fluxo.

### Camada 4 — Rota de erro por sucesso=false
O nó `Filtro - Sucesso IA` verifica o campo `sucesso` antes de gravar no Sheets. Tickets com `sucesso: false` são desviados para o canal `#erros-nexus` no Slack com remetente, assunto, ID do ticket e mensagem de erro — sem perder o rastro do e-mail.

### Camada 5 — Error Trigger global
Workflow separado (`Nexus - Error Handler`) vinculado nas configurações do workflow principal. Captura qualquer falha não tratada em qualquer nó — Sheets offline, Slack fora do ar, credencial expirada — e notifica via Slack com nome do nó, mensagem de erro e horário.

---

## Categorias de classificação

| Categoria | Exemplos |
|---|---|
| `tecnico` | Internet lenta, sem conexão, queda de sinal, problema no roteador |
| `financeiro` | Boleto, fatura, cobrança errada, segunda via |
| `cancelamento` | Cancelar plano, rescisão, encerrar contrato |
| `elogio` | Agradecimento, satisfação, elogio ao atendimento |
| `outro` | Qualquer assunto fora das categorias acima |

---

## Campos do ticket gerado

| Campo | Descrição |
|---|---|
| `id_ticket` | ID único no formato `TKT-XXXXXX` |
| `data_hora` | Timestamp no fuso America/Sao_Paulo |
| `nome_cliente` | Primeiro nome extraído do remetente |
| `email_cliente` | Endereço de e-mail do remetente |
| `assunto_original` | Assunto do e-mail recebido |
| `categoria` | Classificação da IA |
| `urgencia` | `alta`, `media` ou `baixa` |
| `resumo_ia` | Resumo em até 15 palavras gerado pela IA |
| `prazo_resposta` | `até 2 horas`, `até 4 horas` ou `até 1 dia útil` |
| `thread_id` | ID da thread do Gmail para rastreamento |
| `sucesso` | `true` se a IA processou corretamente, `false` se houve falha |

---

## Stack

- **n8n** — orquestração do workflow
- **OpenAI GPT-4o-mini** — classificação e extração de dados
- **Google Sheets** — log de tickets e descartados
- **Slack** — notificações por categoria e alertas de erro
- **Gmail** — trigger de entrada e resposta automática
- **Oracle Cloud Free Tier** — infraestrutura (Ubuntu 22.04, Docker)

---

## Como importar

1. Baixe o arquivo `Nexus_Telecom.json`
2. No n8n, clique em **+** → **Import from file**
3. Configure as credenciais: Gmail, OpenAI, Google Sheets, Slack
4. Ajuste o ID da planilha Google Sheets nas configurações do nó `Sheets - Tickets`
5. Crie um workflow separado com o nó `Error Trigger` e vincule nas Settings → Error Workflow
6. Ative o workflow

---

## Autor

**Matheus Marques**
Automation Builder | n8n · Claude Code · Agentes de IA | Background Industrial | AAMP Certified

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Matheus%20Marques-blue)](https://linkedin.com/in/seu-perfil)
[![GitHub](https://img.shields.io/badge/GitHub-matheusmmarinho7-black)](https://github.com/matheusmmarinho7)
