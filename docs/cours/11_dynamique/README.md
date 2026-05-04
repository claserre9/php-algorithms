# Chapitre 11 — Programmation dynamique

## 11.1 Concept

La **programmation dynamique** (PD) est une technique d'optimisation pour les problèmes qui exhibent deux propriétés :

1. **Sous-problèmes chevauchants** — le problème se décompose en sous-problèmes identiques calculés plusieurs fois
2. **Sous-structure optimale** — la solution optimale d'un problème contient les solutions optimales de ses sous-problèmes

```
Fibonacci naïf — arbre de récursion :

         fib(5)
        /      \
    fib(4)    fib(3)
    /    \    /    \
 fib(3) fib(2) fib(2) fib(1)
 ...

fib(3) calculé 2 fois, fib(2) calculé 3 fois... → O(2^n)
Avec mémoïsation → O(n) !
```

### Deux approches

| | Mémoïsation (top-down) | Tabulation (bottom-up) |
|-|----------------------|----------------------|
| Direction | Problème → sous-problèmes | Sous-problèmes → problème |
| Récursion | Oui + cache | Non (boucles) |
| Espace | O(n) + pile d'appel | O(n) ou O(1) optimisé |
| Calculs | Seulement les nécessaires | Tous les sous-problèmes |

---

## 11.2 Fibonacci — les 4 niveaux

```php
<?php
declare(strict_types=1);

// ── Niveau 0 : Naïf — O(2^n) ────────────────────────────────
function fibNaif(int $n): int
{
    if ($n <= 1) return $n;
    return fibNaif($n - 1) + fibNaif($n - 2);
}

// ── Niveau 1 : Mémoïsation top-down — O(n) ──────────────────
function fibMemo(int $n, array &$memo = []): int
{
    if ($n <= 1)           return $n;
    if (isset($memo[$n]))  return $memo[$n];

    return $memo[$n] = fibMemo($n - 1, $memo) + fibMemo($n - 2, $memo);
}

// ── Niveau 2 : Tabulation bottom-up — O(n) temps, O(n) espace
function fibTable(int $n): int
{
    if ($n <= 1) return $n;

    $dp    = array_fill(0, $n + 1, 0);
    $dp[1] = 1;

    for ($i = 2; $i <= $n; $i++) {
        $dp[$i] = $dp[$i - 1] + $dp[$i - 2];
    }
    return $dp[$n];
}

// ── Niveau 3 : Optimisé — O(n) temps, O(1) espace ───────────
function fibOpt(int $n): int
{
    if ($n <= 1) return $n;

    $prev2 = 0;
    $prev1 = 1;

    for ($i = 2; $i <= $n; $i++) {
        $curr  = $prev1 + $prev2;
        $prev2 = $prev1;
        $prev1 = $curr;
    }
    return $prev1;
}
```

---

## 11.3 Problème du sac à dos (0/1 Knapsack)

Étant donnés n objets de poids et valeurs différents, et un sac de capacité W, maximisez la valeur totale sans dépasser W.

```php
<?php
declare(strict_types=1);

// Tabulation — O(n·W) temps, O(n·W) espace
function knapsack(array $poids, array $valeurs, int $capacite): int
{
    $n  = count($poids);
    $dp = array_fill(0, $n + 1, array_fill(0, $capacite + 1, 0));

    for ($i = 1; $i <= $n; $i++) {
        for ($w = 0; $w <= $capacite; $w++) {
            // Ne pas prendre l'objet i
            $dp[$i][$w] = $dp[$i - 1][$w];

            // Prendre l'objet i (si ça rentre)
            if ($poids[$i - 1] <= $w) {
                $avecObjet  = $valeurs[$i - 1] + $dp[$i - 1][$w - $poids[$i - 1]];
                $dp[$i][$w] = max($dp[$i][$w], $avecObjet);
            }
        }
    }
    return $dp[$n][$capacite];
}

// Version espace O(W) — une seule ligne
function knapsackOpt(array $poids, array $valeurs, int $capacite): int
{
    $n  = count($poids);
    $dp = array_fill(0, $capacite + 1, 0);

    for ($i = 0; $i < $n; $i++) {
        // Parcourir de droite à gauche pour éviter de réutiliser l'objet
        for ($w = $capacite; $w >= $poids[$i]; $w--) {
            $dp[$w] = max($dp[$w], $valeurs[$i] + $dp[$w - $poids[$i]]);
        }
    }
    return $dp[$capacite];
}

// Reconstruction de la solution
function knapsackSolution(array $poids, array $valeurs, int $capacite): array
{
    $n  = count($poids);
    $dp = array_fill(0, $n + 1, array_fill(0, $capacite + 1, 0));

    for ($i = 1; $i <= $n; $i++) {
        for ($w = 0; $w <= $capacite; $w++) {
            $dp[$i][$w] = $dp[$i - 1][$w];
            if ($poids[$i-1] <= $w) {
                $dp[$i][$w] = max($dp[$i][$w], $valeurs[$i-1] + $dp[$i-1][$w-$poids[$i-1]]);
            }
        }
    }

    // Remonter le chemin
    $objets = [];
    $w = $capacite;
    for ($i = $n; $i > 0; $i--) {
        if ($dp[$i][$w] !== $dp[$i - 1][$w]) {
            $objets[] = $i - 1;
            $w -= $poids[$i - 1];
        }
    }
    return ['valeur' => $dp[$n][$capacite], 'objets' => array_reverse($objets)];
}

$poids   = [2, 3, 4, 5];
$valeurs = [3, 4, 5, 6];
echo knapsack($poids, $valeurs, 5);     // 7 (objets 0+1 : poids=5, valeur=7)
echo knapsackOpt($poids, $valeurs, 5);  // 7
```

