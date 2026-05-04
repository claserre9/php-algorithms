# Chapitre 9 — Algorithmes de tri

## 9.1 Vue d'ensemble

| Algorithme | Temps (moy) | Temps (pire) | Espace | Stable ? | In-place ? |
|-----------|-------------|--------------|--------|----------|-----------|
| Bubble Sort | O(n²) | O(n²) | O(1) | Oui | Oui |
| Selection Sort | O(n²) | O(n²) | O(1) | Non | Oui |
| Insertion Sort | O(n²) | O(n²) | O(1) | Oui | Oui |
| Merge Sort | O(n log n) | O(n log n) | O(n) | Oui | Non |
| Quick Sort | O(n log n) | O(n²) | O(log n) | Non | Oui |
| Heap Sort | O(n log n) | O(n log n) | O(1) | Non | Oui |
| Counting Sort | O(n+k) | O(n+k) | O(k) | Oui | Non |
| Radix Sort | O(d·n) | O(d·n) | O(n+k) | Oui | Non |
| Tim Sort | O(n log n) | O(n log n) | O(n) | Oui | Non |

**Stable** = préserve l'ordre relatif des éléments égaux.

---

## 9.2 Tri à bulles (Bubble Sort)

```php
<?php
declare(strict_types=1);

// Version de base — O(n²) toujours
function bubbleSort(array $arr): array
{
    $n = count($arr);
    for ($i = 0; $i < $n - 1; $i++) {
        for ($j = 0; $j < $n - $i - 1; $j++) {
            if ($arr[$j] > $arr[$j + 1]) {
                [$arr[$j], $arr[$j + 1]] = [$arr[$j + 1], $arr[$j]];
            }
        }
    }
    return $arr;
}

// Version optimisée — O(n) si déjà trié
function bubbleSortOpt(array $arr): array
{
    $n = count($arr);
    for ($i = 0; $i < $n - 1; $i++) {
        $echange = false;
        for ($j = 0; $j < $n - $i - 1; $j++) {
            if ($arr[$j] > $arr[$j + 1]) {
                [$arr[$j], $arr[$j + 1]] = [$arr[$j + 1], $arr[$j]];
                $echange = true;
            }
        }
        if (!$echange) break;  // aucun échange = déjà trié
    }
    return $arr;
}

print_r(bubbleSortOpt([5, 3, 8, 1, 9, 2]));
// [1, 2, 3, 5, 8, 9]
```

**Quand utiliser** : jamais en production — utile seulement pour apprendre. Bon pour des données quasi-triées (version optimisée ≈ O(n)).

---

## 9.3 Tri par sélection (Selection Sort)

```php
<?php
declare(strict_types=1);

// O(n²) — toujours, n²/2 comparaisons, n échanges
function selectionSort(array $arr): array
{
    $n = count($arr);
    for ($i = 0; $i < $n - 1; $i++) {
        $minIdx = $i;
        for ($j = $i + 1; $j < $n; $j++) {
            if ($arr[$j] < $arr[$minIdx]) {
                $minIdx = $j;
            }
        }
        if ($minIdx !== $i) {
            [$arr[$i], $arr[$minIdx]] = [$arr[$minIdx], $arr[$i]];
        }
    }
    return $arr;
}
```

**Quand utiliser** : quand les écritures sont coûteuses (mémoire flash) — effectue exactement n-1 échanges.

---

## 9.4 Tri par insertion (Insertion Sort)

```php
<?php
declare(strict_types=1);

// O(n²) pire cas, O(n) si quasi-trié
function insertionSort(array $arr): array
{
    $n = count($arr);
    for ($i = 1; $i < $n; $i++) {
        $cle = $arr[$i];
        $j   = $i - 1;

        // Décaler les éléments plus grands vers la droite
        while ($j >= 0 && $arr[$j] > $cle) {
            $arr[$j + 1] = $arr[$j];
            $j--;
        }
        $arr[$j + 1] = $cle;
    }
    return $arr;
}

// Version binaire — O(n log n) comparaisons mais O(n²) déplacements
function insertionSortBinaire(array $arr): array
{
    $n = count($arr);
    for ($i = 1; $i < $n; $i++) {
        $cle  = $arr[$i];
        // Trouver la position d'insertion par recherche binaire
        $pos = rechercherInsertion($arr, 0, $i - 1, $cle);
        // Décaler
        for ($j = $i - 1; $j >= $pos; $j--) {
            $arr[$j + 1] = $arr[$j];
        }
        $arr[$pos] = $cle;
    }
    return $arr;
}

function rechercherInsertion(array $arr, int $l, int $r, int $cle): int
{
    while ($l <= $r) {
        $m = intdiv($l + $r, 2);
        if ($arr[$m] === $cle) return $m + 1;
        if ($arr[$m] < $cle)   $l = $m + 1;
        else                   $r = $m - 1;
    }
    return $l;
}
```

