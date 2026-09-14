NEURAL MEMORY — Embedding RAG Lab

Ein kleines, transparentes Labor für Embeddings, Vektorräume und semantische Suche.

NEURAL MEMORY zeigt, wie Text nicht über exakte Wörter, sondern über seine Position in einem Embedding-Vektorraum auffindbar wird.

BEVOR DA SIGGI WIEDER MECKERT, EINE SACHE ZUR KLARSTELLUNG:
Das Projekt ist bewusst als sichtbares Experiment aufgebaut: Der komplette Weg von Text → Embedding → Vektor → Ähnlichkeit → Ranking soll nachvollziehbar bleiben.

-->So geht der Autor vor, wenn er etwas verstehen möchte.

⸻

1. Grundidee

Ein Text

[
x
]

wird durch ein Embedding-Modell in einen Vektor überführt:

[
f(x)=\mathbf v
]

mit

[
\mathbf v\in\mathbb R^d.
]

In diesem Projekt verwenden wir:

Xenova/multilingual-e5-small

mit

[
d=384.
]

Damit wird jeder gespeicherte Text zu einem Punkt in einem 384-dimensionalen Vektorraum.

⸻

2. Der mathematische Ablauf

Die Pipeline lautet:

Text
  ↓
Tokenisierung
  ↓
Embedding-Modell
  ↓
384-dimensionale Repräsentation
  ↓
Vector Store
  ↓
Query-Embedding
  ↓
Cosine Similarity
  ↓
Ranking
  ↓
Top-k relevante Texte

Embedding

Für einen Text (x):

[
f:\mathcal T\rightarrow\mathbb R^{384}
]

und damit:

[
f(x)=
(v_1,v_2,\ldots,v_{384}).
]

Die 384 Zahlen sind keine einzelnen Wörter.

Sie bilden gemeinsam die numerische Repräsentation des Textes im vom Modell gelernten Vektorraum.

⸻

3. Anfrage und gespeicherte Texte

Auch eine Suchanfrage wird in denselben Vektorraum übertragen.

Für die Anfrage (q):

[
f(q)=\mathbf q.
]

Für einen gespeicherten Text (x_i):

[
f(x_i)=\mathbf v_i.
]

Damit können wir (\mathbf q) mit jedem gespeicherten (\mathbf v_i) vergleichen.

⸻

4. Cosine Similarity

Die Ähnlichkeit wird über die Kosinusähnlichkeit berechnet:

[
\operatorname{sim}(\mathbf q,\mathbf v_i)

\frac{\mathbf q\cdot\mathbf v_i}
{|\mathbf q||\mathbf v_i|}.
]

Das Skalarprodukt ist:

[
\mathbf q\cdot\mathbf v_i

\sum_{j=1}^{384}q_jv_{ij}.
]

Die Länge eines Vektors ist beispielsweise:

[
|\mathbf q|

\sqrt{\sum_{j=1}^{384}q_j^2}.
]

Geometrisch betrachtet vergleichen wir damit vor allem die Richtung zweier Vektoren.

Je ähnlicher die Richtung, desto höher die Ähnlichkeit.

⸻

5. Ranking

Angenommen, wir haben (n) gespeicherte Texte.

Für jeden Text berechnen wir:

[
s_i=
\operatorname{sim}(\mathbf q,\mathbf v_i).
]

Wir erhalten:

[
s_1,s_2,\ldots,s_n.
]

Anschließend sortieren wir absteigend:

[
s_{(1)}
\ge
s_{(2)}
\ge
\ldots
\ge
s_{(n)}.
]

Die obersten Treffer bilden das Top-k-Ergebnis.

Damit beantwortet das System zunächst nur:

Welche gespeicherten Informationen sind für diese Anfrage am relevantesten?

⸻

6. Was Embeddings NICHT bedeuten

Ein hoher Similarity-Wert bedeutet nicht automatisch:

„Diese beiden Aussagen sind wahr.“

Oder:

„Diese beiden Datensätze sind identisch.“

Oder:

„Diese beiden Buchungen müssen zusammengehören.“

Das Embedding liefert eine Relevanzbeziehung, keine Wahrheit.

Deshalb trennen wir:

Embedding
    ↓
Relevanz

von:

Deterministische Regeln
    ↓
Entscheidung

Das ist eine wichtige Architekturgrenze.

⸻

7. RAG

Embedding-Suche kann als Retrieval-Schicht eines RAG-Systems verwendet werden.

RAG = Retrieval-Augmented Generation

Der Ablauf ist:

Benutzerfrage
     ↓
Query-Embedding
     ↓
Vektorsuche
     ↓
Top-k relevante Dokumente
     ↓
Kontext
     ↓
LLM
     ↓
Antwort

Das LLM erhält also nicht zwangsläufig die gesamte Datenbank.

Es bekommt zunächst die Informationen, die durch die Retrieval-Schicht als relevant gefunden wurden.

⸻

8. E5 und die Präfixe

Das verwendete Modell multilingual-e5-small unterscheidet zwischen Suchanfrage und gespeicherten Inhalten.

Gespeicherter Text:

passage: Der Kunde hat am Freitag einen Termin in Stuttgart.

Suchanfrage:

query: Wann ist der Termin des Kunden?

Im Code wird deshalb zwischen beiden Modi unterschieden:

const prepared = `${mode}: ${text.trim()}`;

wobei mode entweder query oder passage ist.

⸻

9. Beispiel

Wir speichern:

Der Kunde hat am Freitag einen Termin in Stuttgart.

und:

Die Lieferung für München erfolgt nächste Woche.

Dann fragen wir:

Wann findet der Termin des Kunden statt?

Die Anfrage wird ebenfalls in einen Vektor transformiert.

