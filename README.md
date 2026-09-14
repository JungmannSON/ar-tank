# NEURAL MEMORY — Embedding RAG Lab

Ein kleines, transparentes Labor für Embeddings, Vektorräume und semantische Suche.

NEURAL MEMORY zeigt, wie Text nicht über exakte Wörter, sondern über seine Position in einem Embedding-Vektorraum auffindbar wird.

Das Projekt ist bewusst als sichtbares Experiment aufgebaut: Der komplette Weg von Text → Embedding → Vektor → Ähnlichkeit → Ranking soll nachvollziehbar bleiben.

---

## 1. Grundidee

Ein Text

x

wird durch ein Embedding-Modell in einen Vektor überführt:

f(x) = v

mit

v ∈ R^d

In diesem Projekt verwenden wir:

Xenova/multilingual-e5-small

mit

d = 384

Damit wird jeder gespeicherte Text zu einem Punkt in einem 384-dimensionalen Vektorraum.

---

## 2. Der mathematische Ablauf

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

---

## 3. Embedding

Für einen Text x:

f : T → R^384

und damit:

f(x) = (v₁, v₂, ..., v₃₈₄)

Die 384 Zahlen sind keine einzelnen Wörter.

Sie bilden gemeinsam die numerische Repräsentation des Textes im vom Modell gelernten Vektorraum.

---

## 4. Anfrage und gespeicherte Texte

Auch eine Suchanfrage wird in denselben Vektorraum übertragen.

Für die Anfrage q:

f(q) = q

Für einen gespeicherten Text xᵢ:

f(xᵢ) = vᵢ

Damit können wir q mit jedem gespeicherten vᵢ vergleichen.

---

## 5. Cosine Similarity

Die Ähnlichkeit wird über die Kosinusähnlichkeit berechnet:

sim(q, vᵢ) =
(q · vᵢ) / (||q|| ||vᵢ||)

Das Skalarprodukt ist:

q · vᵢ = Σ(j=1 bis 384) qⱼ vᵢⱼ

Die Länge eines Vektors ist beispielsweise:

||q|| = √(Σ(j=1 bis 384) qⱼ²)

Geometrisch betrachtet vergleichen wir damit vor allem die Richtung zweier Vektoren.

Je ähnlicher die Richtung, desto höher die Ähnlichkeit.

---

## 6. Ranking

Angenommen, wir haben n gespeicherte Texte.

Für jeden Text berechnen wir:

sᵢ = sim(q, vᵢ)

Wir erhalten:

s₁, s₂, ..., sₙ

Anschließend sortieren wir absteigend:

s₍₁₎ ≥ s₍₂₎ ≥ ... ≥ s₍ₙ₎

Die obersten Treffer bilden das Top-k-Ergebnis.

Damit beantwortet das System zunächst nur:

"Welche gespeicherten Informationen sind für diese Anfrage am relevantesten?"

---

## 7. Was Embeddings NICHT bedeuten

Ein hoher Similarity-Wert bedeutet nicht automatisch:

"Diese beiden Aussagen sind wahr."

Oder:

"Diese beiden Datensätze sind identisch."

Oder:

"Diese beiden Buchungen müssen zusammengehören."

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

---

## 8. RAG

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

---

## 9. E5 und die Präfixe

Das verwendete Modell multilingual-e5-small unterscheidet zwischen Suchanfrage und gespeicherten Inhalten.

Gespeicherter Text:

passage: Der Kunde hat am Freitag einen Termin in Stuttgart.

Suchanfrage:

query: Wann ist der Termin des Kunden?

Im Code wird deshalb zwischen beiden Modi unterschieden:

const prepared = `${mode}: ${text.trim()}`;

wobei mode entweder query oder passage ist.

---

## 10. Beispiel

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
├── Vektor "Termin Stuttgart"
│       → hohe Ähnlichkeit
│
└── Vektor "Lieferung München"
        → geringere Ähnlichkeit

Das funktioniert auch dann, wenn die Formulierungen nicht identisch sind.

---

## 11. Warum das interessant ist

