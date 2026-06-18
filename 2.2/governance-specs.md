# Governança de Specs — NovaTech Assistant

> **Localização no repositório:** `docs/governance-specs.md`
> **Versão:** 1.0 · **Autor:** Tech Lead · **Data:** Maio 2026
> **Status:** Aprovado

---

## 1. Princípio central

Specs não são documentos passivos — são contratos executáveis.
Um `requirements.md` define o **o quê** e determina o critério de aceite.
Um `plan.md` define o **como** e responde por decisões arquiteturais.
Um `tasks.md` define **unidades atômicas** executáveis por agentes de IA.

Cada transição entre camadas é um checkpoint humano. Nenhuma camada avança sem aprovação explícita da anterior.

---

## 2. Módulos do projeto e seus responsáveis

| # | Módulo | Pasta no repositório | Requirements | Plan | Tasks |
|---|--------|---------------------|-------------|------|-------|
| M1 | Pipeline de ingestão | `specs/pipeline-ingestao/` | PS | TL | Dev Sr + Copilot |
| M2 | API de busca (query endpoint) | `specs/query-endpoint/` | PS | TL | Dev Sr + Copilot |
| M3 | API de feedback | `specs/feedback-api/` | PS | TL | Dev Pl + Copilot |
| M4 | Bot do Teams | `specs/teams-bot/` | PS | TL | Dev Pl + Copilot |
| M5 | Painel web | `specs/painel-web/` | PS | TL | Dev Pl + Copilot |

**Legenda:** PS = Product Specialist · TL = Tech Lead · Dev Sr = Desenvolvedor Sênior · Dev Pl = Desenvolvedor Pleno

---

## 3. Quem cria cada tipo de spec

### 3.1 `requirements.md` — Product Specialist

O PS é o autor e responsável pelos requirements. Usa Claude e Claude Design como apoio para redigir, estruturar e validar cobertura de casos de uso.

**O PS não define implementação.** Qualquer detalhe técnico inserido em requirements é removido na revisão do TL.

Ferramentas permitidas:
- Claude: rascunho inicial a partir de épicos do Azure DevOps
- Claude Design: wireframes referenciados no documento
- Azure DevOps: critérios de aceite vinculados a histórias

### 3.2 `plan.md` — Tech Lead

O TL é o autor e responsável pelo plan. Usa Copilot e Claude como apoio para explorar abordagens arquiteturais, validar consistência com ADRs e gerar rascunhos de interfaces TypeScript.

**O plan não redefine requisitos.** Se o TL identificar que um requisito é inviável, abre issue no requirements antes de escrever o plan.

Ferramentas permitidas:
- Claude: rascunho de plan a partir do requirements aprovado
- Copilot: sugestão de interfaces TypeScript, contratos de API, estrutura de módulos
- GitHub: referências a ADRs no repositório

### 3.3 `tasks.md` — Dev (Sênior ou Pleno) com apoio de Copilot

O Dev é o autor das tasks. Usa Claude para decompor histórias do plan em unidades atômicas e Copilot para sugerir estrutura técnica de cada task.

**O Dev não inventa escopo.** Tasks devem ser rastreáveis a parágrafos específicos do plan. Task sem referência ao plan é inválida.

Ferramentas permitidas:
- Claude: decomposição do plan.md em tasks com input/output/critério de done
- Copilot: scaffold técnico de cada task (tipos, dependências, estrutura de teste)
- Azure DevOps: criação das tasks vinculadas à história correspondente

---

## 4. Estrutura obrigatória de cada arquivo

### 4.1 Template de `requirements.md`

```markdown
# Requirements — [Nome do Módulo]

> **Módulo:** [slug]
> **Autor:** [nome] (Product Specialist)
> **Versão:** [X.Y]
> **Status:** [Rascunho | Em revisão | Aprovado]
> **Aprovado por:** [nome do TL] · [data]
> **Vinculado a:** [link do épico no Azure DevOps]

---

## Contexto

[Por que este módulo existe. Problema que resolve para o atendimento NovaTech.]

## Personas e casos de uso

[Para cada persona: quem é, o que precisa, como usa este módulo.]

## Requisitos funcionais

### RF-[MÓDULO]-001 — [Título]
**Descrição:** [O que o sistema deve fazer.]
**Critério de aceite:** [Condição verificável. Sem ambiguidade.]
**Prioridade:** [Must / Should / Could]

[Repetir para cada requisito.]

## Requisitos não funcionais

| ID | Requisito | Valor alvo | Critério de medição |
|----|-----------|-----------|---------------------|
| RNF-[M]-001 | Latência de resposta | < 3s p95 | Azure Monitor |
| RNF-[M]-002 | Disponibilidade | ≥ 99,5% | Azure Monitor |

## Fora de escopo

[O que explicitamente não será feito neste módulo. Evita expansão silenciosa.]

## Referências

[ADRs relevantes, wireframes, glossário de domínio.]
```

