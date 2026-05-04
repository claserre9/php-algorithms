# Chapitre 12 — Algorithmes gloutons & Diviser pour régner

## 12.1 Algorithmes gloutons (Greedy)

Un algorithme **glouton** construit une solution en faisant à chaque étape le choix **localement optimal**, en espérant que l'enchaînement de ces choix mène à la solution globalement optimale.

### Quand ça marche ?

Un algorithme glouton est correct si le problème satisfait :
1. **Propriété glouton** : il existe une solution optimale incluant le premier choix glouton
2. **Sous-structure optimale** : la solution optimale globale contient les solutions optimales des sous-problèmes

---

## 12.2 Sélection d'activités

Étant donné n activités avec des heures de début et de fin, sélectionner le maximum d'activités non chevauchantes.

```php
<?php
declare(strict_types=1);

/**
 * O(n log n) — trier par heure de fin, puis sélection gloutonne.
 * Choix glouton : toujours choisir l'activité qui finit le plus tôt.
 */
function selectionActivites(array $debut, array $fin): array
{
    $n = count($debut);

    // Créer les activités et trier par heure de fin
    $activites = [];
    for ($i = 0; $i < $n; $i++) {
        $activites[] = ['debut' => $debut[$i], 'fin' => $fin[$i], 'id' => $i];
    }
    usort($activites, fn($a, $b) => $a['fin'] <=> $b['fin']);

    $selection    = [$activites[0]['id']];
    $derniereFin  = $activites[0]['fin'];

    for ($i = 1; $i < $n; $i++) {
        // Si l'activité commence après la dernière sélectionnée
        if ($activites[$i]['debut'] >= $derniereFin) {
            $selection[]  = $activites[$i]['id'];
            $derniereFin  = $activites[$i]['fin'];
        }
    }
    return $selection;
}

$debut = [1, 3, 0, 5, 8, 5];
$fin   = [2, 4, 6, 7, 9, 9];

print_r(selectionActivites($debut, $fin));  // [0, 1, 3, 4] ou similar
```

---

## 12.3 Sac à dos fractionnaire

Contrairement au 0/1 Knapsack, on peut prendre des fractions d'objets.

```php
<?php
declare(strict_types=1);

/**
 * O(n log n) — trier par valeur/poids décroissant.
 * Choix glouton : toujours prendre l'objet avec la meilleure densité de valeur.
 */
function knapsackFractionnaire(array $poids, array $valeurs, int $capacite): float
{
    $n = count($poids);

    $items = [];
    for ($i = 0; $i < $n; $i++) {
        $items[] = [
            'poids'   => $poids[$i],
            'valeur'  => $valeurs[$i],
            'ratio'   => $valeurs[$i] / $poids[$i],
        ];
    }

    // Tri décroissant par ratio valeur/poids
    usort($items, fn($a, $b) => $b['ratio'] <=> $a['ratio']);

    $totalValeur    = 0.0;
    $poidsRestant   = $capacite;

    foreach ($items as $item) {
        if ($poidsRestant <= 0) break;

        $prendre      = min($item['poids'], $poidsRestant);
        $totalValeur += $prendre * $item['ratio'];
        $poidsRestant -= $prendre;
    }

    return $totalValeur;
}

$poids   = [10, 20, 30];
$valeurs = [60, 100, 120];
echo knapsackFractionnaire($poids, $valeurs, 50);  // 240.0
// (20 entier + 30 entier + 10/10*60... hm non)
// Ratios: 6, 5, 4 → prendre tout item0 + tout item1 + 20/30*item2 = 60+100+80=240
```

---

## 12.4 Codage de Huffman

Compression sans perte optimale basée sur les fréquences de caractères.

