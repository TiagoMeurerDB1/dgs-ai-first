# Gestão de Mudança (GMUD) — NovaTech Assistant

> **Localização:** `docs/gmud.md`
> **Versão:** 1.0 · **Autor:** Delivery Manager · **Data:** Maio 2026
> **Status:** Aprovado

---

## 1. Princípio central

Toda mudança que afeta produção, specs aprovadas, prompts versionados ou a base documental do RAG precisa ser **registrada, classificada e aprovada** antes de ser executada. O modelo é **híbrido**: mudanças rotineiras seguem um fluxo leve (PR + checklist); mudanças críticas seguem um fluxo formal com aprovadores em cadeia e janela de execução controlada.

A GMUD não existe para criar burocracia — existe para garantir que nenhuma mudança silenciosa degrade a acurácia do assistente ou quebre o contrato com a NovaTech.

---

## 2. Escopo de cobertura

A GMUD cobre **todos os artefatos que afetam o comportamento do assistente em produção**:

| Escopo | Exemplos |
|--------|---------|
| **Código e infraestrutura** | Azure Functions, pipeline RAG, bot Teams, painel web, Bicep |
| **Specs SDD** | Qualquer alteração em `requirements.md`, `plan.md` ou `tasks.md` aprovados |
| **Prompts do assistente** | `prompts/system-prompt.md`, estratégia de chunking, context budget |
| **Base documental do RAG** | Adição, remoção ou atualização de documentos na base indexada (SharePoint, Confluence, planilhas) |

---

## 3. Classificação de mudanças

Toda mudança é classificada em uma de três categorias antes de qualquer ação.

### Tipo 1 — Rotineira (fluxo leve)

Mudanças de baixo risco com impacto previsível e reversão simples.

| Característica | Detalhe |
|---------------|---------|
| Aprovadores | 1 (TL ou Dev Sr) |
| Janela de execução | Qualquer horário comercial |
| Rollback | Imediato via revert de commit ou reindexação |
| Registro | PR no GitHub com label `gmud:rotineira` |
| SLA de aprovação | 4 horas úteis |

**Exemplos:**
- Correção de bug sem alteração de interface pública
- Atualização de dependência menor (patch/minor sem breaking change)
- Adição de documento novo à base RAG (sem remover existentes)
- Ajuste de wording em spec não aprovada
- Atualização de runbook ou documentação interna

---

### Tipo 2 — Significativa (fluxo padrão)

Mudanças de médio risco que alteram comportamento observável ou contratos entre componentes.

| Característica | Detalhe |
|---------------|---------|
| Aprovadores | TL + PS (ambos obrigatórios) |
| Janela de execução | Horário comercial, notificação 24h antes |
| Rollback | Plano de rollback documentado no formulário |
| Registro | PR + formulário GMUD-T2 preenchido |
| SLA de aprovação | 1 dia útil |

**Exemplos:**
- Nova feature ou endpoint
- Alteração em spec aprovada (requirements ou plan)
- Mudança no system prompt do assistente
- Atualização major de dependência
- Remoção ou substituição de documento da base RAG
- Alteração na estratégia de chunking ou overlap
- Mudança em parâmetros do Azure AI Search (número de resultados, scoring)
- Adição de novo ADR que revisa decisão anterior

---

### Tipo 3 — Crítica (fluxo formal)

Mudanças de alto risco com potencial de degradar a acurácia do assistente, quebrar integrações ou afetar dados em produção.

| Característica | Detalhe |
|---------------|---------|
| Aprovadores | TL + DM + NovaTech (representante designado) |
| Janela de execução | Fora do horário de pico do atendimento (definido com NovaTech) |
| Rollback | Plano testado em staging antes da execução |
| Registro | Formulário GMUD-T3 + registro no Azure DevOps + ata de aprovação |
| SLA de aprovação | 2 dias úteis |
| Notificação | NovaTech notificada 48h antes da execução |

**Exemplos:**
- Troca de modelo LLM (ex: GPT-4o → GPT-4o-mini ou versão nova)
- Mudança no schema do índice Azure AI Search (reindexação completa)
- Alteração do context budget (tamanho dos chunks, número de resultados)
- Mudança na política de documentos contraditórios (ADR-0003)
- Deploy de novo componente em produção pela primeira vez
- Remoção em lote de documentos da base RAG (> 10% do total)
- Alteração nas permissões de acesso ao assistente

---

## 4. Fluxo por tipo de mudança

### Tipo 1 — Rotineira