---

## 11.4 Plus longue sous-séquence commune (LCS)

```php
<?php
declare(strict_types=1);

// LCS — Longest Common Subsequence — O(m·n)
function lcs(string $a, string $b): int
{
    $m  = strlen($a);
    $n  = strlen($b);
    $dp = array_fill(0, $m + 1, array_fill(0, $n + 1, 0));

    for ($i = 1; $i <= $m; $i++) {
        for ($j = 1; $j <= $n; $j++) {
            if ($a[$i - 1] === $b[$j - 1]) {
                $dp[$i][$j] = 1 + $dp[$i - 1][$j - 1];
            } else {
                $dp[$i][$j] = max($dp[$i - 1][$j], $dp[$i][$j - 1]);
            }
        }
    }
    return $dp[$m][$n];
}

// Reconstruire la séquence
function lcsSequence(string $a, string $b): string
{
    $m  = strlen($a);
    $n  = strlen($b);
    $dp = array_fill(0, $m + 1, array_fill(0, $n + 1, 0));

    for ($i = 1; $i <= $m; $i++) {
        for ($j = 1; $j <= $n; $j++) {
            if ($a[$i - 1] === $b[$j - 1]) {
                $dp[$i][$j] = 1 + $dp[$i - 1][$j - 1];
            } else {
                $dp[$i][$j] = max($dp[$i - 1][$j], $dp[$i][$j - 1]);
            }
        }
    }

    // Remonter
    $resultat = '';
    $i = $m; $j = $n;
    while ($i > 0 && $j > 0) {
        if ($a[$i-1] === $b[$j-1]) {
            $resultat = $a[$i-1] . $resultat;
            $i--; $j--;
        } elseif ($dp[$i-1][$j] > $dp[$i][$j-1]) {
            $i--;
        } else {
            $j--;
        }
    }
    return $resultat;
}

echo lcs("ABCBDAB", "BDCAB");         // 4
echo lcsSequence("ABCBDAB", "BDCAB"); // BCAB

// Distance d'édition (Levenshtein) — O(m·n)
function distanceEdition(string $a, string $b): int
{
    $m  = strlen($a);
    $n  = strlen($b);
    $dp = array_fill(0, $m + 1, array_fill(0, $n + 1, 0));

    for ($i = 0; $i <= $m; $i++) $dp[$i][0] = $i;
    for ($j = 0; $j <= $n; $j++) $dp[0][$j] = $j;

    for ($i = 1; $i <= $m; $i++) {
        for ($j = 1; $j <= $n; $j++) {
            if ($a[$i-1] === $b[$j-1]) {
                $dp[$i][$j] = $dp[$i-1][$j-1];
            } else {
                $dp[$i][$j] = 1 + min(
                    $dp[$i-1][$j],    // suppression
                    $dp[$i][$j-1],    // insertion
                    $dp[$i-1][$j-1]   // substitution
                );
            }
        }
    }
    return $dp[$m][$n];
}

echo distanceEdition("kitten", "sitting");  // 3
```

---

## 11.5 Plus longue sous-séquence croissante (LIS)