**Quand utiliser** : excellent pour des petits tableaux (< 15 éléments) et des données quasi-triées. C'est pourquoi TimSort l'utilise pour les petites sous-séquences.

---

## 9.5 Tri fusion (Merge Sort)

```php
<?php
declare(strict_types=1);

// O(n log n) garanti — stable — mais O(n) mémoire
function mergeSort(array $arr): array
{
    $n = count($arr);
    if ($n <= 1) return $arr;

    $mid   = intdiv($n, 2);
    $left  = mergeSort(array_slice($arr, 0, $mid));
    $right = mergeSort(array_slice($arr, $mid));

    return fusion($left, $right);
}

function fusion(array $gauche, array $droite): array
{
    $result = [];
    $i = $j = 0;

    while ($i < count($gauche) && $j < count($droite)) {
        // <= pour la stabilité
        if ($gauche[$i] <= $droite[$j]) {
            $result[] = $gauche[$i++];
        } else {
            $result[] = $droite[$j++];
        }
    }

    // Ajouter les restes (un seul des deux while s'exécutera)
    while ($i < count($gauche)) $result[] = $gauche[$i++];
    while ($j < count($droite)) $result[] = $droite[$j++];

    return $result;
}

// Version bottom-up (itérative) — évite la récursion
function mergeSortIteratif(array $arr): array
{
    $n = count($arr);

    for ($taille = 1; $taille < $n; $taille *= 2) {
        for ($debut = 0; $debut < $n; $debut += 2 * $taille) {
            $milieu = min($debut + $taille - 1, $n - 1);
            $fin    = min($debut + 2 * $taille - 1, $n - 1);

            if ($milieu < $fin) {
                $gauche = array_slice($arr, $debut, $milieu - $debut + 1);
                $droite = array_slice($arr, $milieu + 1, $fin - $milieu);
                $merged = fusion($gauche, $droite);
                array_splice($arr, $debut, count($merged), $merged);
            }
        }
    }
    return $arr;
}

// ── Comptage d'inversions (variante merge sort) ──────────────
// Une inversion est une paire (i,j) telle que i<j mais arr[i]>arr[j]
function compterInversions(array $arr): int
{
    if (count($arr) <= 1) return 0;

    $mid     = intdiv(count($arr), 2);
    $gauche  = array_slice($arr, 0, $mid);
    $droite  = array_slice($arr, $mid);

    $count   = compterInversions($gauche) + compterInversions($droite);
    $i = $j  = 0;
    $merged  = [];

    while ($i < count($gauche) && $j < count($droite)) {
        if ($gauche[$i] <= $droite[$j]) {
            $merged[] = $gauche[$i++];
        } else {
            $count   += count($gauche) - $i;  // tous les restants dans gauche
            $merged[] = $droite[$j++];
        }
    }
    while ($i < count($gauche)) $merged[] = $gauche[$i++];
    while ($j < count($droite)) $merged[] = $droite[$j++];

    // Modifier $arr en place (simulation)
    for ($k = 0; $k < count($merged); $k++) $arr[$k] = $merged[$k];

    return $count;
}
```

---

## 9.6 Tri rapide (Quick Sort)