```php
<?php
declare(strict_types=1);

class NoeudHuffman
{
    public function __construct(
        public readonly string      $car,
        public readonly int         $freq,
        public readonly ?NoeudHuffman $gauche = null,
        public readonly ?NoeudHuffman $droite = null,
    ) {}

    public function estFeuille(): bool
    {
        return $this->gauche === null && $this->droite === null;
    }
}

/**
 * Construit l'arbre de Huffman et retourne les codes.
 * O(n log n) avec min-heap.
 */
function huffman(array $frequences): array
{
    // Min-heap par fréquence
    $heap = new SplPriorityQueue();
    $heap->setExtractFlags(SplPriorityQueue::EXTR_BOTH);

    foreach ($frequences as $car => $freq) {
        $heap->insert(new NoeudHuffman((string)$car, $freq), -$freq);  // priorité négative = min
    }

    // Fusionner tant qu'il reste plus d'un nœud
    while ($heap->count() > 1) {
        $g = $heap->extract()['data'];
        $d = $heap->extract()['data'];

        $fusion = new NoeudHuffman('', $g->freq + $d->freq, $g, $d);
        $heap->insert($fusion, -$fusion->freq);
    }

    $racine = $heap->extract()['data'];
    $codes  = [];
    genererCodes($racine, '', $codes);

    return $codes;
}

function genererCodes(NoeudHuffman $noeud, string $code, array &$codes): void
{
    if ($noeud->estFeuille()) {
        $codes[$noeud->car] = $code ?: '0';  // cas d'un seul caractère
        return;
    }
    if ($noeud->gauche !== null) genererCodes($noeud->gauche, $code . '0', $codes);
    if ($noeud->droite !== null) genererCodes($noeud->droite, $code . '1', $codes);
}

$freq  = ['a' => 45, 'b' => 13, 'c' => 12, 'd' => 16, 'e' => 9, 'f' => 5];
$codes = huffman($freq);
arsort($freq);
foreach ($codes as $car => $code) {
    echo "$car (freq={$freq[$car]}): $code\n";
}
// a: 0      (1 bit  — 45%)
// d: 101    (3 bits — 16%)
// b: 100    (3 bits — 13%)
// c: 111    (3 bits — 12%)
// e: 1101   (4 bits — 9%)
// f: 1100   (4 bits — 5%)
```

---

## 12.5 Problème du voyageur de commerce — Greedy approximation

```php
<?php
declare(strict_types=1);

/**
 * Heuristique du voisin le plus proche pour TSP.
 * Pas optimal mais O(n²) — garantit au pire 2× l'optimal.
 */
function tspVoisinPlusProche(array $distances, int $depart = 0): array
{
    $n       = count($distances);
    $visite  = array_fill(0, $n, false);
    $chemin  = [$depart];
    $visite[$depart] = true;
    $cout    = 0;

    $courant = $depart;
    for ($i = 1; $i < $n; $i++) {
        $minDist = PHP_INT_MAX;
        $suivant = -1;

        for ($j = 0; $j < $n; $j++) {
            if (!$visite[$j] && $distances[$courant][$j] < $minDist) {
                $minDist = $distances[$courant][$j];
                $suivant = $j;
            }
        }

        $visite[$suivant] = true;
        $chemin[]         = $suivant;
        $cout            += $minDist;
        $courant          = $suivant;
    }

    // Retour au départ
    $cout    += $distances[$courant][$depart];
    $chemin[] = $depart;

    return ['chemin' => $chemin, 'cout' => $cout];
}
```

---

## 12.6 Paradigme Diviser pour Régner

**Diviser pour régner** (Divide and Conquer) découpe le problème en sous-problèmes de même nature, résout chacun indépendamment, puis combine les résultats.

```
Résoudre(P)
├── Si P est petit → résoudre directement
├── Sinon :
│   ├── Diviser P en P₁, P₂, ..., Pₖ
│   ├── Résoudre(P₁), Résoudre(P₂), ..., Résoudre(Pₖ)
│   └── Combiner les résultats
```

### Théorème maître (analyse de complexité)

Pour T(n) = a·T(n/b) + f(n) :

- Si f(n) = O(n^(log_b a - ε)) → T(n) = Θ(n^(log_b a))
- Si f(n) = Θ(n^(log_b a)) → T(n) = Θ(n^(log_b a) · log n)
- Si f(n) = Ω(n^(log_b a + ε)) → T(n) = Θ(f(n))

Exemples :
- Merge Sort : T(n) = 2T(n/2) + O(n) → O(n log n)
- Recherche binaire : T(n) = T(n/2) + O(1) → O(log n)
- Strassen : T(n) = 7T(n/2) + O(n²) → O(n^2.81)

---

## 12.7 Exemples classiques

### 12.7.1 Multiplication de grands entiers (Karatsuba)