Eine klassische Suche kann beispielsweise nach dem Wort

"Termin"

suchen.

Embedding-Suche arbeitet dagegen auf der gelernten Repräsentation des gesamten Textes.

Dadurch können auch unterschiedliche Formulierungen als relevant erkannt werden.

Der wesentliche Unterschied:

Keyword Search
↓
Wort / Zeichen / Muster

gegen:

Semantic Search
↓
Position und Richtung im Embedding-Raum

---

## 12. Die 384 Dimensionen

Die Zahl

384

bedeutet nicht:

- 384 Wörter
- 384 Bedeutungen
- 384 Kategorien

Sie ist die feste Dimension des Embedding-Modells.

Ein Text kann sehr kurz oder sehr lang sein.

Die resultierende Repräsentation besitzt trotzdem immer:

384

Komponenten.

Beispielsweise:

Text A
→ [0.02, -0.14, 0.08, ..., 0.03]

Text B
→ [0.01, -0.12, 0.07, ..., 0.04]

Beide liegen damit im selben

R^384

Raum.

---

## 13. Aktuelle Implementierung

Das Projekt verwendet:

- HTML
- CSS
- JavaScript
- Transformers.js
- Xenova/multilingual-e5-small
- Cosine Similarity
- einen einfachen In-Memory Vector Store

Die Embeddings werden direkt im Browser erzeugt.

Der aktuelle Vector Store ist daher nur während der laufenden Sitzung vorhanden.

Beim Neuladen der Seite wird der Speicher zurückgesetzt.

---

## 14. Aktuelle Architektur

┌────────────────────────────────────────────┐
│              NEURAL MEMORY                 │
│                                            │
│  Text                                      │
│   ↓                                        │
│  E5 Embedding Model                        │
│   ↓                                        │
│  Vector ∈ R^384                            │
│   ↓                                        │
│  Vector Store                              │
│   ↓                                        │
│  Cosine Similarity                         │
│   ↓                                        │
│  Ranking                                   │
│                                            │
└────────────────────────────────────────────┘

---

## 15. 3D-Visualisierung

Die aktuelle Visualisierung ist eine didaktische Darstellung.

Sie verwendet die ersten drei Komponenten:

(v₁, v₂, v₃)

des 384-dimensionalen Vektors.

Das bedeutet ausdrücklich:

Die Darstellung ist keine echte Projektion des gesamten 384D-Raums.

Für eine echte Visualisierung des Embedding-Raums wären beispielsweise Verfahren wie

- PCA
- t-SNE
- UMAP

möglich.

Diese könnten später ergänzt werden.

---

## 16. Embeddings und BOOKING_RECON

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

Welche offenen Schaltungen gehören zu Auftrag 0042?

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

---

## 17. ChromaDB

Ein Vector Store wie ChromaDB bestimmt nicht die Dimension des Embeddings.

Die Dimension wird vom Embedding-Modell bestimmt.

Bei unserem Modell:

d = 384

Der Vector Store übernimmt anschließend Aufgaben wie:

- Vektoren speichern
- IDs verwalten
- Dokumente zuordnen
- Metadaten speichern
- Ähnlichkeitssuche durchführen

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

---

## 18. Der wichtigste Gedanke

NEURAL MEMORY behandelt Embeddings nicht als "magische Erinnerung".

Es handelt sich um eine mathematische Repräsentation:

x
↓
f
↓
v ∈ R^384

Danach wird mit Vektorgeometrie gearbeitet:

(q, vᵢ)
↓
Cosine Similarity
↓
Ranking

Damit entsteht eine Relevanzschicht über Informationen.

Die eigentliche Bedeutung, Wahrheit oder Geschäftsentscheidung muss von einer dafür geeigneten weiteren Schicht kommen.

---

## Ziel des Projekts

NEURAL MEMORY soll sichtbar machen, was hinter moderner semantischer Suche tatsächlich passiert:

Text wird zu Geometrie.

Geometrie wird zu Ähnlichkeit.

Ähnlichkeit wird zu Retrieval.

Retrieval liefert Kontext.

Und genau dort beginnt RAG.