```php
<?php
declare(strict_types=1);

// O(n log n) moyen — O(n²) pire cas (tableau trié + pivot premier)
function quickSort(array $arr, int $bas = 0, int $haut = -1): array
{
    if ($haut === -1) $haut = count($arr) - 1;

    if ($bas < $haut) {
        $pivotIdx = partition($arr, $bas, $haut);
        quickSort($arr, $bas, $pivotIdx - 1);
        quickSort($arr, $pivotIdx + 1, $haut);
    }
    return $arr;
}

// Schéma de partition de Lomuto
function partition(array &$arr, int $bas, int $haut): int
{
    $pivot = $arr[$haut];  // dernier élément comme pivot
    $i     = $bas - 1;    // indice du plus petit élément

    for ($j = $bas; $j < $haut; $j++) {
        if ($arr[$j] <= $pivot) {
            $i++;
            [$arr[$i], $arr[$j]] = [$arr[$j], $arr[$i]];
        }
    }
    [$arr[$i + 1], $arr[$haut]] = [$arr[$haut], $arr[$i + 1]];
    return $i + 1;
}

// ── Pivot médian-de-trois (réduit le pire cas) ───────────────
function choisirPivot(array &$arr, int $bas, int $haut): void
{
    $mid = intdiv($bas + $haut, 2);

    // Trier les 3 candidats : bas, milieu, haut
    if ($arr[$bas] > $arr[$mid])  [$arr[$bas], $arr[$mid]]  = [$arr[$mid], $arr[$bas]];
    if ($arr[$bas] > $arr[$haut]) [$arr[$bas], $arr[$haut]] = [$arr[$haut], $arr[$bas]];
    if ($arr[$mid] > $arr[$haut]) [$arr[$mid], $arr[$haut]] = [$arr[$haut], $arr[$mid]];

    // La médiane est en $mid, la mettre en avant-dernière position
    [$arr[$mid], $arr[$haut - 1]] = [$arr[$haut - 1], $arr[$mid]];
}

// ── Quick Sort 3-way (Dutch National Flag) ───────────────────
// Excellent pour les tableaux avec beaucoup de doublons
function quickSort3Way(array &$arr, int $bas, int $haut): void
{
    if ($bas >= $haut) return;

    $pivot = $arr[$bas];
    $lt    = $bas;          // tout < pivot
    $gt    = $haut;         // tout > pivot
    $i     = $bas + 1;

    while ($i <= $gt) {
        if ($arr[$i] < $pivot) {
            [$arr[$lt++], $arr[$i++]] = [$arr[$i], $arr[$lt]];
        } elseif ($arr[$i] > $pivot) {
            [$arr[$i], $arr[$gt--]] = [$arr[$gt], $arr[$i]];
        } else {
            $i++;
        }
    }
    // arr[bas..lt-1] < pivot, arr[lt..gt] == pivot, arr[gt+1..haut] > pivot
    quickSort3Way($arr, $bas, $lt - 1);
    quickSort3Way($arr, $gt + 1, $haut);
}

// ── Kième plus petit (QuickSelect) ───────────────────────────
// O(n) moyen — plus rapide que trier pour trouver un seul élément
function quickSelect(array $arr, int $k): int
{
    $n = count($arr);
    if ($n === 1) return $arr[0];

    $pivotIdx = rand(0, $n - 1);
    $pivot    = $arr[$pivotIdx];

    $petits = array_filter($arr, fn($v) => $v < $pivot);
    $egaux  = array_filter($arr, fn($v) => $v === $pivot);
    $grands = array_filter($arr, fn($v) => $v > $pivot);

    $lp = count($petits);
    $le = count($egaux);

    if ($k <= $lp) {
        return quickSelect(array_values($petits), $k);
    } elseif ($k <= $lp + $le) {
        return $pivot;
    } else {
        return quickSelect(array_values($grands), $k - $lp - $le);
    }
}
```

---

## 9.7 Tri par comptage (Counting Sort)

```php
<?php
declare(strict_types=1);

// O(n + k) — k = intervalle des valeurs
// Idéal pour des entiers dans une plage connue et étroite
function countingSort(array $arr, int $max): array
{
    $compteur = array_fill(0, $max + 1, 0);

    // Compter les occurrences
    foreach ($arr as $v) $compteur[$v]++;

    // Cumuler (pour la version stable)
    for ($i = 1; $i <= $max; $i++) {
        $compteur[$i] += $compteur[$i - 1];
    }

    // Construire le tableau trié (de droite à gauche pour la stabilité)
    $resultat = array_fill(0, count($arr), 0);
    for ($i = count($arr) - 1; $i >= 0; $i--) {
        $resultat[--$compteur[$arr[$i]]] = $arr[$i];
    }

    return $resultat;
}

print_r(countingSort([4, 2, 2, 8, 3, 3, 1], 8));
// [1, 2, 2, 3, 3, 4, 8]
```

---

## 9.8 Tri par base (Radix Sort)