```php
<?php
declare(strict_types=1);

// LIS O(n²) — version simple
function lisSimple(array $arr): int
{
    $n  = count($arr);
    $dp = array_fill(0, $n, 1);

    for ($i = 1; $i < $n; $i++) {
        for ($j = 0; $j < $i; $j++) {
            if ($arr[$j] < $arr[$i]) {
                $dp[$i] = max($dp[$i], $dp[$j] + 1);
            }
        }
    }
    return max($dp);
}

// LIS O(n log n) — avec patience sorting
function lis(array $arr): int
{
    $tails = [];

    foreach ($arr as $v) {
        // Recherche binaire : première queue >= v
        $g = 0;
        $d = count($tails);

        while ($g < $d) {
            $m = intdiv($g + $d, 2);
            if ($tails[$m] < $v) $g = $m + 1;
            else                 $d = $m;
        }

        $tails[$g] = $v;
        // Si $g === count($tails)-1, on a étendu la LIS
    }
    return count($tails);
}

echo lisSimple([10, 9, 2, 5, 3, 7, 101, 18]);  // 4 ([2,3,7,101] ou [2,5,7,101])
echo lis([10, 9, 2, 5, 3, 7, 101, 18]);         // 4
```

---

## 11.6 Problème du rendu de monnaie

```php
<?php
declare(strict_types=1);

// Nombre minimum de pièces — O(montant · nombre_pièces)
function renduMonnaie(array $pieces, int $montant): int
{
    $dp     = array_fill(0, $montant + 1, PHP_INT_MAX);
    $dp[0]  = 0;

    for ($i = 1; $i <= $montant; $i++) {
        foreach ($pieces as $p) {
            if ($p <= $i && $dp[$i - $p] !== PHP_INT_MAX) {
                $dp[$i] = min($dp[$i], 1 + $dp[$i - $p]);
            }
        }
    }
    return $dp[$montant] === PHP_INT_MAX ? -1 : $dp[$montant];
}

// Nombre de façons de rendre la monnaie
function nombreFaconsMonnaie(array $pieces, int $montant): int
{
    $dp    = array_fill(0, $montant + 1, 0);
    $dp[0] = 1;

    foreach ($pieces as $p) {                // coin change II (ordre stable)
        for ($i = $p; $i <= $montant; $i++) {
            $dp[$i] += $dp[$i - $p];
        }
    }
    return $dp[$montant];
}

echo renduMonnaie([1, 5, 10, 25], 36);       // 3 (25+10+1)
echo nombreFaconsMonnaie([1, 2, 5], 5);      // 4 (5, 2+2+1, 2+1+1+1, 1+1+1+1+1)
```

---

## 11.7 Chemin dans une grille

```php
<?php
declare(strict_types=1);

// Nombre de chemins uniques dans une grille m×n (mouvements droite/bas)
function nombreChemins(int $m, int $n): int
{
    $dp = array_fill(0, $m, array_fill(0, $n, 1));

    for ($i = 1; $i < $m; $i++) {
        for ($j = 1; $j < $n; $j++) {
            $dp[$i][$j] = $dp[$i-1][$j] + $dp[$i][$j-1];
        }
    }
    return $dp[$m-1][$n-1];
}

// Somme minimale d'un chemin — O(m·n)
function cheminSommeMin(array $grille): int
{
    $m  = count($grille);
    $n  = count($grille[0]);
    $dp = $grille;

    // Remplir la première ligne et colonne
    for ($j = 1; $j < $n; $j++) $dp[0][$j] += $dp[0][$j-1];
    for ($i = 1; $i < $m; $i++) $dp[$i][0] += $dp[$i-1][0];

    for ($i = 1; $i < $m; $i++) {
        for ($j = 1; $j < $n; $j++) {
            $dp[$i][$j] += min($dp[$i-1][$j], $dp[$i][$j-1]);
        }
    }
    return $dp[$m-1][$n-1];
}

echo nombreChemins(3, 7);                         // 28
echo cheminSommeMin([[1,3,1],[1,5,1],[4,2,1]]);   // 7 (1→3→1→1→1)
```

---

## 11.8 Multiplication de chaîne de matrices

```php
<?php
declare(strict_types=1);

/**
 * Trouve le nombre minimum de multiplications scalaires
 * pour multiplier une chaîne de matrices.
 * O(n³)
 */
function chainMultiplication(array $dims): int
{
    $n  = count($dims) - 1;  // nombre de matrices
    $dp = array_fill(0, $n, array_fill(0, $n, 0));

    // l = longueur de la sous-chaîne
    for ($l = 2; $l <= $n; $l++) {
        for ($i = 0; $i <= $n - $l; $i++) {
            $j       = $i + $l - 1;
            $dp[$i][$j] = PHP_INT_MAX;

            for ($k = $i; $k < $j; $k++) {
                $cout       = $dp[$i][$k] + $dp[$k+1][$j] + $dims[$i] * $dims[$k+1] * $dims[$j+1];
                $dp[$i][$j] = min($dp[$i][$j], $cout);
            }
        }
    }
    return $dp[0][$n - 1];
}

// Matrices : A(10×30), B(30×5), C(5×60)
echo chainMultiplication([10, 30, 5, 60]);  // 27000
```