```php
<?php
declare(strict_types=1);

/**
 * Karatsuba — O(n^1.585) vs O(n²) pour la multiplication naïve
 */
function karatsuba(string $x, string $y): string
{
    // Cas de base
    if (strlen($x) <= 1 || strlen($y) <= 1) {
        return bcmul($x, $y);
    }

    // Égaliser les longueurs
    $n  = max(strlen($x), strlen($y));
    $m  = intdiv($n, 2);

    // x = a·10^m + b, y = c·10^m + d
    $a = substr($x, 0, -$m) ?: '0';
    $b = substr($x, -$m);
    $c = substr($y, 0, -$m) ?: '0';
    $d = substr($y, -$m);

    $ac    = karatsuba($a, $c);
    $bd    = karatsuba($b, $d);
    $abcd  = karatsuba(bcadd($a, $b), bcadd($c, $d));
    $adbc  = bcsub(bcsub($abcd, $ac), $bd);

    // Résultat = ac·10^(2m) + adbc·10^m + bd
    return bcadd(
        bcadd(
            bcmul($ac, bcpow('10', (string)(2 * $m))),
            bcmul($adbc, bcpow('10', (string)$m))
        ),
        $bd
    );
}
```

### 12.7.2 Problème de la plus proche paire de points

```php
<?php
declare(strict_types=1);

/**
 * Plus proche paire de points — O(n log n)
 * Brute force serait O(n²)
 */
function plusProchePaire(array $points): float
{
    // Trier par x
    usort($points, fn($a, $b) => $a[0] <=> $b[0]);
    return plusprocheRec($points, 0, count($points) - 1);
}

function plusprocheRec(array $pts, int $g, int $d): float
{
    if ($d - $g <= 2) {
        return minDistanceBrute($pts, $g, $d);
    }

    $m       = intdiv($g + $d, 2);
    $xMid    = $pts[$m][0];

    $dg      = plusprocheRec($pts, $g, $m);
    $dd      = plusprocheRec($pts, $m + 1, $d);
    $delta   = min($dg, $dd);

    // Points dans la bande verticale [-delta, +delta] autour de xMid
    $bande = [];
    for ($i = $g; $i <= $d; $i++) {
        if (abs($pts[$i][0] - $xMid) < $delta) {
            $bande[] = $pts[$i];
        }
    }

    // Trier la bande par y et chercher les paires dans la bande
    usort($bande, fn($a, $b) => $a[1] <=> $b[1]);

    $minBande = $delta;
    for ($i = 0; $i < count($bande); $i++) {
        for ($j = $i + 1; $j < count($bande) && ($bande[$j][1] - $bande[$i][1]) < $minBande; $j++) {
            $dist     = sqrt(($bande[$i][0] - $bande[$j][0])**2 + ($bande[$i][1] - $bande[$j][1])**2);
            $minBande = min($minBande, $dist);
        }
    }

    return min($delta, $minBande);
}

function minDistanceBrute(array $pts, int $g, int $d): float
{
    $min = PHP_FLOAT_MAX;
    for ($i = $g; $i <= $d; $i++) {
        for ($j = $i + 1; $j <= $d; $j++) {
            $dist = sqrt(($pts[$i][0] - $pts[$j][0])**2 + ($pts[$i][1] - $pts[$j][1])**2);
            $min  = min($min, $dist);
        }
    }
    return $min;
}
```

### 12.7.3 Maximum de sous-tableau par D&C

```php
<?php
declare(strict_types=1);

// O(n log n) — moins efficace que Kadane mais démontre D&C
function maxSousTableauDC(array $arr, int $g, int $d): int
{
    if ($g === $d) return $arr[$g];

    $m = intdiv($g + $d, 2);

    return max(
        maxSousTableauDC($arr, $g, $m),        // dans la moitié gauche
        maxSousTableauDC($arr, $m + 1, $d),    // dans la moitié droite
        maxChevauchant($arr, $g, $m, $d)       // chevauchant le milieu
    );
}

function maxChevauchant(array $arr, int $g, int $m, int $d): int
{
    // Extension vers la gauche
    $sommeG = PHP_INT_MIN;
    $somme  = 0;
    for ($i = $m; $i >= $g; $i--) {
        $somme  += $arr[$i];
        $sommeG  = max($sommeG, $somme);
    }

    // Extension vers la droite
    $sommeD = PHP_INT_MIN;
    $somme  = 0;
    for ($i = $m + 1; $i <= $d; $i++) {
        $somme  += $arr[$i];
        $sommeD  = max($sommeD, $somme);
    }

    return $sommeG + $sommeD;
}

$arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4];
echo maxSousTableauDC($arr, 0, count($arr) - 1);  // 6
```

---

## 12.8 Comparaison : Greedy vs DP vs D&C