```php
<?php
declare(strict_types=1);

// O(d·n) — d = nombre de chiffres, n = nombre d'éléments
// Tri stable : trier chaque position de droite à gauche
function radixSort(array $arr): array
{
    $max = max($arr);

    // Trier par chaque position (unités, dizaines, centaines...)
    for ($exp = 1; intdiv($max, $exp) > 0; $exp *= 10) {
        $arr = countingSortParPosition($arr, $exp);
    }
    return $arr;
}

function countingSortParPosition(array $arr, int $exp): array
{
    $n        = count($arr);
    $resultat = array_fill(0, $n, 0);
    $compteur = array_fill(0, 10, 0);

    // Compter les occurrences du chiffre à la position $exp
    foreach ($arr as $v) {
        $chiffre = intdiv($v, $exp) % 10;
        $compteur[$chiffre]++;
    }

    // Positions cumulées
    for ($i = 1; $i < 10; $i++) {
        $compteur[$i] += $compteur[$i - 1];
    }

    // Construire de droite à gauche (stabilité)
    for ($i = $n - 1; $i >= 0; $i--) {
        $chiffre              = intdiv($arr[$i], $exp) % 10;
        $resultat[--$compteur[$chiffre]] = $arr[$i];
    }

    return $resultat;
}

print_r(radixSort([170, 45, 75, 90, 802, 24, 2, 66]));
// [2, 24, 45, 66, 75, 90, 170, 802]
```

---

## 9.9 Tri natif PHP

```php
<?php
declare(strict_types=1);

$arr = [3, 1, 4, 1, 5, 9, 2, 6];

// sort / rsort — trie ET réindexe — O(n log n) — TimSort
sort($arr);    // [1, 1, 2, 3, 4, 5, 6, 9]
rsort($arr);   // [9, 6, 5, 4, 3, 2, 1, 1]

// asort — préserve les associations clé/valeur
$scores = ['alice' => 85, 'bob' => 92, 'charlie' => 78];
asort($scores);   // charlie:78, alice:85, bob:92

// usort — tri personnalisé
$personnes = [
    ['nom' => 'Bob', 'age' => 30],
    ['nom' => 'Alice', 'age' => 25],
    ['nom' => 'Charlie', 'age' => 35],
];
usort($personnes, fn($a, $b) => $a['age'] <=> $b['age']);
// Alice(25), Bob(30), Charlie(35)

// uasort — personnalisé, préserve les clés
$data = ['b' => 3, 'a' => 1, 'c' => 2];
uasort($data, fn($a, $b) => $a <=> $b);

// uksort — tri par clé avec comparaison personnalisée
uksort($data, fn($a, $b) => strcmp($a, $b));
```

---

## 9.10 Quand utiliser quel algorithme ?

```
Données petites (n < 15)      → Insertion Sort
Données quasi-triées           → Insertion Sort
Garantie O(n log n)            → Merge Sort
In-place + rapide en pratique  → Quick Sort (pivot aléatoire)
Entiers, plage connue          → Counting Sort ou Radix Sort
Données quelconques            → sort() de PHP (TimSort)
Écriture minimale              → Selection Sort
```

---

## Exercices

!!! exercise "Exercice 9.1 — Trier les couleurs (Dutch National Flag)"
    Étant donné un tableau contenant 0, 1 et 2, triez-le en place en un seul passage. O(n) temps, O(1) espace.
    ```
    [2, 0, 2, 1, 1, 0]  →  [0, 0, 1, 1, 2, 2]
    ```
    ??? success "Réponse"
        ```php
        function sortirCouleurs(array &$arr): void
        {
            $bas = 0; $milieu = 0; $haut = count($arr) - 1;
            while ($milieu <= $haut) {
                match ($arr[$milieu]) {
                    0 => [[$arr[$bas], $arr[$milieu]] = [$arr[$milieu], $arr[$bas]], $bas++, $milieu++],
                    1 => [$milieu++],
                    2 => [[$arr[$milieu], $arr[$haut]] = [$arr[$haut], $arr[$milieu]], $haut--],
                };
            }
        }
        ```

!!! exercise "Exercice 9.2 — Trier une matrice en spirale"
    Triez une matrice n×n dans l'ordre spirale (de l'extérieur vers l'intérieur).

!!! exercise "Exercice 9.3 — Tri stable personnalisé"
    Implémentez un tri stable pour des objets ayant plusieurs critères (d'abord par département, puis par nom, puis par salaire décroissant).
