## 📦 Lost & Found Gerado — Inventário T2 | Mercado Livre

Solução completa de Business Intelligence para monitoramento de inconsistências de estoque (Lost & Found) em tempo real, desenvolvida com BigQuery, Google Sheets e Looker Studio.

---

### 🎯 O Problema

O time de inventário do hub BRSP06 (Mercado Livre) gera diariamente milhares de registros de `found` e `lost` durante processos de reestoque, auditoria e conciliação de estoque, sem visibilidade centralizada dos dados. Não havia como responder perguntas como:

- Qual a proporção de Lost x Found no período?
- Quais zonas e processos concentram mais inconsistências?
- Quem são os colaboradores e líderes com maior volume de ocorrências?
- Quais casos exigem atenção prioritária (maior quantidade por ocorrência)?

---

### 💡 A Solução

Desenvolvi um pipeline completo de dados — do banco de dados ao dashboard visual — que atualiza automaticamente a cada 1 hora e permite acompanhar as inconsistências de estoque do time em tempo real.

---

### 🛠️ Tecnologias Utilizadas

| Ferramenta | Uso |
|---|---|
| **BigQuery** | Banco de dados, processamento da query SQL e agendamento automático via Scheduled Query |
| **Google Sheets** | Camada de extração e enriquecimento dos dados (colaborador, líder) |
| **Looker Studio** | Visualização e dashboard interativo |

---

### ⚙️ Como Funciona — Arquitetura do Pipeline

```
BigQuery (banco de dados)
  ↓ Scheduled Query (atualização automática a cada 1h)
Google Sheets (folha vinculada em tempo real)
  ↓ PROCV para enriquecer com colaborador, líder e classificação de zona
Looker Studio (dashboard interativo)
```

---

### 📊 Query SQL no BigQuery

**1. Extração dos eventos de Lost & Found (query agendada):**

```sql
SELECT
    DATETIME_ADD(FBM_CREATED_DATE, INTERVAL 30 MINUTE) AS DATA,
    EXTRACT(HOUR FROM DATETIME_ADD(FBM_CREATED_DATE, INTERVAL 30 MINUTE)) AS HORA,
    FBM_USER_ID AS ID_GROOT,
    FBM_REASON_PROCESS AS PROCESSO,
    INVENTORY_ID,
    ADDRESS_FROM,
    ADDRESS_TO,
    SUM(FBM_QUANTITY) AS PECAS
FROM meli-bi-data.whowner.bt_fbm_stock_movement
GROUP BY 1, 2, 3, 4, 5, 6, 7
```

**2. Enriquecimento com nome e time do colaborador:**

```sql
WITH team AS (
  SELECT * FROM UNNEST([
    STRUCT(1955498 AS FBM_USER_ID, 'MARIA DOS SANTOS SILVA' AS COLABORADOR),
    STRUCT(2252805 AS FBM_USER_ID, 'MARIA EDUARDA SILVA DE AGUIAR' AS COLABORADOR),
    STRUCT(2420267 AS FBM_USER_ID, 'THIAGO ROCHA SILVA' AS COLABORADOR),
    STRUCT(1772414 AS FBM_USER_ID, 'MIRELLA PELOGIA COELHO' AS COLABORADOR)
    -- ... demais colaboradores do time
  ])
)
SELECT *
FROM team
```

---

### 📈 Resultados do Dashboard (T2 — Jun/Jul 2026)

| Métrica | Valor |
|---|---|
| Total de Inconsistências | 66.941 |
| Found (Ocorrências) | 45.380 |
| Lost (Ocorrências) | 21.561 |
| Unidades Geradas | 168.018 |
| % Lost | 32,21% |

**Principais painéis do dashboard:**
- Found x Lost diário (série temporal)
- Zonas ofensoras (ranking por zona de armazenagem)
- Processo ofensor (found, cycle_count, stock_audit, audit_v2_conciliation, packing etc.)
- Casos de atenção (maiores quantidades por ocorrência)
- Top ofensores por colaborador
- Medidor de inconsistência (% Found x Lost)

Filtros disponíveis: período, tipo de inconsistência, zona, líder, colaborador, processo e faixa de quantidade.

---

### 🖼️ Preview

*(inserir print do dashboard `Lost & Found Gerado — Inventário T2`)*

---

### 👤 Autor

**Hiago Prates**
