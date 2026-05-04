# Chapitre 10 — Algorithmes de recherche

## 10.1 Vue d'ensemble

| Algorithme | Pré-requis | Temps | Espace |
|-----------|-----------|-------|--------|
| Recherche linéaire | Aucun | O(n) | O(1) |
| Recherche binaire | Tableau trié | O(log n) | O(1) |
| Recherche ternaire | Tableau trié | O(log₃ n) | O(1) |
| Recherche par interpolation | Tableau trié uniforme | O(log log n) moy | O(1) |
| Recherche par saut (Jump) | Tableau trié | O(√n) | O(1) |
| Recherche exponentielle | Tableau trié | O(log n) | O(1) |

---

## 10.2 Recherche linéaire

```php
<?php
declare(strict_types=1);

// O(n) — applicable à tout tableau
function rechercheLineaire(array $arr, mixed $cible): int
{
    foreach ($arr as $i => $v) {
        if ($v === $cible) return $i;
    }
    return -1;
}

// Version avec prédicat — plus flexible
function rechercherSi(array $arr, callable $predicat): int
{
    foreach ($arr as $i => $v) {
        if ($predicat($v)) return $i;
    }
    return -1;
}

$idx = rechercherSi([1, 4, 7, 9, 12], fn($v) => $v > 8);  // 3
```

---

## 10.3 Recherche binaire

```php
<?php
declare(strict_types=1);

// Version itérative — O(log n), O(1) espace
function rechercheBinaire(array $arr, int $cible): int
{
    $gauche = 0;
    $droite = count($arr) - 1;

    while ($gauche <= $droite) {
        // Évite l'overflow : ($g + $d)/2 peut déborder en C/Java, pas en PHP
        $milieu = $gauche + intdiv($droite - $gauche, 2);

        if ($arr[$milieu] === $cible) return $milieu;
        if ($arr[$milieu] < $cible)  $gauche = $milieu + 1;
        else                         $droite  = $milieu - 1;
    }
    return -1;
}

// Version récursive — O(log n), O(log n) espace
function rechercheBinaireRec(array $arr, int $cible, int $g = 0, int $d = -1): int
{
    if ($d === -1) $d = count($arr) - 1;
    if ($g > $d)  return -1;

    $m = $g + intdiv($d - $g, 2);

    if ($arr[$m] === $cible) return $m;
    if ($arr[$m] < $cible)   return rechercheBinaireRec($arr, $cible, $m + 1, $d);
    return                          rechercheBinaireRec($arr, $cible, $g, $m - 1);
}
```

---

## 10.4 Variantes de la recherche binaire

Ces variantes sont **essentielles** en pratique (LeetCode, entretiens, problèmes réels).