```
[Autor] Identifica necessidade de mudança
    ↓
[Autor] Abre PR com:
    - Descrição da mudança
    - Classificação: Tipo 1 (Rotineira)
    - Label: gmud:rotineira
    - Checklist T1 preenchido
    ↓
[TL ou Dev Sr] Code review → aprovação
    SLA: 4h úteis
    ↓
[Autor] Merge + execução
    ↓
[Autor] Fecha item no Azure DevOps com link do PR
```

**Checklist T1 (no PR):**
```markdown
## GMUD — Tipo 1: Rotineira

- [ ] A mudança não altera interfaces públicas entre componentes
- [ ] A mudança não afeta prompts, chunking ou parâmetros do RAG
- [ ] Testes existentes continuam passando (CI verde)
- [ ] Rollback possível via revert de commit em < 5 min
- [ ] Sem impacto na acurácia do assistente (confirmado por análise do autor)
```

---

### Tipo 2 — Significativa

```
[Autor] Identifica necessidade de mudança
    ↓
[Autor] Preenche Formulário GMUD-T2 (ver seção 5)
    ↓
[DM] Classifica como Tipo 2 e registra no Azure DevOps
    ↓
[Autor] Abre PR com formulário GMUD-T2 anexado
    Label: gmud:significativa
    ↓
[TL] Revisão técnica → aprovação
    SLA: 1 dia útil
    ↓
[PS] Revisão de impacto em produto → aprovação
    SLA: 1 dia útil
    ↓
[Autor] Merge + execução em janela comercial
    ↓
[QA] Smoke tests pós-execução (20 perguntas críticas)
    ↓
[DM] Fecha registro no Azure DevOps com resultado
```

---

### Tipo 3 — Crítica

```
[Autor] Identifica necessidade de mudança
    ↓
[Autor] Preenche Formulário GMUD-T3 (ver seção 5)
    ↓
[DM] Classifica como Tipo 3
    Notifica: TL + PS + NovaTech (representante)
    Registra no Azure DevOps com tag: gmud-critica
    ↓
[TL] Revisão técnica aprofundada
    Valida rollback plan em staging
    SLA: 2 dias úteis
    ↓
[DM + NovaTech] Aprovação formal
    Registro de aprovação em ata (e-mail ou Teams)
    Define janela de execução
    ↓
[TL] Executa em staging → valida acurácia RAG
    Critério: acurácia ≥ 85% no dataset de 200 perguntas
    ↓
[DM] Notificação 48h antes à NovaTech
    ↓
[TL + Dev Sr] Execução em produção na janela aprovada
    ↓
[QA] Smoke tests pós-execução
    Monitoramento ativo por 24h (Azure Monitor)
    ↓
[DM] Fecha registro com resultado, evidências e métricas de acurácia
    Notifica NovaTech com relatório de execução
```

**Critério de rollback automático (Tipo 3):**
Se acurácia cair abaixo de 80% nas primeiras 24h após execução, rollback é acionado sem necessidade de nova aprovação. TL executa e DM notifica NovaTech em até 1h.

---

## 5. Formulários GMUD

### Formulário T2 — Mudança Significativa

```markdown
## GMUD-T2 — Mudança Significativa

**ID:** GMUD-[ANO]-[NNN]
**Data de abertura:** 
**Solicitante:** 
**Módulo afetado:** 

---

### Descrição da mudança
[O que será alterado e por quê.]

### Classificação de escopo
- [ ] Código / infraestrutura
- [ ] Spec SDD (requirements / plan / tasks)
- [ ] Prompt do assistente
- [ ] Base documental do RAG

### Impacto esperado
**Comportamento antes:** 
**Comportamento depois:** 
**Usuários afetados:** 

### Risco de regressão
[Quais funcionalidades existentes podem ser afetadas.]

### Plano de rollback
[Como reverter em caso de falha. Tempo estimado de rollback.]

### Plano de teste pós-execução
[O que será validado após a mudança. Quem executa. SLA de validação.]

### Janela de execução proposta
**Data/hora:** 
**Duração estimada:** 

### Aprovações
- [ ] TL: _______________________ Data: _______
- [ ] PS: _______________________ Data: _______
```

---

### Formulário T3 — Mudança Crítica

