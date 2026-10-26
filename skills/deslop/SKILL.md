---
name: deslop
description: Digest a slop-heavy Azure DevOps work item into a two-minute read for a human. Use for "/deslop", "deslopifica", "resume-me o AB#…", "não consigo ler este issue", an Epic or Feature to survey ("o que falta no Ofício?"), or pasted text and MR descriptions. Read-only.
---

# Deslop: a work item as a two-minute read

The reader is a developer who must grasp an item fast: what goes wrong today, what changes, how to see it, what is still open. Most items that need this come out of `/pms:UserStory`: thirteen fixed sections, grounding notes full of `file:line`, Given/When/Then criteria. The digest keeps every fact a tester or developer acts on and moves the scaffolding to the **cut list**. It is a **two-minute read**: one screen, and the first three lines alone say what the item is.

## Input

- `AB#<id>` of a Story, Bug, Issue or Task: the steps below.
- An Epic or Feature: **Survey**, at the end.
- Pasted text, or an MR (`gitlab_get_merge_request`): skip step 1 and fill the sections that apply.

Default org `primary`, project `Core-Host PMS`, through the devflow `ado_*` tools.

## Steps

1. **Read everything.** `ado_get_work_item` with `includeComments: true`. Read every HTML field with content: `System.Description`, `Microsoft.VSTS.TCM.ReproSteps`, `Microsoft.VSTS.Common.AcceptanceCriteria`, `Custom.DeveloperNotes`, and any other `Custom.*` text field. Open every `<img>` and every image `AttachedFile` with `ado_get_attachment`; on short items written by a person, the images often are the request. Get the titles and states of the linked items (parent, predecessors, related) with one `ado_query_work_items` call on their ids. Done when every field, comment, image and link is in hand.
2. **Sort** each piece with the map below into exactly one digest section or the cut list. A sentence you keep loses every pattern the `unslop` skill lists.
3. **Write** the digest in the template, in the language of the request. Quote the item verbatim only for error messages, screen labels, parameter values and a tester's rejection. Write it by the `unslop` skill, quotes included. A quote keeps the item's words and takes unslop's punctuation.
4. **Check** before sending:
   - every acceptance criterion is a line in **Testar**; merge duplicates, drop none;
   - every open question, MEDIUM or LOW grounding note, blocking dependency, and contradiction is in **Em aberto**. A contradiction is two places promising different behaviour. Differing metadata (Effort against SP), the template's fixed headings, and text that was true when written are not;
   - every sentence traces to the item, its images or its links' titles; the example uses the item's own numbers, or says "exemplo ilustrativo";
   - the part above **Testar**, header and labels included, stays under 150 words;
   - unslop's self-audit question finds nothing left to fix.

## Sort map

| Source (`/pms:UserStory` section or field) | Goes to |
|---|---|
| The requester's original text, above a "Bloco acrescentado" line | O pedido, as the authority; where the added block goes further, Em aberto |
| "Título sugerido" | the title line |
| Story, Contexto, Propósito, Âmbito, ReproSteps | O problema, O pedido |
| An AC's "DADO" case, or a Conceito flow | Exemplo |
| Método de invocação: Navegação, Elemento | Onde, as the menu path a tester clicks |
| Parametrização: parameter, country, licence, rights | Regras, only lines that gate behaviour ("Nenhum" lines are cut) |
| Fora de âmbito | Fora, one line, siblings by AB# or short title |
| Acceptance criteria | Testar, one line each, without the AC-xxx ids |
| Exploração criativa "Sim", MEDIUM/LOW grounding, "a confirmar", blocking dependency | Em aberto |
| Developer Notes | O pedido on a finished item; where they differ from the ACs, Em aberto |
| A review finding pasted into Developer Notes (severity mark, "Fix:") | Em aberto; Estado instead when a later commit title says it was fixed |
| Tester and other human comments | Estado, the latest rejection verbatim |
| DevOpsBot commit comments | Estado, only the merge, as "MR !n merged dd/mm" |
| Persona, Classificação, Story Points derivation, HIGH grounding notes, Localização `file:line`, Critérios de sucesso that repeat the ACs, Ref de grounding, the "Story criada com /pms:UserStory" line | cut list |

## Template

Omit a section that has nothing. Em aberto always appears; when empty it reads "Nada."

```
**AB#<id>** · <type> · <state> · <StoryPoints> SP · dev <System.AssignedTo> · tester <Custom.Tester> · <Custom.Version>
<the title in at most 10 words, when the original runs longer>

**O problema.** <1-2 sentences: what the user sees today, in screen terms>
**O pedido.** <1-3 sentences: what happens after the change>
**Exemplo.** <one case, start to finish, with the item's numbers and screen labels>
**Onde.** <menu path>

**Regras**
- <country, parameter id and value, right id, time window>

**Fora.** <one line>

**Testar**
1. <situation> → <result>

**Em aberto**
- <question, risk, dependency, contradiction>

**Estado.** <latest rejection verbatim, MR, what is missing>

_Lido: <fields read>, <n> comentários, <n> de bots, <n> imagens, <n> itens ligados. Cortado: <cut-list sections; count the grounding notes>._
```

## Survey (Epic or Feature)

1. Collect the children with WIQL `[System.Parent] = <id>`. For an Epic, repeat on each child Feature.
2. Group by state: **Por fazer** (New, Active), **Em teste** (To Test), **Feito** (Resolved, Closed).
3. One line per child: `AB#<id> · <assignee> · <what changes, at most 12 words>`. State a prefix all titles share once in the header and strip it from the lines. Take "what changes" from the title; open the item only when the title does not say it.
4. Under **Atenção**, name each pair of children where a later one replaces or contradicts an earlier one's behaviour.
5. End with the count per group.
