# Velocidade da Lux 2.0 — Dashboard de Vendas

Dashboard fullscreen para telão (4 TVs em grid 2×2) da campanha de vendas Lux.

---

## Arquivos

| Arquivo | Descrição |
|---|---|
| `index.html` | Telão fullscreen — abrir no Chrome em modo quiosque |
| `admin.html` | Painel de gestão — senha padrão: `lux2025` |

---

## Como usar

### 1. Configurar o telão

Abra o `index.html` no Chrome em modo fullscreen:

```
chrome.exe --kiosk "A:\Marketing\Apresentações\2026\Velocidade da Lux 2.0\index.html"
```

Ou simplesmente abra o arquivo e pressione **F11** para fullscreen.

O dashboard atualiza automaticamente a cada **60 segundos**.

### 2. Acessar o admin

Abra `admin.html` em qualquer navegador. Senha padrão: **`lux2025`**

Para alterar a senha, edite a linha no `admin.html`:
```js
const SENHA_ADMIN = 'lux2025';
```

### 3. Fluxo de operação

1. **Equipe** → Cadastre todos os EDNs e Executivos
2. **Histórico 2025** → Preencha os totais mensais para o gráfico comparativo
3. **Metas do Mês** → Configure a meta de reuniões (EDNs) e fechamentos (Executivos)
4. **Lançar Eventos** → Registre prospecções, reuniões e fechamentos ao longo do mês
5. **Início de cada mês** → Acesse "Destaques do Mês Ant." e defina os 3 vencedores

---

## Quadrantes do telão

| Quadrante | Conteúdo |
|---|---|
| **Q1** (sup. esq.) | Ranking ao vivo: EDNs por Reuniões Realizadas + Executivos por Receita |
| **Q2** (sup. dir.) | Gráfico comparativo 2025 vs 2026 + últimos 3 fechamentos do mês |
| **Q3** (inf. esq.) | Destaques do mês anterior (definidos manualmente no admin) |
| **Q4** (inf. dir.) | Progresso individual vs meta do mês |

---

## Sistema de pontuação

| Evento | Pontos |
|---|---|
| Prospecção | 1 pt |
| Reunião Agendada | 3 pts |
| Reunião Realizada | 5 pts |
| Fechamento | 20 pts |
| Bônus de Receita | Configurável no admin (faixas de valor) |

---

## Backup e restauração

### Exportar
No admin → seção **Backup** → botão **Exportar JSON**.
Salva um arquivo `lux_backup_YYYY-MM-DD.json`.

### Importar
No admin → seção **Backup** → selecione o arquivo JSON → **Importar e Sobrescrever**.
> ⚠️ A importação sobrescreve todos os dados. Sempre exporte antes de importar.

### Onde os dados ficam
Todos os dados são armazenados no **localStorage** do navegador.
Para manter os dados entre máquinas, use a função de Backup/Exportar e Importar.

---

## Integração Bitrix24 (Fase 2)

O código foi arquitetado com uma camada `DataService` que isola o acesso a dados.
Na Fase 2, basta substituir os métodos GET do `DataService` no `index.html`
**sem alterar nenhum Renderer ou lógica de UI**.

### Mapeamento de métodos → endpoints Bitrix24

```js
// No arquivo index.html, localize o objeto DataService e substitua:

getVendedores() {
  // Fase 2: GET /rest/crm.contact.list
  // Filtrar por campo customizado "perfil_campanha" = "EDN" ou "Executivo"
  // Mapear para: { id, nome, sobrenome, perfil, foto }
},

getEventos() {
  // Fase 2: GET /rest/crm.activity.list
  // Filtrar por tipo de atividade customizado da campanha "Velocidade Lux 2.0"
  // Mapear para: { id, vendedorId, data, tipo, ...dadosFechamento }
},

getFechamentos() {
  // Fase 2: GET /rest/crm.deal.list
  // Filtrar por stage_id = ID do estágio "Fechado"
  // Mapear para o mesmo schema de fechamento
},
```

### Exemplo de chamada Bitrix24 com fetch

```js
async getVendedores() {
  const resp = await fetch('https://suaempresa.bitrix24.com.br/rest/1/SEU_TOKEN/crm.contact.list.json?'
    + new URLSearchParams({
        filter: JSON.stringify({ 'UF_CRM_CAMPANHA': 'velocidade_lux_2' }),
        select: JSON.stringify(['ID','NAME','LAST_NAME','UF_CRM_PERFIL','PHOTO']),
      }));
  const { result } = await resp.json();
  return result.map(c => ({
    id:        c.ID,
    nome:      c.NAME,
    sobrenome: c.LAST_NAME,
    perfil:    c.UF_CRM_PERFIL, // campo customizado: 'EDN' ou 'Executivo'
    foto:      c.PHOTO || null,
  }));
},
```

### Ativar polling automático

Com a integração Bitrix24 ativa, o auto-refresh de 60 segundos já buscará
os dados atualizados da API automaticamente — nenhuma outra alteração necessária.

---

## Personalização rápida

| O que mudar | Onde |
|---|---|
| Senha do admin | `admin.html` → linha `const SENHA_ADMIN` |
| Intervalo de refresh | `index.html` → linha `INTERVALO_MS: 60000` |
| Cores / paleta | `index.html` → bloco `:root { --azul-1: ... }` |
| Máx. de pessoas no ranking | `index.html` → `.slice(0, 6)` nos Renderers |
| Meta padrão se não configurada | `index.html` → `DataService.getMetas()` → objeto `padrao` |