---

### 4.2 Template de `plan.md`

```markdown
# Plan — [Nome do Módulo]

> **Módulo:** [slug]
> **Autor:** [nome] (Tech Lead)
> **Versão:** [X.Y]
> **Status:** [Rascunho | Em revisão | Aprovado]
> **Aprovado por:** [nome PS + nome Dev Sr] · [data]
> **Baseado em:** requirements.md v[X.Y] (aprovado em [data])
> **ADRs referenciados:** [lista de ADRs]

---

## Visão arquitetural

[Diagrama ou descrição de como este módulo se encaixa na arquitetura geral.
Referência ao ADR relevante para cada decisão arquitetural não trivial.]

## Componentes

### [Nome do componente]
**Localização no repositório:** `src/[caminho]`
**Responsabilidade:** [Uma frase.]
**Interface de entrada:** [Tipos TypeScript ou contrato de API]
**Interface de saída:** [Tipos TypeScript ou contrato de API]
**Dependências internas:** [outros componentes deste projeto]
**Dependências externas:** [Azure AI Search, Azure OpenAI, etc.]

[Repetir para cada componente.]

## Fluxo de dados

[Sequência de chamadas entre componentes. Texto ou ASCII diagram.]

## Decisões técnicas

| Decisão | Alternativas consideradas | Justificativa | ADR |
|---------|--------------------------|---------------|-----|
| [ex: chunking por parágrafo] | [sentença, token fixo] | [motivo] | ADR-0004 |

## Estratégia de testes

[Quais tipos de teste este módulo exige (unit, integration, e2e, acurácia RAG).
Para módulos com RAG: definir dataset de avaliação e critério de acurácia mínima.]

## Estimativa de esforço

| Componente | Perfil | Estimativa | Observações |
|-----------|--------|-----------|-------------|
| [nome] | Dev Sr | 3 dias | Inclui testes de integração |

## Riscos técnicos

[Riscos identificados durante o planejamento. Referência ao risk register do projeto se aplicável.]
```

---

### 4.3 Template de `tasks.md`

```markdown
# Tasks — [Nome do Módulo]

> **Módulo:** [slug]
> **Autor:** [nome] (Dev Sênior / Pleno)
> **Versão:** [X.Y]
> **Status:** [Rascunho | Em revisão | Aprovado]
> **Aprovado por:** [nome TL] · [data]
> **Baseado em:** plan.md v[X.Y] (aprovado em [data])

---

## TASK-[MÓDULO]-001 — [Título da task]

**Tipo:** `implementation` | `test` | `infrastructure` | `documentation`
**Perfil:** Dev Sênior / Dev Pleno
**Estimativa:** [N horas]
**Dependências:** [TASK-XXX-YYY ou "nenhuma"]
**Referência no plan:** [seção do plan.md que origina esta task]
**Item no Azure DevOps:** [link]

**Input:**
[O que esta task recebe para começar. Tipos, contratos, dados de fixture.]

**Output:**
[O que esta task produz ao terminar. Arquivo, função, endpoint, etc.]

**Critério de done:**
- [ ] [Condição verificável 1]
- [ ] [Condição verificável 2]
- [ ] Testes passando com cobertura ≥ 80%
- [ ] Code review aprovado pelo TL ou Dev Sr
- [ ] Sem violação de lint/type-check

**Contexto para o agente:**
[Instrução específica para Copilot ou Claude ao implementar esta task.
Skills do projeto a usar: `skills/foundation/[nome].md`, `skills/domain/[nome].md`]

---

[Repetir para cada task.]

## Ordem de execução

```
TASK-[M]-001 → TASK-[M]-002 → TASK-[M]-003
                              ↘ TASK-[M]-004 (paralela)
```

## Checklist de fechamento do módulo

- [ ] Todas as tasks com status "done" no Azure DevOps
- [ ] Todos os RFs do requirements.md cobertos por pelo menos uma task
- [ ] Acurácia RAG ≥ 85% (se módulo envolver retrieval)
- [ ] Documentação de runbook criada em `docs/runbooks/[módulo].md`
```