```php
<?php
declare(strict_types=1);

// ── Première occurrence — lower_bound ───────────────────────
// Retourne l'index du premier élément >= cible
function premierOccurrence(array $arr, int $cible): int
{
    $g = 0;
    $d = count($arr);  // $d peut être hors du tableau

    while ($g < $d) {
        $m = $g + intdiv($d - $g, 2);
        if ($arr[$m] < $cible) {
            $g = $m + 1;
        } else {
            $d = $m;   // peut être la réponse, mais on continue à gauche
        }
    }
    return $g;  // $g === $d : position d'insertion ou premier >= cible
}

// ── Dernière occurrence — upper_bound - 1 ───────────────────
// Retourne l'index du dernier élément == cible, ou -1
function derniereOccurrence(array $arr, int $cible): int
{
    $g = 0;
    $d = count($arr);

    while ($g < $d) {
        $m = $g + intdiv($d - $g, 2);
        if ($arr[$m] <= $cible) {
            $g = $m + 1;
        } else {
            $d = $m;
        }
    }
    // $g-1 est l'index du dernier élément <= cible
    return ($g > 0 && $arr[$g - 1] === $cible) ? $g - 1 : -1;
}

// ── Plage d'occurrences [premier, dernier] ──────────────────
function plageOccurrences(array $arr, int $cible): array
{
    $premier  = premierOccurrence($arr, $cible);
    $dernier  = derniereOccurrence($arr, $cible);

    if ($premier === count($arr) || $arr[$premier] !== $cible) {
        return [-1, -1];
    }
    return [$premier, $dernier];
}

$arr = [2, 3, 3, 3, 5, 7];
print_r(plageOccurrences($arr, 3));  // [1, 3]

// ── Recherche dans un tableau rotatif ───────────────────────
// [4, 5, 6, 7, 0, 1, 2] — trié puis "rotaté"
function rechercheRotatif(array $arr, int $cible): int
{
    $g = 0;
    $d = count($arr) - 1;

    while ($g <= $d) {
        $m = $g + intdiv($d - $g, 2);

        if ($arr[$m] === $cible) return $m;

        // La moitié gauche est triée
        if ($arr[$g] <= $arr[$m]) {
            if ($arr[$g] <= $cible && $cible < $arr[$m]) {
                $d = $m - 1;  // cible dans la moitié gauche
            } else {
                $g = $m + 1;
            }
        } else {
            // La moitié droite est triée
            if ($arr[$m] < $cible && $cible <= $arr[$d]) {
                $g = $m + 1;  // cible dans la moitié droite
            } else {
                $d = $m - 1;
            }
        }
    }
    return -1;
}

echo rechercheRotatif([4, 5, 6, 7, 0, 1, 2], 0);  // 4
echo rechercheRotatif([4, 5, 6, 7, 0, 1, 2], 3);  // -1

// ── Rechercher dans une matrice triée 2D ────────────────────
// Chaque ligne triée, première val de chaque ligne > dernière val de la précédente
function rechercheMatrice(array $matrix, int $cible): bool
{
    $rows = count($matrix);
    $cols = count($matrix[0]);
    $g    = 0;
    $d    = $rows * $cols - 1;

    while ($g <= $d) {
        $m   = $g + intdiv($d - $g, 2);
        $val = $matrix[intdiv($m, $cols)][$m % $cols];

        if ($val === $cible) return true;
        if ($val < $cible)   $g = $m + 1;
        else                 $d = $m - 1;
    }
    return false;
}

// ── Trouver le minimum dans un tableau rotatif ──────────────
function minimumRotatif(array $arr): int
{
    $g = 0;
    $d = count($arr) - 1;

    while ($g < $d) {
        $m = $g + intdiv($d - $g, 2);

        if ($arr[$m] > $arr[$d]) {
            $g = $m + 1;  // minimum est dans la moitié droite
        } else {
            $d = $m;      // minimum est dans la moitié gauche (ou $m)
        }
    }
    return $arr[$g];
}

echo minimumRotatif([4, 5, 6, 7, 0, 1, 2]);  // 0
echo minimumRotatif([3, 4, 5, 1, 2]);         // 1

// ── Recherche de pic (Peak element) ─────────────────────────
// Un pic est un élément plus grand que ses voisins
function rechercherPic(array $arr): int
{
    $g = 0;
    $d = count($arr) - 1;

    while ($g < $d) {
        $m = $g + intdiv($d - $g, 2);

        if ($arr[$m] > $arr[$m + 1]) {
            $d = $m;      // pic dans la moitié gauche (ou à $m)
        } else {
            $g = $m + 1;  // pic dans la moitié droite
        }
    }
    return $g;  // index du pic
}
```

---

## 10.5 Recherche par interpolation

```php
<?php
declare(strict_types=1);

// O(log log n) si distribution uniforme
// Utilise une formule d'interpolation plutôt que le milieu fixe
function rechercheInterpolation(array $arr, int $cible): int
{
    $g = 0;
    $d = count($arr) - 1;

    while ($g <= $d && $cible >= $arr[$g] && $cible <= $arr[$d]) {
        if ($g === $d) {
            return $arr[$g] === $cible ? $g : -1;
        }

        // Position estimée par interpolation linéaire
        $pos = (int) ($g + (($d - $g) / ($arr[$d] - $arr[$g])) * ($cible - $arr[$g]));

        if ($arr[$pos] === $cible) return $pos;
        if ($arr[$pos] < $cible)  $g = $pos + 1;
        else                      $d = $pos - 1;
    }
    return -1;
}

$arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100];
echo rechercheInterpolation($arr, 70);  // 6
```

---

## 10.6 Recherche par saut (Jump Search)

```php
<?php
declare(strict_types=1);

// O(√n) — compromis entre linéaire et binaire, bon pour les E/S disque
function rechercheParSaut(array $arr, int $cible): int
{
    $n    = count($arr);
    $saut = (int) sqrt($n);
    $prev = 0;

    // Sauter par blocs
    while ($arr[min($saut, $n) - 1] < $cible) {
        $prev = $saut;
        $saut += (int) sqrt($n);
        if ($prev >= $n) return -1;
    }

    // Recherche linéaire dans le bloc trouvé
    while ($arr[$prev] < $cible) {
        $prev++;
        if ($prev === min($saut, $n)) return -1;
    }

    return $arr[$prev] === $cible ? $prev : -1;
}
```

---

## 10.7 Recherche exponentielle

```php
<?php
declare(strict_types=1);

// O(log n) — utile pour les tableaux infinis ou de taille inconnue
function rechercheExponentielle(array $arr, int $cible): int
{
    $n = count($arr);

    if ($arr[0] === $cible) return 0;

    // Trouver la plage : doubler l'index jusqu'à trouver > cible
    $i = 1;
    while ($i < $n && $arr[$i] <= $cible) {
        $i *= 2;
    }

    // Recherche binaire dans la plage [i/2, min(i, n-1)]
    return rechercheBinairePartielle($arr, $cible, intdiv($i, 2), min($i, $n - 1));
}

function rechercheBinairePartielle(array $arr, int $cible, int $g, int $d): int
{
    while ($g <= $d) {
        $m = $g + intdiv($d - $g, 2);
        if ($arr[$m] === $cible) return $m;
        if ($arr[$m] < $cible)  $g = $m + 1;
        else                    $d = $m - 1;
    }
    return -1;
}
```