| Critère | Greedy | DP | D&C |
|---------|--------|-----|-----|
| Sous-problèmes | Indépendants | Chevauchants | Indépendants |
| Ordre de résolution | Un seul passage | Tous les sous-problèmes | Diviser puis combiner |
| Cache nécessaire | Non | Oui (mémo/table) | Non |
| Complexité typique | O(n log n) | O(n²) ou O(n·k) | O(n log n) |
| Exemple | Huffman, Dijkstra | Knapsack, LCS | Merge Sort, FFT |
| Optimal ? | Pas toujours | Oui (si applicable) | Oui |

---

## 12.9 Algorithmes gloutons sur les graphes

```php
<?php
declare(strict_types=1);

// ── Algorithme de Prim (Arbre couvrant minimal) ──────────────
// O(E log V) avec min-heap
function prim(array $adjList, int $n): array
{
    $cle     = array_fill(0, $n, PHP_INT_MAX);
    $parent  = array_fill(0, $n, -1);
    $dansMST = array_fill(0, $n, false);
    $cle[0]  = 0;

    $heap = new SplMinHeap();
    $heap->insert([0, 0]);  // [cout, sommet]

    $cout = 0;
    while (!$heap->isEmpty()) {
        [$c, $u] = $heap->extract();

        if ($dansMST[$u]) continue;
        $dansMST[$u] = true;
        $cout       += $c;

        foreach ($adjList[$u] as [$v, $w]) {
            if (!$dansMST[$v] && $w < $cle[$v]) {
                $cle[$v]    = $w;
                $parent[$v] = $u;
                $heap->insert([$w, $v]);
            }
        }
    }

    return ['parent' => $parent, 'cout' => $cout];
}

// ── Coloration gloutonne d'un graphe ─────────────────────────
// Pas nécessairement optimal mais simple
function colorationGloutonne(array $adj, int $n): array
{
    $couleurs = array_fill(0, $n, -1);
    $couleurs[0] = 0;

    for ($u = 1; $u < $n; $u++) {
        $disponibles = array_fill(0, $n, true);

        // Marquer les couleurs des voisins
        foreach ($adj[$u] as $v) {
            if ($couleurs[$v] !== -1) {
                $disponibles[$couleurs[$v]] = false;
            }
        }

        // Assigner la première couleur disponible
        for ($c = 0; $c < $n; $c++) {
            if ($disponibles[$c]) {
                $couleurs[$u] = $c;
                break;
            }
        }
    }
    return $couleurs;
}
```

---

## Exercices

!!! exercise "Exercice 12.1 — Stations d'essence"
    Vous faites un trajet de A à B avec n stations d'essence sur le chemin. Votre réservoir peut contenir k litres. Trouvez le nombre minimum d'arrêts. Algorithme glouton O(n log n).

!!! exercise "Exercice 12.2 — Multiplication rapide (FFT)"
    Expliquez comment la Transformée de Fourier Rapide (FFT) peut être utilisée pour multiplier deux polynômes en O(n log n) au lieu de O(n²). Implémentez la version récursive.

!!! exercise "Exercice 12.3 — Tâches avec délais et pénalités"
    Étant donné n tâches, chacune avec un délai (deadline) et une pénalité si non complétée à temps, ordonnez-les pour minimiser la pénalité totale. Démontrez que l'algorithme glouton par pénalité décroissante est optimal.

!!! exercise "Exercice 12.4 — Bilan final : comparer les trois paradigmes"
    Pour le problème des pièces de monnaie :
    - Greedy fonctionne-t-il toujours ? Donnez un contre-exemple.
    - Implémentez la solution DP.
    - Y a-t-il une solution D&C efficace ?

---

## Récapitulatif général du tutoriel

```
Partie 1 — Fondations
├── Complexité Big O     → analyser tout algorithme
├── Tableaux             → O(1) accès, O(n) insert/delete en milieu
├── Piles / Files        → LIFO/FIFO, O(1) toutes ops
└── Listes chaînées      → O(1) insert en tête, O(n) accès

Partie 2 — Structures avancées
├── Tables de hachage    → O(1) amorti get/put/delete
├── Arbres BST/AVL       → O(log n) garanti avec équilibrage
├── Tas / PQ             → O(log n) insert/extract, O(1) min/max
└── Graphes              → BFS O(V+E), Dijkstra O((V+E)logV)

Partie 3 — Algorithmes
├── Tri                  → n log n optimal, O(n) pour entiers
├── Recherche            → O(log n) binaire, O(n+m) KMP
├── PD                   → mémoïsation ou tabulation, O(n²) typique
└── Glouton / D&C        → O(n log n), preuves d'optimalité
```
