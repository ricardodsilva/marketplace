---
name: deslop
description: Digest a slop-heavy Azure DevOps work item into a two-minute read for a human. Use for "/deslop", "deslopifica", "resume-me o AB#…", "não consigo ler este issue", an Epic or Feature to survey as in "o que falta no Ofício?", or pasted text and MR descriptions. Read-only.
---

# A work item as a two-minute read

The reader is a developer who must learn fast what goes wrong today, what changes, how to see it and what is still open. Most items that need this come out of `/pms:UserStory`, with thirteen fixed sections, grounding notes full of `file:line` and Given/When/Then criteria. The digest keeps every fact a tester or developer acts on and moves the scaffolding to the **cut list**. It is a **two-minute read**. It fits one screen, and its first three lines alone say what the item is.

## Input

- An `AB#<id>` of a Story, Bug, Issue or Task runs the steps below.
- An Epic or Feature runs the **Survey** at the end.
- Pasted text, or an MR read with `gitlab_get_merge_request`, skips step 1 and fills the sections that apply.

Default org `primary`, project `Core-Host PMS`, through the devflow `ado_*` tools.

## Steps

1. **Read everything.** `ado_get_work_item` with `includeComments: true`. Read every HTML field with content: `System.Description`, `Microsoft.VSTS.TCM.ReproSteps`, `Microsoft.VSTS.Common.AcceptanceCriteria`, `Custom.DeveloperNotes`, and any other `Custom.*` text field. Open every `<img>` and every image `AttachedFile` with `ado_get_attachment`; on short items written by a person, the images often are the request. Get the titles and states of the parent, predecessor and related items with one `ado_query_work_items` call on their ids. Done when every field, comment, image and link is in hand.
2. **Sort** each piece with the map below into exactly one digest section or the cut list.
3. **Write** the digest in the template, in the language of the request, by the **Writing** rules. Quote the item verbatim only for error messages, screen labels, parameter values and a tester's rejection.
4. **Check** before sending:
   - every acceptance criterion is a line in **Testar**; merge duplicates, drop none;
   - every open question, MEDIUM or LOW grounding note, blocking dependency, and contradiction is in **Em aberto**. A contradiction is two places promising different behaviour. Effort against SP and other metadata, the template's fixed headings, and text that was true when written are not;
   - every sentence traces to the item, its images or its links' titles; the example uses the item's own numbers, or says "exemplo ilustrativo";
   - the part above **Testar**, header and labels included, stays under 150 words;
   - read once as a stranger, nothing in the digest still sounds machine-written.

## Sort map

| Source, a `/pms:UserStory` section or field | Goes to |
|---|---|
| The requester's original text, above a "Bloco acrescentado" line | O pedido, as the authority; where the added block goes further, Em aberto |
| "Título sugerido" | the title line |
| Story, Contexto, Propósito, Âmbito, ReproSteps | O problema, O pedido |
| An AC's "DADO" case, or a Conceito flow | Exemplo |
| Método de invocação: Navegação, Elemento | Onde, as the menu path a tester clicks |
| Parametrização: parameter, country, licence, rights | Regras, only lines that gate behaviour; "Nenhum" lines are cut |
| Fora de âmbito | Fora, one line, siblings by AB# or short title |
| Acceptance criteria | Testar, one line each, without the AC-xxx ids |
| Exploração criativa "Sim", MEDIUM/LOW grounding, "a confirmar", blocking dependency | Em aberto |
| Developer Notes | O pedido on a finished item; where they differ from the ACs, Em aberto |
| A review finding pasted into Developer Notes, with a severity mark and "Fix:" | Em aberto; Estado instead when a later commit title says it was fixed |
| Tester and other human comments | Estado, the latest rejection verbatim |
| DevOpsBot commit comments | Estado, only the merge, as "MR !n merged dd/mm" |
| Persona, Classificação, Story Points derivation, HIGH grounding notes, Localização `file:line`, Critérios de sucesso that repeat the ACs, Ref de grounding, the "Story criada com /pms:UserStory" line | cut list |

## Template

Omit a section that has nothing. Em aberto always appears; when empty it reads "Nada."

```
**AB#<id>** · <type> · <state> · <StoryPoints> SP · dev <System.AssignedTo> · tester <Custom.Tester> · <Custom.Version>
<the title in at most 10 words, when the original runs longer>

**O problema.** <what the user sees today, in screen terms, in 1-2 sentences>
**O pedido.** <what happens after the change, in 1-3 sentences>
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

## Writing

The digest is plain speech a tester could read aloud. Write it clean on the first pass, and use the same rules to strip the sentences you keep from the item.

- **Concrete.** Say what happens on screen or in the data, with the number or the label. A sentence that could sit unchanged in another item says nothing, so cut it.
- **One idea per sentence.** Split a sentence the reader would have to read twice.
- **Active voice, actor named.** "A receção anula", "o servidor recusa", not "a fatura é anulada".
- **Plain verbs.** "Garantir que", "de forma a", "é importante referir", "assegurar", "viabilizar" and trailing gerunds such as "permitindo" or "garantindo" become the plain verb, or go. "Poderá eventualmente" becomes "pode".
- **One term per concept.** Use the screen's own label and repeat it. When the item mixes "re-faturação", "re-fatura" and "reemissão", pick one.
- **Punctuation.** Join clauses with periods and commas. A colon only opens a list. An aside becomes its own sentence instead of a parenthesis or a dash. Quotes are straight. A quote keeps the item's words and takes these marks too.
- **Bold** only on the template's labels.

## Survey of an Epic or Feature

1. Collect the children with WIQL `[System.Parent] = <id>`. For an Epic, repeat on each child Feature.
2. Group by state, **Por fazer** for New and Active, **Em teste** for To Test, **Feito** for Resolved and Closed.
3. One line per child: `AB#<id> · <assignee> · <what changes, at most 12 words>`. State a prefix all titles share once in the header and strip it from the lines. Take "what changes" from the title; open the item only when the title does not say it.
4. Under **Atenção**, name each pair of children where a later one replaces or contradicts an earlier one's behaviour.
5. End with the count per group.