---

## 10.8 Recherche dans des structures non-triées

### Recherche dans un BST

```php
<?php
declare(strict_types=1);

// Voir chapitre 6 — O(log n) moyen
function chercherBST(?NoeudArbre $racine, int $cible): ?NoeudArbre
{
    $courant = $racine;
    while ($courant !== null) {
        if ($cible === $courant->valeur) return $courant;
        $courant = $cible < $courant->valeur ? $courant->gauche : $courant->droite;
    }
    return null;
}
```

### Recherche de pattern dans une chaîne (KMP)

```php
<?php
declare(strict_types=1);

/**
 * Algorithme Knuth-Morris-Pratt — O(n + m)
 * Recherche toutes les occurrences d'un motif dans un texte.
 */
function kmpRechercher(string $texte, string $motif): array
{
    $n = strlen($texte);
    $m = strlen($motif);

    if ($m === 0) return [];

    $lps        = calculerLPS($motif);   // Longest Proper Prefix-Suffix
    $occurrences = [];
    $i = $j     = 0;

    while ($i < $n) {
        if ($texte[$i] === $motif[$j]) {
            $i++;
            $j++;
        }

        if ($j === $m) {
            $occurrences[] = $i - $j;  // occurrence trouvée à cet index
            $j = $lps[$j - 1];
        } elseif ($i < $n && $texte[$i] !== $motif[$j]) {
            if ($j !== 0) {
                $j = $lps[$j - 1];   // utiliser le suffixe déjà matché
            } else {
                $i++;
            }
        }
    }
    return $occurrences;
}

/** Calcule la table LPS — O(m) */
function calculerLPS(string $motif): array
{
    $m   = strlen($motif);
    $lps = array_fill(0, $m, 0);
    $len = 0;
    $i   = 1;

    while ($i < $m) {
        if ($motif[$i] === $motif[$len]) {
            $lps[$i++] = ++$len;
        } elseif ($len !== 0) {
            $len = $lps[$len - 1];  // reculer sans déplacer $i
        } else {
            $lps[$i++] = 0;
        }
    }
    return $lps;
}

print_r(kmpRechercher("AABAACAADAABAABA", "AABA"));
// [0, 9, 12]
```

### Recherche en BFS dans une grille

```php
<?php
declare(strict_types=1);

/**
 * BFS sur grille — plus court chemin entre deux cases.
 * O(n·m) pour une grille n×m.
 */
function bfsGrille(array $grille, array $depart, array $arrivee): int
{
    $rows = count($grille);
    $cols = count($grille[0]);
    $dirs = [[0,1],[0,-1],[1,0],[-1,0]];

    $visite = array_fill(0, $rows, array_fill(0, $cols, false));
    $queue  = new SplQueue();

    $queue->enqueue([$depart[0], $depart[1], 0]);
    $visite[$depart[0]][$depart[1]] = true;

    while (!$queue->isEmpty()) {
        [$r, $c, $dist] = $queue->dequeue();

        if ($r === $arrivee[0] && $c === $arrivee[1]) return $dist;

        foreach ($dirs as [$dr, $dc]) {
            $nr = $r + $dr;
            $nc = $c + $dc;

            if ($nr >= 0 && $nr < $rows && $nc >= 0 && $nc < $cols
                && !$visite[$nr][$nc] && $grille[$nr][$nc] === 0) {
                $visite[$nr][$nc] = true;
                $queue->enqueue([$nr, $nc, $dist + 1]);
            }
        }
    }
    return -1;  // pas de chemin
}

$grille = [
    [0, 0, 0],
    [1, 1, 0],
    [0, 0, 0],
];
echo bfsGrille($grille, [0,0], [2,2]);  // 4
```

---

## Résumé visuel

```
Tableau non trié      → Recherche linéaire O(n)
Tableau trié          → Recherche binaire  O(log n)
Distribution uniforme → Interpolation      O(log log n)
Taille inconnue       → Exponentielle      O(log n)
Accès disque lent     → Jump Search        O(√n)
Pattern dans texte    → KMP                O(n+m)
Graphe/Grille         → BFS                O(V+E)
```

---

## Exercices

!!! exercise "Exercice 10.1 — Minimum absolu d'une fonction unimodale"
    Utilisez la recherche ternaire pour trouver le minimum d'une fonction unimodale f(x) sur un intervalle réel [a, b].

!!! exercise "Exercice 10.2 — Recherche binaire sur la réponse"
    Un ouvrier peut peindre des clôtures. Il y a n maisons et m ouvriers. Chaque ouvrier peint des maisons consécutives. Trouvez le temps minimum pour finir en utilisant la recherche binaire sur la réponse.

!!! exercise "Exercice 10.3 — Algorithme de Boyer-Moore-Horspool"
    Implémentez l'algorithme de recherche de motif Boyer-Moore-Horspool. Comparez ses performances avec KMP sur des textes longs.