Das System vergleicht anschließend:

Query-Vektor
      │
      ├── Vektor Termin Stuttgart
      │       → hohe Ähnlichkeit
      │
      └── Vektor Lieferung München
              → geringere Ähnlichkeit

Das funktioniert auch dann, wenn die Formulierungen nicht identisch sind.

⸻

10. Warum das interessant ist

Eine klassische Suche kann beispielsweise nach dem Wort

Termin

suchen.

Embedding-Suche arbeitet dagegen auf der gelernten Repräsentation des gesamten Textes.

Dadurch können auch unterschiedliche Formulierungen als relevant erkannt werden.

Das ist der wesentliche Unterschied:

Keyword Search
    ↓
Wort / Zeichen / Muster

gegen:

Semantic Search
    ↓
Position und Richtung im Embedding-Raum

⸻

11. Die 384 Dimensionen

Die Zahl

[
384
]

bedeutet nicht:

* 384 Wörter
* 384 Bedeutungen
* 384 Kategorien

Sie ist die feste Dimension des Embedding-Modells.

Ein Text kann sehr kurz oder sehr lang sein.

Die resultierende Repräsentation besitzt trotzdem immer:

[
384
]

Komponenten.

Beispielsweise:

Text A
→ [0.02, -0.14, 0.08, ..., 0.03]
Text B
→ [0.01, -0.12, 0.07, ..., 0.04]

Beide liegen damit im selben

[
\mathbb R^{384}
]

Raum.

⸻

12. Aktuelle Implementierung

Das Projekt verwendet:

* HTML
* CSS
* JavaScript
* Transformers.js
* Xenova/multilingual-e5-small
* Cosine Similarity
* einen einfachen In-Memory Vector Store

Die Embeddings werden direkt im Browser erzeugt.

Der aktuelle Vector Store ist daher nur während der laufenden Sitzung vorhanden.

Beim Neuladen der Seite wird der Speicher zurückgesetzt.

⸻

13. Aktuelle Architektur

┌────────────────────────────────────────────┐
│              NEURAL MEMORY                 │
│                                            │
│  Text                                       │
│   ↓                                        │
│  E5 Embedding Model                        │
│   ↓                                        │
│  Vector ∈ ℝ^384                            │
│   ↓                                        │
│  Vector Store                              │
│   ↓                                        │
│  Cosine Similarity                         │
│   ↓                                        │
│  Ranking                                   │
│                                            │
└────────────────────────────────────────────┘

⸻

14. 3D-Visualisierung

Die aktuelle Visualisierung ist eine didaktische Darstellung.

Sie verwendet die ersten drei Komponenten:

[
(v_1,v_2,v_3)
]

des 384-dimensionalen Vektors.

Das bedeutet ausdrücklich:

Die Darstellung ist keine echte Projektion des gesamten 384D-Raums.

Für eine echte Visualisierung des Embedding-Raums wären beispielsweise Verfahren wie

* PCA
* t-SNE
* UMAP

möglich.

Diese könnten später ergänzt werden.

⸻

15. Embeddings und BOOKING_RECON

Die gleiche Technologie kann als zusätzliche Retrieval-Schicht für das Projekt BOOKING_RECON dienen.

Beispielsweise kann ein Datensatz als Text repräsentiert werden:

Schaltung SCH-20260914-0001.
Auftrag AUF-20260914-0042.
Datum 2026-09-14.
Status offen.
Menge 1.
Betrag 49,99 Euro.

Dieser Text kann eingebettet und anschließend semantisch gesucht werden.

Zum Beispiel:

Welche offenen Schaltungen gehören zu
Auftrag 0042?

Das Embedding-System kann relevante Datensätze als Kandidaten finden.

Die eigentliche Entscheidung bleibt jedoch deterministisch:

Embedding
    ↓
Kandidaten / Kontext
    ↓
explizite Regeln
    ↓
Reconciliation
    ↓
Ergebnis

Nicht:

Embedding
    ↓
"Das sieht ähnlich aus"
    ↓
Buchung ist identisch

⸻

16. ChromaDB

Ein Vector Store wie ChromaDB bestimmt nicht die Dimension des Embeddings.

Die Dimension wird vom Embedding-Modell bestimmt.

Bei unserem Modell:

[
d=384.
]

Der Vector Store übernimmt anschließend Aufgaben wie:

* Vektoren speichern
* IDs verwalten
* Dokumente zuordnen
* Metadaten speichern
* Ähnlichkeitssuche durchführen

Vereinfacht:

Embedding Model
      │
      │ 384D Vector
      ▼
Vector Store
      │
      ├── Vector
      ├── Document
      ├── ID
      └── Metadata

⸻

17. Der wichtigste Gedanke

NEURAL MEMORY behandelt Embeddings nicht als „magische Erinnerung“.

Es handelt sich um eine mathematische Repräsentation:

[
x
\overset{f}{\longrightarrow}
\mathbf v\in\mathbb R^{384}.
]

Danach wird mit Vektorgeometrie gearbeitet:

[
(\mathbf q,\mathbf v_i)
\longrightarrow
\operatorname{cosine\ similarity}
\longrightarrow
ranking.
]

Damit entsteht eine Relevanzschicht über Informationen.

Die eigentliche Bedeutung, Wahrheit oder Geschäftsentscheidung muss von einer dafür geeigneten weiteren Schicht kommen.

⸻

Ziel des Projekts

NEURAL MEMORY soll sichtbar machen, was hinter moderner semantischer Suche tatsächlich passiert:

Text wird zu Geometrie.
Geometrie wird zu Ähnlichkeit.
Ähnlichkeit wird zu Retrieval.
Retrieval liefert Kontext.

Und genau dort beginnt RAG.