---

## 5. Convenções de nomenclatura e versionamento

### 5.1 Localização no repositório

```
specs/
├── pipeline-ingestao/
│   ├── requirements.md
│   ├── plan.md
│   └── tasks.md
├── query-endpoint/
│   ├── requirements.md
│   ├── plan.md
│   └── tasks.md
[...demais módulos]
```

O nome da pasta é o **slug do módulo**: kebab-case, sem acentos, sem espaços.
Nenhum arquivo de spec vive fora de sua pasta de módulo.

### 5.2 Versionamento semântico de specs

Specs seguem versionamento `MAJOR.MINOR` no frontmatter do documento:

| Tipo de mudança | Ação | Exemplo |
|----------------|------|---------|
| Correção de redação, sem impacto em implementação | MINOR +1 | 1.0 → 1.1 |
| Adição de requisito, componente ou task | MINOR +1 | 1.2 → 1.3 |
| Remoção de requisito ou mudança de escopo | MAJOR +1 | 1.3 → 2.0 |
| Refatoração completa após reprovação | MAJOR +1 | 1.0 → 2.0 |

**Regra de compatibilidade:** `plan.md` deve referenciar a versão do `requirements.md` em que foi baseado. `tasks.md` deve referenciar a versão do `plan.md`. Se o requirements for atualizado para v2.0, o plan e as tasks precisam ser revistos e re-aprovados.

### 5.3 Status válidos

Toda spec tem um campo `Status` com um dos seguintes valores:

| Status | Significado |
|--------|-------------|
| `Rascunho` | Sendo escrito pelo autor. Não pode ser usado como base. |
| `Em revisão` | PR aberto, aguardando aprovação do revisor designado. |
| `Aprovado` | Aprovado pelo revisor. Pode ser usado como base para a próxima camada. |
| `Obsoleto` | Substituído por versão mais nova. Mantido para histórico. |

---

## 6. Fluxo de aprovação (checkpoints humanos)

```
[PS] Escreve requirements.md (Rascunho)
         ↓
     Abre PR no GitHub
         ↓
[TL] Code review + aprovação
     → Verifica: escopo claro, critérios verificáveis, sem detalhe técnico
     → SLA: 1 dia útil
         ↓
     requirements.md → Status: Aprovado
         ↓ ━━━━━━━━━━━━━━━━━━ CHECKPOINT 1 ━━━━━━━━━━━━━━━━━━
[TL] Escreve plan.md (Rascunho)
     Referencia: requirements.md vX.Y (Aprovado)
         ↓
     Abre PR no GitHub
         ↓
[PS + Dev Sr] Code review + aprovação (ambos obrigatórios)
     → PS verifica: plan cobre todos os RFs, sem scope creep
     → Dev Sr verifica: abordagem técnica é implementável, estimativas coerentes
     → SLA: 1 dia útil
         ↓
     plan.md → Status: Aprovado
         ↓ ━━━━━━━━━━━━━━━━━━ CHECKPOINT 2 ━━━━━━━━━━━━━━━━━━
[Dev] Escreve tasks.md (Rascunho)
     Usa Claude + Copilot para decompor plan.md
     Referencia: plan.md vX.Y (Aprovado)
         ↓
     Abre PR no GitHub
         ↓
[TL + QA] Code review + aprovação (ambos obrigatórios)
     → TL verifica: tasks são atômicas, rastreáveis ao plan, sem escopo oculto
     → QA verifica: cada task tem critério de done testável, plano de teste vinculado
     → SLA: 4 horas úteis
         ↓
     tasks.md → Status: Aprovado
         ↓ ━━━━━━━━━━━━━━━━━━ CHECKPOINT 3 ━━━━━━━━━━━━━━━━━━
[Dev] Implementação pode iniciar
```

**Regra de bloqueio:** Nenhuma implementação começa em módulo com spec no status `Rascunho` ou `Em revisão`. O Azure DevOps deve refletir isso: tasks do Azure DevOps são movidas para `Em andamento` somente após o `tasks.md` do módulo estar `Aprovado`.

---

## 7. Como mudanças são rastreadas

### 7.1 Toda mudança em spec passa por PR

Nenhuma alteração direta em branch principal. O PR deve conter:

- Descrição da mudança: o que mudou e por quê
- Impacto nas camadas dependentes: se `requirements.md` muda, indicar se `plan.md` e `tasks.md` precisam de revisão
- Referência ao item do Azure DevOps que motivou a mudança (bug, feedback de sprint, change request)