```markdown
## GMUD-T3 — Mudança Crítica

**ID:** GMUD-[ANO]-[NNN]
**Data de abertura:** 
**Solicitante:** 
**Módulo afetado:** 
**Impacto em produção:** Alto

---

### Descrição detalhada
[O que será alterado, por quê, e qual o gatilho desta mudança.]

### Classificação de escopo
- [ ] Modelo LLM
- [ ] Schema do índice (reindexação)
- [ ] Context budget / chunking
- [ ] Política de documentos contraditórios
- [ ] Permissões de acesso
- [ ] Base documental em lote (> 10%)
- [ ] Outro: _______________

### Análise de impacto

**Impacto na acurácia do assistente:**
[Estimativa qualitativa + evidência do protótipo ou staging.]

**Impacto nas integrações:**
[Teams bot, painel web, Azure DevOps, outros.]

**Impacto para o usuário final (atendente NovaTech):**
[O que muda na experiência de uso.]

**Dependências:**
[Outras mudanças que precisam ocorrer antes ou depois desta.]

### Evidência de validação em staging
**Acurácia RAG em staging (pós-mudança):** ___%  (mínimo: 85%)
**Dataset usado:** prompts/eval/golden-queries.json
**Executado por:** 
**Data:** 

### Plano de rollback testado
[Passos detalhados. Tempo de execução do rollback em staging: ___ min]

### Plano de monitoramento pós-execução
- Período de monitoramento ativo: 24h
- Critério de rollback automático: acurácia < 80%
- Responsável pelo monitoramento: TL / Dev Sr
- Ferramenta: Azure Monitor (alert configurado em: ___)

### Janela de execução aprovada
**Data/hora:** 
**Duração estimada:** 
**Justificativa da janela:** (fora do pico de atendimento NovaTech)

### Comunicação
- [ ] NovaTech notificada 48h antes (data: _______)
- [ ] Time DB1 notificado (data: _______)

### Aprovações formais
- [ ] TL: _______________________ Data: _______
- [ ] DM: _______________________ Data: _______
- [ ] NovaTech (representante): _______________________ Data: _______
```

---

## 6. Regras específicas por escopo

### 6.1 Mudanças em prompts (`prompts/`)

Todo commit em `prompts/system-prompt.md` é classificado como **no mínimo Tipo 2**.

Adicionalmente:
- O commit deve registrar a versão anterior e nova em `prompts/prompt-changelog.md`
- A nova versão deve ser avaliada contra `prompts/eval/golden-queries.json` antes de ir a produção
- O resultado da avaliação deve constar no formulário GMUD-T2 ou T3
- Se a mudança de prompt reduzir a acurácia em mais de 3 pontos percentuais → promovida automaticamente para Tipo 3

**Formato de entrada no `prompt-changelog.md`:**
```markdown
## v[X.Y] — [Data]
**Autor:** [nome]
**GMUD:** GMUD-[ANO]-[NNN]
**Motivo:** [Por que o prompt mudou]
**Resultado esperado:** [O que deve melhorar]
**Acurácia antes:** [X%] | **Acurácia depois:** [Y%]
```

---

### 6.2 Mudanças na base documental do RAG

A base documental tem três operações, cada uma com classificação padrão:

| Operação | Classificação padrão | Condição de upgrade |
|----------|---------------------|---------------------|
| Adicionar documentos novos | Tipo 1 | Tipo 2 se > 50 docs de uma vez |
| Atualizar documentos existentes | Tipo 2 | Tipo 3 se afeta > 10% da base |
| Remover documentos | Tipo 2 | Tipo 3 se > 10% da base |

**Processo de reindexação:**
1. Documentos novos/atualizados passam pelo pipeline de qualidade (score ≥ 4 nas 4 dimensões)
2. Execução em staging primeiro — validar que acurácia não regride
3. Indexação em produção fora do horário de pico
4. Log de reindexação registrado em `docs/runbooks/reindexacao.md`
5. Para remoções: documentos são marcados como `obsoleto` antes de serem removidos — nunca remoção direta

---

### 6.3 Mudanças em specs aprovadas

Uma spec com status `Aprovado` só pode ser alterada via GMUD:

| Camada da spec | Classificação | Aprovadores adicionais |
|---------------|---------------|----------------------|
| `tasks.md` | Tipo 1 ou 2 | TL + QA |
| `plan.md` | Tipo 2 | TL + PS + Dev Sr |
| `requirements.md` | Tipo 2 ou 3 | TL + PS + NovaTech (se impacta escopo contratado) |

Toda alteração em spec aprovada deve atualizar o campo `Versão` e adicionar entrada no `Histórico de versões` do documento (ver `docs/governance-specs.md`).

---

