# Sententia Summarize

Plugin gratuito per Claude che, quando scrivi in chat il riferimento a una sentenza svizzera, la cerca nelle banche dati pubbliche e ti risponde in un formato fisso e sobrio. Nessun login, nessuna configurazione, nessun dato salvato.

> **Avviso importante.** Le risposte sono generate con l'aiuto dell'intelligenza artificiale a partire da testi pubblici. Sono uno strumento di supporto, non una consulenza legale e non sostituiscono la lettura della sentenza. Verifica sempre il testo originale prima di citarlo in un atto o di fondarvi una decisione.

## Cosa fa

Tre funzioni, tutte attivate scrivendo in linguaggio naturale (anche in tedesco, francese o inglese: il plugin risponde nella lingua della tua richiesta).

**1. Riassunto di una sentenza**

> riassumi 115 II 15

Restituisce una tabella con numero, data e tribunale, poi quattro punti fissi: fattispecie, articoli principali citati, considerandi principali, conclusioni.

**2. Un considerando preciso**

> cerca BGE 142 II 481 E. 3.3

Restituisce il considerando richiesto **testuale, nella lingua originale della sentenza**. Se la lingua della tua richiesta è diversa, aggiunge sotto una traduzione, chiaramente distinta dal testo originale.

**3. Verifica di un'affermazione**

> La controparte cita la DTF 140 III 86 consid. 4.2 sostenendo che [affermazione]. È vero?

Controlla se la sentenza (o quel considerando) dice davvero quello che viene sostenuto. L'esito può essere: confermata, parzialmente confermata, non confermata, smentita, non correlata. Segue un breve rapporto in prosa. Utile quando la controparte cita una sentenza a sostegno di una tesi.

Formati di citazione riconosciuti: DTF/BGE/ATF (anche solo `115 II 15`), numeri di incarto federali (`6B_51/2021`) e sentenze cantonali.

## Cosa NON fa

- Non inventa riassunti: se non riesce a recuperare il testo reale di una sentenza, non produce nulla in questi formati e lascia che Claude risponda normalmente.
- Non si attiva quando citi una sentenza solo di passaggio in un discorso su altro.
- Non salva nulla: nessuno storico, nessun account.
- Non cerca sentenze per argomento (solo per citazione).

## Installazione

In Claude Code:

```
/plugin marketplace add sententiaki/sententia-summarize
/plugin install sententia-summarize@sententia-summarize
```

Il plugin è pensato per Claude (Claude Code, Claude Desktop, claude.ai). L'uso su Cowork Desktop dovrebbe funzionare perché usa solo connessioni HTTP, ma **non è ancora stato verificato**. Non è disponibile per ChatGPT.

## Su Cowork: abilitare i collegamenti (una sola volta)

Dopo l'installazione, Cowork può tenere spenti i collegamenti ai servizi del plugin finché non li abiliti tu:

1. Apri **Personalizza → Sfoglia plugin → Personale** e scegli `sententia-summarize`.
2. Vai su **Gestisci → Connettori** e controlla che i due collegamenti del plugin (`entscheidsuche` e `swiss-caselaw`) siano **accesi**.
3. Per ciascuno scegli **Consenti sempre**.
4. Apri una conversazione **nuova**.

Se i collegamenti non sono attivi, il riassunto e la verifica di un'affermazione funzionano comunque, leggendo il testo dalle fonti ufficiali sul web, ma con meno controlli; il **considerando testuale** richiede invece i collegamenti attivi.

## Da dove arrivano i dati

Il plugin non ha un server proprio. Si collega a due servizi pubblici gestiti da terzi:

| Servizio | Indirizzo | A cosa serve |
|---|---|---|
| OpenCaseLaw | `mcp.opencaselaw.ch` | Sentenze federali, verifica delle citazioni, considerandi |
| entscheidsuche.ch (server ufficiale dell'associazione) | `mcp.entscheidsuche.ch` | Ricerca per numero di incarto nelle sentenze federali e cantonali |

Le sentenze sono testi pubblici dei tribunali svizzeri. Se questi servizi non sono raggiungibili, il plugin non può rispondere.

## Privacy

- Il plugin non salva dati e non ha account.
- Quando lo usi, il testo che scrivi (il numero di sentenza, e nel caso della verifica anche l'affermazione) viene inviato ai servizi elencati sopra e, come ogni messaggio in chat, ad Anthropic.
- **Non incollare dati identificativi di clienti o informazioni coperte dal segreto professionale.** Usa solo riferimenti a sentenze pubbliche e affermazioni generali.

## Limiti noti

- La copertura delle sentenze cantonali dipende dalle banche dati pubbliche e non è garantita per tutti i cantoni.
- Il riassunto è una sintesi: può omettere dettagli rilevanti. Il considerando testuale è la fonte da citare.
- La traduzione dei considerandi è un aiuto alla lettura, non un testo ufficiale.

## Licenza

Apache License 2.0 — vedi [LICENSE](LICENSE) e [NOTICE](NOTICE). La licenza riguarda solo i file di questo plugin: non concede diritti sul nome "Sententia" e non regola le condizioni d'uso dei servizi esterni da cui arrivano le sentenze.

Il plugin è fornito "così com'è", senza garanzie. Resta valido l'avviso in cima a questo file: le risposte vanno sempre verificate sul testo originale.
