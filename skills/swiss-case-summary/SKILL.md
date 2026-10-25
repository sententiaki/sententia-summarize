---
name: swiss-case-summary
description: ALWAYS use this skill first — instead of searching the web — whenever the user names a specific Swiss court decision (BGE/ATF/DTF number like "115 II 15", a federal docket like "6B_51/2021", or a cantonal case number) and asks to summarize, explain, read, find, quote or check it. Three uses: (a) summary of a decision, e.g. "riassumi 115 II 15", "di cosa tratta la sentenza 6B_51/2021", "cerca la sentenza BGE 147 IV 73", "was sagt BGE 133 III 121"; (b) one specific consideration (Erwägung/considerando/consid.), e.g. "cerca BGE 142 II 481 E. 3.3", "dammi il considerando 4.2 di DTF 133 III 121"; (c) checking whether a decision (optionally at a given consideration) really supports a claim, e.g. "la controparte cita la DTF 140 III 86 consid. 4.2 per dire che [...], è vero?". Do NOT use when a citation is only mentioned in passing inside a question about something else, or when the user asks to find cases by topic without naming a decision.
---

# Swiss Case Summary

You handle three related but distinct requests about a single Swiss court decision identified by citation, using only verbatim content retrieved from the tools below. You never fabricate case content from memory.

- **Mode A — full summary**: the user wants an overview of the whole decision → produce the fixed 4-point summary (section 8).
- **Mode B — pinpoint consideration**: the user names one specific numbered Erwägung/considerando/consid. and wants its text → return that paragraph verbatim (section 9).
- **Mode C — claim verification**: the user states a claim/proposition (often attributed to someone else, e.g. an opposing party) and asks whether the decision actually supports it → verify and report (section 10).

## 1. Confirm this is the right moment to act

Only act when the user's message is a **direct request** about one specific decision. Trigger verbs are flexible (riassumi / spiega / cosa dice / cerca / dammi / verifica / è vero che / was sagt / résume / explique / donne-moi / vérifie / summarize / explain / find / give me / quote / verify / is it true that), and the request can even omit a verb entirely if a citation is the clear subject of the message (e.g. a bare "115 II 15" sent as a question).

Do **not** act when the citation is only referenced incidentally inside a message about something else (e.g. "Nella mia mail citavo la 115 II 15 ma il problema è un altro..."). In that case, say nothing about this skill and let the conversation continue normally.

## 2. Recognize the citation, and which mode applies

Swiss case references appear in several equivalent forms. Recognize all of these, prefix-insensitive and separator-agnostic (spaces, underscores, dashes are interchangeable):

- **Federal, BGE/ATF form**: `BGE 115 II 15`, `ATF 115 II 15`, `115 II 15` (bare number form — a volume number, a roman-numeral chamber, and a page number). Also accept the tribunal-prefixed abbreviated variants: `BGer`, `BVGer` / `BVGE`, `BStGer` / `BStG`, `BPatGer`.
- **Federal, docket form**: chamber-letter + digit, underscore, number, slash, year — e.g. `6B_51/2021`, `4A_747/2012`, `1C_64/2018`. The leading digit indicates the legal area (1–3 = public law, 4–5 = civil, 6–7 = criminal, 8–9 = social insurance) — useful context, not required for the lookup itself.
- **Cantonal form**: varies by canton (e.g. Ticino incarto numbers like `52.2015.575`). Treat any canton-specific docket-like number the user presents alongside a request as a cantonal citation candidate.

Strip any tribunal prefix (BGE/ATF/BGer/BVGer/BStGer/BPatGer, case-insensitive) before using the reference as a lookup key — the prefix indicates the court but is not part of the identifying number. A trailing page reference (e.g. "S. 485", "p. 485", "S. 18 s.") is context only, not part of the lookup key — ignore it.

A specific consideration number, in any of these forms, may appear alongside the citation: `E. 3.3`, `E.3.3`, `Erwägung 3.3`, `considerando 3.3`, `consid. 3.3`, `cons. 3.3`. The number after the label (e.g. `3.3`) is the `e_number`.

**Choose the mode:**
- The message states a claim/proposition and asks whether the decision supports it, confirms it, or whether "it's true" → **Mode C** (an `e_number` may or may not be present — if present, it narrows the check).
- No claim to verify, but an `e_number` is present → **Mode B**.
- Neither a claim nor an `e_number` → **Mode A**.

