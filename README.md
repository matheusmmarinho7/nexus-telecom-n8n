# Nexus Telecom — Automação de Atendimento com n8n + IA

Projeto de automação desenvolvido como case study de triagem inteligente de e-mails e monitoramento de SLA para uma empresa de telecomunicações fictícia.

## Workflows

### 1. Pipeline de Triagem de E-mails
Automatiza a classificação e roteamento de e-mails recebidos.

**Fluxo:**
Gmail Trigger → Classificação por IA (GPT-4o-mini) → Switch → Slack (canal por categoria) → Google Sheets (log) → Auto-reply

**Categorias identificadas pela IA:** Suporte Técnico, Financeiro, Comercial, Outros

### 2. Monitor de SLA
Verifica periodicamente tickets em aberto e dispara alertas quando o prazo está próximo de vencer.

**Fluxo:**
Schedule Trigger → Google Sheets (leitura) → Code JS (cálculo de prazo com UTC-3) → IF → Slack (alerta)

## Stack
- n8n (orquestração)
- OpenAI GPT-4o-mini (classificação)
- Gmail, Google Sheets, Slack
- JavaScript (parsing de datas)

## Objetivo
Reduzir tempo de resposta e eliminar triagem manual de chamados, garantindo SLA dentro do prazo.
