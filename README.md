1. Text wird zu einem Vektor

Wir haben einen Text x. Ein Embedding-Modell bildet ihn auf einen festen Vektor ab:

f(x)=\mathbf v

mit

\mathbf v=(v_1,v_2,\ldots,v_d)\in\mathbb R^d

Bei unserem aktuellen Modell ist

d=384.

Also: jeder Text wird zu einem Punkt in einem 384-dimensionalen Vektorraum.

⸻

2. Anfrage wird genauso abgebildet

Für eine Suchanfrage q:

f(q)=\mathbf q

Damit liegen Anfrage und gespeicherte Texte im gleichen Vektorraum.

⸻

3. Ähnlichkeit über den Winkel

Wir berechnen zwischen Anfragevektor \mathbf q und gespeicherten Vektor \mathbf v_i:

\operatorname{sim}(\mathbf q,\mathbf v_i)
=
\frac{\mathbf q\cdot\mathbf v_i}
{\|\mathbf q\|\|\mathbf v_i\|}

Dabei ist

\mathbf q\cdot\mathbf v_i
=
\sum_{j=1}^{d}q_jv_{ij}.

Das ist das Skalarprodukt.

Die Norm ist:

\|\mathbf q\|
=
\sqrt{\sum_{j=1}^{d}q_j^2}.

⸻

4. Ranking

Wir berechnen diese Zahl für alle gespeicherten Vektoren:

s_1,s_2,\ldots,s_n

und sortieren:

s_{(1)}\ge s_{(2)}\ge\ldots\ge s_{(n)}.

Die höchsten Werte sind die semantisch relevantesten Treffer.