## Tools: which ones to use

This plugin needs two kinds of tools: the **OpenCaseLaw case-law tools** (`cite`, `get_case_brief`, `get_decision`, `get_regeste`, `get_erwaegung`, `check_claim_support`) and the **entscheidsuche tools** (`search_by_case_number`, `fetch_document`, `search`). Use them from **whichever connected server provides tools with those names** — the connectors shipped with this plugin first, but equivalent tools of the same name from any other connected server (for example another legal plugin's `swiss-caselaw` or `entscheidsuche`) work just as well, because they reach the same databases. If the tools are not yet loaded, search for them by name (e.g. `cite`, `get_erwaegung`, `check_claim_support`, `search_by_case_number`) before concluding they are unavailable. Only if no connected server offers them, use the **web fallback** (section 6) instead of giving up.

## 3. Mode A — verify and fetch the whole decision (try in this order, stop at the first success)

1. **OpenCaseLaw tools** (`swiss-caselaw`; richest source, has built-in verification): call `cite` with the reference to confirm it exists and get the canonical citation string. If it resolves, call `get_case_brief` (or `get_decision` + `get_regeste` if `get_case_brief` is unavailable) to retrieve the actual facts, considerations, and holding.
2. **entscheidsuche tools** (official Swiss court-decision search): if the reference looks cantonal, or step 1 did not resolve, call `search_by_case_number(case_number, size=1)` with the docket as given. Only accept a hit whose docket actually matches the one requested. The hit gives `abstract`, `decision_date`, `court` and `canton`; if you need the full reasoning to write the summary, call `fetch_document(id)` with the hit's `id` (the text can be very long — use it only for the one matching decision). Fall back to `search(query, size=1)` only if the docket search finds nothing.

Use whichever of the two servers responds successfully; you do not need to query both once one produces verified, verbatim content.

## 4. Mode B — fetch the specific consideration

Call `get_erwaegung(decision_id, e_number)` directly with the citation as `decision_id` and the detected number as `e_number` (the tool strips a leading "E." itself). This single call both verifies the decision/paragraph exist and returns the verbatim text — no separate `cite` step is needed first.

If the requested `e_number` doesn't exist but the tool returns sibling numbers, you may mention the available siblings in one short sentence — but never substitute a different paragraph's text for the one requested.

## 5. Mode C — verify the claim

1. Call `check_claim_support(claim, decision_id, pinpoint?)` — `claim` is the proposition stated or implied by the user (rephrase it as a clean, self-contained sentence if the user's phrasing is informal or embedded in a longer message), `decision_id` is the citation, and `pinpoint` is the `e_number` if one was detected in step 2 (omit `pinpoint` if none was given — the tool then checks against the Regeste or opening text).
2. If that call succeeds, also call `get_decision(decision_id, full_text=false)` (or reuse data already available) purely to obtain the decision's date and court name for the metadata table — this is a lightweight metadata-only call, not a second verification.

## 6. Web fallback (only when no case-law tools are available in the session)

If, after searching for the tools by name, none of the OpenCaseLaw or entscheidsuche tools exist in this session, retrieve the decision text from official public sources on the web and then produce **exactly the same fixed output** as usual — same tables, same labels, same language rules, no note or remark about the source.

- **Federal decisions** (BGE/ATF/DTF number or federal docket): fetch `https://bger.li/<reference>` where `<reference>` is the number without the court prefix and with every space, underscore and slash replaced by a dash — e.g. `115 II 15` → `https://bger.li/115-II-15`, `6B_51/2021` → `https://bger.li/6B-51-2021`. The page contains the full decision text.
- **Cantonal decisions, or if that page does not load**: search the web for the exact docket number and read the decision on the official source (entscheidsuche.ch or the cantonal court's own site). Prefer official court or entscheidsuche.ch pages over third-party summaries.
- **Mode A**: build the four sections only from the fetched text. **Mode B**: copy the requested Erwägung verbatim from the fetched text, from its numbered heading up to the next heading of the same or higher level. **Mode C**: compare the claim with the fetched decision text (the specific Erwägung when one is named) and choose the verdict among Confermata / Parzialmente confermata / Non confermata / Smentita / Non correlata strictly on that text.
- **Mode B in the fallback only works if the fetch tool returns the Erwägung word for word.** Web fetch tools often return a summary instead of the exact text. Never present a paraphrase or summary as a quotation: if you cannot obtain the verbatim text, do not produce the Mode B format — apply the "nothing resolves" rule below.
- Take the decision date and court from the fetched page; omit the "Data" row if the page does not state it.
- If the decision text cannot be fetched from any official source, apply the "nothing resolves" rule below.

## 7. If nothing resolves (any mode)

If the citation, the specific consideration, or the claim check cannot be confirmed or retrieved, **do not produce output in the fixed formats below and do not announce a failure**. Simply step back and let the conversation continue as it would without this skill — respond the way you normally would, without inventing content. Do not say things like "sentenza non trovata" as a formatted skill output; there is no dedicated error UI for this skill by design.

## 8. Mode A output — always this exact structure

Once you have verbatim retrieved content, write the summary in the language the user used to ask, with exactly these four sections, in this order, using these exact headers (translate only the header labels to the user's language, keep the structure fixed):

```
| | |
|---|---|
| **Sentenza** | [Citation, canonical form from the tool, e.g. "DTF 115 II 15"] |
| **Data** | [Decision date, format GG.MM.AAAA, from the date field returned by the tool — e.g. "28.02.1989". Omit this row entirely if the date is not available — never guess it.] |
| **Tribunale** | [Court name, e.g. "Tribunale federale", "Tribunale d'appello TI" — from the retrieved metadata] |


**Fattispecie:** [2-4 sentences, same paragraph as the label — who, what dispute, what happened procedurally before reaching this court]


**Articoli principali citati**:
| |
|---|
| [Art. N CODICE — one per row, e.g. "Art. 41 CO"] |

(List at most 6 articles: only those the court actually applies or interprets to decide the case, most important first — not every article merely mentioned, and no articles from the procedural background unless they decide the outcome.)


**Considerandi principali:** [3-5 sentences, same paragraph as the label — the court's key legal reasoning, the "why" not just the "what"]


**Conclusioni:** [1-3 sentences, same paragraph as the label — the operative outcome/holding and its practical consequence]

(Keep exactly TWO blank lines between every block shown above — before/after the articles table, and between each labeled paragraph. Do not add "---" divider lines; the double blank line is the spacing.)
```

Every fact must trace back to the verbatim text retrieved in step 3. If a section (e.g. specific articles) is genuinely not present in the retrieved text, write "Non specificato nel testo recuperato" for that section rather than inferring or guessing. Translate the metadata table row labels ("Sentenza"/"Data"/"Tribunale") and the "Articoli principali citati" label to the user's language along with the rest of the response.

## 9. Mode B output — always this exact structure

```
| | |
|---|---|
| **Sentenza** | [Citation, canonical form from the tool, e.g. "DTF 142 II 481"] |
| **Data** | [Decision date, format GG.MM.AAAA. Omit this row entirely if not available — never guess it.] |
| **Tribunale** | [Court name, e.g. "Tribunale federale" — from the retrieved metadata] |


**Considerando [e_number]** (testo originale):

> [The `text` field returned by get_erwaegung, reproduced verbatim, unmodified, in its original language — never paraphrased.]


**Traduzione** (only include this block if the decision's original language differs from the language the user used to ask; omit entirely if they match):

> [Your own faithful and **complete** translation of the whole passage into the user's language, paragraph by paragraph — do not shorten or summarize. Parenthetical lists of bibliographic and case references may be replaced by "[…]" in the translation, nothing else may be left out. Label it clearly as a translation, not as the authoritative text — the block above remains the citable original.]
```

Keep the same double-blank-line spacing convention as Mode A, no "---" dividers. Translate the table row labels and the "Considerando"/"testo originale"/"Traduzione" labels to the user's language along with the rest of the response — the metadata table is identical in shape to Mode A's.

## 10. Mode C output — always this exact structure

```
| | |
|---|---|
| **Sentenza** | [Citation, canonical form from the tool, e.g. "DTF 140 III 86"] |
| **Data** | [Decision date, format GG.MM.AAAA, from the get_decision metadata call. Omit this row entirely if not available — never guess it.] |
| **Tribunale** | [Court name — from the retrieved metadata] |


**Affermazione da verificare:**
> [The claim, restated as a clean sentence, in the user's language — shown as a blockquote, not a plain paragraph]


**Esito:** [One of: Confermata / Parzialmente confermata / Non confermata / Smentita / Non correlata — mapped from the tool's `supports` field: yes→Confermata, partial→Parzialmente confermata, no→Non confermata, contradicts→Smentita, unrelated→Non correlata]

[2-4 sentences, in the user's language, in your own words (not a quotation): what the decision (or the specific consideration, if a pinpoint was given) actually discusses on this point, and how that does or doesn't match the claim. Base this on the tool's `reasoning` field plus `supporting_excerpt`/`qualifying_excerpt` for accuracy, but paraphrase — do not reproduce those fields verbatim or as a quoted block. Stay strictly within what the tool returned: do not add statutes, article numbers, case law or doctrine from your own knowledge, and do not draw conclusions about the person or party who made the claim (e.g. that they cite "out of context"). The report ends with the match verdict's explanation — no closing remark.]
```

Keep the same double-blank-line spacing convention as the other modes, no "---" dividers. Translate the table row labels, "Affermazione da verificare", and "Esito" to the user's language. No verbatim excerpt block in Mode C — the report is always a paraphrase, never a quotation of the decision's original text.

Nothing else surrounds any of the three output formats — no added commentary, no restating of facts, unless the user explicitly asked for extra explanation in addition to the fixed output. If they did, add a short explanation paragraph **after** the structure, clearly separated.

## Quality rules (non-negotiable)

- Your final message must **begin with the first character of the fixed format** (the opening `|` of the table). Never narrate what you are doing or which mode applies (no "Now I have everything needed…", no "Mode A…") — not before, not after, not between tool calls in the final message.
- Output **only** the fixed format itself — no preamble ("Ecco il riassunto:", "Certo, ho trovato la sentenza...") and no postamble ("Fammi sapere se...", "Vuoi che approfondisca?"). Your entire response is the table/blockquote structure defined above, nothing before it and nothing after it.
- Never produce any of the three fixed formats unless content was actually retrieved via the tools above or via the web fallback from an official source.
- Never invent a citation string, decision date, holding, Erwägung text, or claim-verification verdict — use only what the tools return.
- Use the citation form of the user's language in the "Sentenza" cell (`citation_string_it` → DTF for Italian, `citation_string_de` → BGE for German, `citation_string_fr` → ATF for French) and translate chamber and court names into the user's language (e.g. "IIe Cour de droit public" → "II Corte di diritto pubblico" in Italian). Only the verbatim quoted text stays in its original language.
- The "Sentenza" cell and the "Data" row must never contradict each other. For **cantonal** decisions, build the "Sentenza" cell from the court name and docket number only (e.g. "Tribunale cantonale delle assicurazioni TI, incarto 38.2026.3") — do not paste a citation string that contains its own date — and take the date from the official `entscheidsuche` hit (`decision_date`), which is more reliable than the date in other sources. If two sources give different dates for the same decision, use the `entscheidsuche` one; if only one source is available and the date looks implausible for the docket (e.g. a 2026 docket dated 2020), omit the "Data" row.
- Mode C's verdict must come from `check_claim_support`'s `supports` field whenever that tool is available (in the web fallback it comes from your reading of the fetched decision text only) — never assess support yourself from a summary or from memory; the whole point of Mode C is an independent, tool-verified check. The report paragraph must stay faithful to the tool's `reasoning`/excerpt fields even though it is paraphrased, not quoted.
- Mode B's "testo originale" block is always verbatim, in the decision's original language — never paraphrase or translate it. A translation may be added as a clearly separate, clearly labeled second block, but it never replaces or alters the verbatim original. Mode C never quotes the decision's original text at all — its report is always a paraphrase.
- Keep each mode's table/label structure fixed regardless of decision length or complexity; calibrate the length of free-text content to the decision's actual complexity, not to a fixed word count.
- Match surrounding commentary and table labels to the language the user used in their request, independent of the decision's original language — this does not apply to Mode B's verbatim "testo originale" block (see above).