---

## 11.9 Problèmes classiques supplémentaires

```php
<?php
declare(strict_types=1);

// ── Maximum sous-tableau (Kadane) — O(n) ─────────────────────
function kadane(array $arr): int
{
    $maxActuel = $arr[0];
    $maxGlobal = $arr[0];

    for ($i = 1; $i < count($arr); $i++) {
        $maxActuel = max($arr[$i], $maxActuel + $arr[$i]);
        $maxGlobal = max($maxGlobal, $maxActuel);
    }
    return $maxGlobal;
}

echo kadane([-2, 1, -3, 4, -1, 2, 1, -5, 4]);  // 6 ([4,-1,2,1])

// ── Plus long palindrome dans une chaîne ─────────────────────
// Expand Around Center — O(n²) temps, O(1) espace
function plusLongPalindrome(string $s): string
{
    $n     = strlen($s);
    $debut = 0;
    $max   = 1;

    for ($i = 0; $i < $n; $i++) {
        // Palindromes de longueur impaire
        [$d, $len] = expanderPalindrome($s, $i, $i);
        if ($len > $max) { $debut = $d; $max = $len; }

        // Palindromes de longueur paire
        [$d, $len] = expanderPalindrome($s, $i, $i + 1);
        if ($len > $max) { $debut = $d; $max = $len; }
    }
    return substr($s, $debut, $max);
}

function expanderPalindrome(string $s, int $g, int $d): array
{
    $n = strlen($s);
    while ($g >= 0 && $d < $n && $s[$g] === $s[$d]) {
        $g--; $d++;
    }
    $longueur = $d - $g - 1;
    return [$g + 1, $longueur];
}

echo plusLongPalindrome("babad");    // "bab" ou "aba"
echo plusLongPalindrome("cbbd");     // "bb"

// ── Découpage de mot (Word Break) ────────────────────────────
function decoupage(string $s, array $dictionnaire): bool
{
    $n   = strlen($s);
    $dp  = array_fill(0, $n + 1, false);
    $dp[0] = true;  // chaîne vide = valide

    $dico = array_flip($dictionnaire);

    for ($i = 1; $i <= $n; $i++) {
        for ($j = 0; $j < $i; $j++) {
            if ($dp[$j] && isset($dico[substr($s, $j, $i - $j)])) {
                $dp[$i] = true;
                break;
            }
        }
    }
    return $dp[$n];
}

var_dump(decoupage("leetcode", ["leet", "code"]));        // true
var_dump(decoupage("applepenapple", ["apple", "pen"]));  // true
var_dump(decoupage("catsandog", ["cats", "dog", "sand", "and", "cat"])); // false
```

---

## 11.10 Patron de reconnaissance

!!! tip "Comment reconnaître un problème de PD ?"
    1. Demande un **maximum**, **minimum**, **nombre de façons**, **vrai/faux**
    2. On ne peut pas annuler un choix (greedy ne suffit pas)
    3. Le problème peut être décomposé en sous-problèmes **chevauchants**
    4. Il existe une **récurrence** : `dp[i] = f(dp[j], j < i)`

!!! info "Template PD"
    ```php
    // 1. Définir la signification de dp[i] (ou dp[i][j])
    // 2. Identifier les cas de base
    // 3. Écrire la relation de récurrence
    // 4. Ordre de calcul (bottom-up) ou mémoïsation (top-down)
    // 5. Retourner dp[n] (ou max(dp), etc.)
    ```

---

## Exercices

!!! exercise "Exercice 11.1 — Escalier"
    Vous montez un escalier de n marches. Vous pouvez monter 1 ou 2 marches à la fois. Combien de façons différentes pour atteindre le sommet ?
    ??? success "Réponse"
        Exactement comme Fibonacci : `dp[i] = dp[i-1] + dp[i-2]`

!!! exercise "Exercice 11.2 — Sac à dos non borné"
    Comme le 0/1 Knapsack, mais chaque objet peut être pris un nombre illimité de fois. Quelle est la modification dans la tabulation ?

!!! exercise "Exercice 11.3 — Découpage de tige"
    Une tige de longueur n peut être coupée et vendue. Étant donnés les prix pour chaque longueur, maximisez le revenu. O(n²).

!!! exercise "Exercice 11.4 — Interleaving Strings"
    Vérifiez si une chaîne c peut être formée en intercalant les caractères de a et b dans l'ordre.