### 6.4 Mudanças em infraestrutura (`infra/`)

Mudanças em Bicep seguem a classificação padrão com uma regra adicional:

- **Toda mudança em Bicep é executada primeiro em `dev`, depois em `staging`, depois em `prod`** — sem pulos de ambiente
- O PR de infra deve incluir o output do `az deployment what-if` mostrando o que será criado/alterado/destruído
- Mudanças que destroem recursos (ex: recriação de índice AI Search) são automaticamente **Tipo 3**

---

## 7. Registro e rastreabilidade

### 7.1 Identificação de GMUDs

Formato do ID: `GMUD-[ANO]-[NNN]`
Exemplos: `GMUD-2026-001`, `GMUD-2026-012`

O ID é gerado sequencialmente pelo DM no momento do registro no Azure DevOps.

### 7.2 Labels no GitHub

Toda PR de mudança deve ter um dos seguintes labels:

| Label | Tipo |
|-------|------|
| `gmud:rotineira` | Tipo 1 |
| `gmud:significativa` | Tipo 2 |
| `gmud:critica` | Tipo 3 |
| `gmud:emergencia` | Emergência (ver seção 8) |

### 7.3 Registro no Azure DevOps

Toda GMUD Tipo 2 e Tipo 3 tem um item no Azure DevOps com:
- Tag: `gmud`
- Link para o PR do GitHub
- Status: `Aguardando aprovação` → `Aprovado` → `Executado` → `Fechado`
- Campos: ID GMUD, tipo, responsável, data de execução, resultado

### 7.4 Histórico consolidado

O DM mantém uma planilha ou wiki com o histórico de todas as GMUDs executadas:

| ID | Tipo | Escopo | Data | Responsável | Resultado | Acurácia antes/depois |
|----|------|--------|------|-------------|-----------|----------------------|
| GMUD-2026-001 | T2 | Prompt | 2026-05-15 | Ana Lima | Sucesso | 87% → 89% |

---

## 8. Mudança de emergência

Quando um incidente em produção exige correção imediata sem tempo para o fluxo completo:

```
[Qualquer membro] Identifica incidente crítico em produção
    ↓
[TL] Declara emergência → notifica DM imediatamente
    ↓
[TL] Executa correção (hotfix) com aprovação verbal do DM
    (aprovação formal retroativa em até 2h)
    ↓
[QA] Smoke tests imediatos (< 30 min)
    ↓
[TL] Abre PR retroativo com label gmud:emergencia
[DM] Preenche formulário GMUD-T3 retroativamente em até 4h
    ↓
[DM] Notifica NovaTech sobre incidente e correção (em até 1h após resolução)
    ↓
[Time] Post-mortem em até 2 dias úteis
    Registrado em: docs/runbooks/postmortem-[GMUD-ID].md
```

**Critério para declarar emergência:**
- Assistente fora do ar ou retornando erro em > 20% das requisições
- Acurácia caiu abaixo de 70% (queda abrupta, não gradual)
- Vazamento de dados ou acesso não autorizado a documentos restritos

---

## 9. Matriz de responsabilidades (RACI)

| Atividade | TL | Dev Sr | Dev Pl | QA | PS | DM | NovaTech |
|-----------|----|----|----|----|----|----|---------|
| Identificar e classificar mudança | R | C | C | C | C | A | I |
| Preencher formulário GMUD | R | C | C | - | C | A | - |
| Aprovar Tipo 1 | A | R | - | - | - | I | - |
| Aprovar Tipo 2 (técnico) | A | - | - | - | - | I | - |
| Aprovar Tipo 2 (produto) | - | - | - | - | A | I | - |
| Aprovar Tipo 3 | A | - | - | - | - | A | A |
| Executar mudança | R | C | C | - | - | I | - |
| Smoke tests pós-execução | - | - | - | R | - | I | I |
| Notificar NovaTech | - | - | - | - | - | R | A |
| Fechar registro | - | - | - | - | - | R | - |
| Post-mortem (emergência) | R | C | C | C | C | A | I |

**Legenda:** R = Responsável · A = Aprovador · C = Consultado · I = Informado

---

## 10. Referências

- `docs/governance-specs.md` — governança de specs SDD
- `docs/adr/template.md` — template de ADRs
- `prompts/prompt-changelog.md` — histórico de mudanças de prompt
- `docs/runbooks/` — runbooks operacionais por componente
- Azure DevOps: board do projeto (tag `gmud`)
- Repositório: `db1/novatech-assistant`
