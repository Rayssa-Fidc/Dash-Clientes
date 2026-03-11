# Dashboard Gerentes FIDC — LFS Financial Advisory

Dashboard operacional de controle de clientes FIDC, com visualização por gerente, funil de conversão, eficiência e gestão de documentos.

---

## Arquivos

| Arquivo | Descrição |
|---|---|
| `dashboard_gerentes_v3.html` | Dashboard HTML completo (abre direto no navegador) |
| `office_script_gerentes.ts` | Office Script para Excel Online — exporta os dados da planilha como JSON |

---

## Como funciona

### Planilha Excel
O arquivo `Controle de Operação - FIDC1.xlsx` (no OneDrive/SharePoint da empresa) possui duas abas:

- **Controle Aproximação** — dados de clientes, responsáveis, status, datas de entrada, cobranças e assinaturas
- **Docs Pendentes - FIDC** — checklist de documentos por cliente (checkboxes TRUE/FALSE)

### Office Script (`office_script_gerentes.ts`)

Lê as duas abas, processa os dados (incluindo agrupamento automático de empresas do mesmo grupo) e retorna um JSON estruturado.

**Agrupamento automático:** clientes com formato `"Razão Social (Nome do Grupo)"` são consolidados em uma única entrada usando o nome entre parênteses como identificador do grupo.

---

## Modos de uso

### Modo 1 — Atualização manual (mais simples)

1. Abra o arquivo no **Excel Online**
2. Vá em **Automatizar → Novo Script**
3. Cole o conteúdo de `office_script_gerentes.ts` e salve como `ExportarDashboard`
4. Clique em **Executar**
5. Copie o JSON retornado
6. Abra o `dashboard_gerentes_v3.html` no navegador
7. Clique em **"📋 Colar JSON manualmente"** no banner laranja e cole o JSON

### Modo 2 — Atualização automática via Power Automate (recomendado)

1. Crie um fluxo no **Power Automate**
2. Gatilho: agendado (ex: a cada hora, ou ao modificar o arquivo)
3. Ação: **Excel Online → Executar Script** → selecione `ExportarDashboard`
4. Ação: **HTTP** ou **SharePoint** → publique o JSON retornado em uma URL acessível
5. Cole essa URL na constante `API_URL` no `dashboard_gerentes_v3.html`

```js
// dashboard_gerentes_v3.html — linha de configuração
const API_URL = 'https://sua-url-publica.com/dashboard_data.json';
```

> **Opções de hospedagem do JSON:** SharePoint (arquivo público), Azure Blob Storage, GitHub Gist (raw URL), ou qualquer serviço com CORS habilitado.

---

## Estrutura do JSON exportado

```json
{
  "success": true,
  "updated_at": "2026-03-11T14:30:00.000Z",
  "rows": [
    {
      "cliente": "Nome do Cliente",
      "responsavel": "Gerente",
      "tipo": "Antecipação de Convencionais",
      "status": "Contrato Pendente",
      "entrada": "2026-02-10",
      "cob1": "2026-02-19",
      "cob2": "2026-02-23",
      "cob3": "",
      "status_cobranca": "Em tratativa",
      "assinado": "",
      "declinio_motivo": "",
      "observacao": ""
    }
  ],
  "docs_map": {
    "Nome do Cliente": {
      "pendentes": ["Balanço DRE 2025", "CNH Sócios"],
      "recebidos": ["Cartão CNPJ", "Contrato Social"]
    }
  }
}
```

---

## Estrutura da planilha esperada

### Aba "Controle Aproximação"

| Linha | Conteúdo |
|---|---|
| 1 | Título |
| 2 | Grupos de colunas |
| 3 | Sub-colunas / cabeçalhos |
| 4+ | Dados |

Mapeamento de colunas (0-indexed):

| Coluna | Campo |
|---|---|
| A (0) | Cliente |
| B (1) | Responsável |
| C (2) | Tipo |
| D (3) | Status |
| E (4) | Data de Entrada |
| F (5) | Cobrança 1 |
| G (6) | Cobrança 2 |
| H (7) | Cobrança 3 |
| I (8) | Status Cobrança |
| K (10) | Data Assinado |
| S (18) | Motivo Declínio |
| U (20) | Observação |

### Aba "Docs Pendentes - FIDC"

| Coluna | Campo |
|---|---|
| B (1) | Nome do cliente |
| F:Y (5-24) | Checkboxes dos documentos (TRUE = recebido) |

Os nomes dos documentos são lidos da **linha 3** (índice 2).

---

## Status possíveis

| Status | Alerta exibido |
|---|---|
| Finalizado | 🟢 ATIVO |
| Contrato Pendente / Documento Pendente / Contrato e Documentos | 🟠 EM ANDAMENTO |
| Declinado LFS / Declinado Cliente | 🔴 INATIVO |
| Pausado | 🔴 INATIVO (ou 🟡 SEM RETORNO se >30 dias) |
| Qualquer ativo >30 dias sem assinar | 🟡 SEM RETORNO |

---

## Desenvolvimento

Desenvolvido por Rayssa Nascimento — LFS Financial Advisory