### 7.2 Changelog interno de cada spec

Todo arquivo de spec mantém uma seção `## Histórico de versões` ao final:

```markdown
## Histórico de versões

| Versão | Data | Autor | Tipo | Descrição |
|--------|------|-------|------|-----------|
| 1.0 | 2026-05-10 | Ana Lima (PS) | Major | Versão inicial aprovada |
| 1.1 | 2026-05-18 | Ana Lima (PS) | Minor | Adicionado RF-QE-005 (histórico de sessão) |
| 2.0 | 2026-05-25 | Ana Lima (PS) | Major | Reformulação do escopo de feedback pós-sprint 1 |
```

### 7.3 Rastreabilidade bidirecional

Cada spec referencia para cima (o que a originou) e para baixo (o que ela origina):

| Artefato | Referencia | É referenciado por |
|----------|-----------|-------------------|
| `requirements.md` | Épico no Azure DevOps | `plan.md` |
| `plan.md` | `requirements.md` vX.Y · ADRs | `tasks.md` |
| `tasks.md` | `plan.md` vX.Y · Histórias no Azure DevOps | Código em `src/` (via comentários) |

**Convenção no código:** todo arquivo de implementação que resolve uma task deve ter no topo um comentário de referência:

```typescript
/**
 * @spec specs/query-endpoint/tasks.md#TASK-QE-003
 * @plan specs/query-endpoint/plan.md#componentes-search-service
 */
```

---

## 8. Regras para uso de IA na criação de specs

### O que a IA pode fazer

| Atividade | Ferramenta | Regra |
|-----------|-----------|-------|
| Rascunho de requirements a partir de épicos | Claude | Sempre revisado pelo PS antes do PR |
| Rascunho de plan a partir de requirements | Claude | Sempre revisado pelo TL antes do PR |
| Decomposição de tasks a partir do plan | Claude + Copilot | Sempre revisado pelo Dev antes do PR |
| Sugestão de critérios de aceite | Claude | Aprovação do PS (requirements) ou TL (tasks) |
| Geração de interfaces TypeScript no plan | Copilot | Revisão obrigatória do TL |

### O que a IA não pode fazer

- Aprovar uma spec (aprovação é sempre humana)
- Marcar status como `Aprovado` no documento
- Criar PRs de spec sem revisão humana do conteúdo antes de abrir
- Alterar o escopo de um módulo sem que isso seja explicitamente solicitado por um humano

### Regra de transparência

Toda spec que teve participação relevante de IA na redação deve declarar isso no frontmatter:

```markdown
> **Gerado com apoio de IA:** Sim — rascunho inicial gerado por Claude, revisado e aprovado por [nome]
```

---

## 9. Checklist de fechamento de spec por módulo

Antes de marcar um módulo como pronto para desenvolvimento, o TL verifica:

### Checklist de requirements

- [ ] Todos os casos de uso do discovery estão cobertos por pelo menos um RF
- [ ] Cada RF tem critério de aceite verificável (sem "deve funcionar bem")
- [ ] Fora de escopo está explicitamente declarado
- [ ] Aprovado pelo TL com PR mergeado em main
- [ ] Versão registrada no histórico do documento

### Checklist de plan

- [ ] Cada RF do requirements tem pelo menos um componente correspondente no plan
- [ ] Todas as decisões técnicas não triviais têm ADR referenciado
- [ ] Interfaces TypeScript de entrada e saída definidas para cada componente
- [ ] Estratégia de testes definida (incluindo critério de acurácia RAG se aplicável)
- [ ] Aprovado por PS e Dev Sr com PR mergeado em main

### Checklist de tasks

- [ ] Cada task é atômica: pode ser implementada em uma sessão de trabalho (≤ 1 dia)
- [ ] Cada task referencia o parágrafo do plan que a origina
- [ ] Cada task tem critério de done com checklist verificável
- [ ] Cada task está vinculada a um item no Azure DevOps
- [ ] Ordem de execução e dependências entre tasks estão explícitas
- [ ] Aprovado por TL e QA com PR mergeado em main

---

## 10. Referências

- `docs/adr/template.md` — template para ADRs
- `docs/onboarding.md` — guia de onboarding do time
- `docs/governance-specs.md` — este documento
- `skills/foundation/` — skills de convenções do projeto
- Azure DevOps: board do projeto NovaTech Assistant
- Repositório: `db1/novatech-assistant`
