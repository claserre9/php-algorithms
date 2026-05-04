# Chapitre 2 — Tableaux & listes dynamiques

## 2.1 Le tableau en PHP

En PHP, un `array` est une **structure hybride** qui implémente à la fois :

- Un **tableau indexé** (liste ordonnée à accès O(1))
- Un **tableau associatif** (map clé→valeur, hash table interne)
- Une **file** ou **pile** via `array_push`, `array_pop`, etc.

```php
<?php
declare(strict_types=1);

// Tableau indexé (liste)
$nombres = [10, 20, 30, 40, 50];

// Accès O(1)
echo $nombres[2];          // 30

// Tableau associatif (map)
$utilisateur = [
    'nom'  => 'Alice',
    'age'  => 30,
    'ville' => 'Paris',
];

echo $utilisateur['nom'];  // Alice
```

---

## 2.2 Opérations fondamentales et complexités

| Opération | PHP | Complexité |
|-----------|-----|-----------|
| Accès par index | `$arr[i]` | O(1) |
| Modification | `$arr[i] = v` | O(1) |
| Ajout en fin | `$arr[] = v` ou `array_push` | O(1) amorti |
| Suppression en fin | `array_pop` | O(1) |
| Ajout en début | `array_unshift` | O(n) |
| Suppression en début | `array_shift` | O(n) |
| Recherche (valeur) | `in_array` | O(n) |
| Recherche (clé) | `isset($arr[$k])` | O(1) |
| Longueur | `count` | O(1) |
| Copie | `$b = $a` | O(n) |

!!! warning "array_unshift / array_shift sont O(n)"
    Ces fonctions réindexent tout le tableau. Préférez une `SplDoublyLinkedList` si vous avez besoin d'insertions fréquentes en tête.

---

## 2.3 Implémentation d'une ArrayList

Une `ArrayList` est un tableau redimensionnable avec gestion manuelle de la capacité — analogue à `ArrayList` en Java ou `std::vector` en C++.

```php
<?php
declare(strict_types=1);

/**
 * Liste dynamique générique basée sur un tableau PHP natif.
 * Capacité doublée automatiquement à chaque débordement.
 *
 * @template T
 */
class ArrayList
{
    /** @var array<int, mixed> */
    private array $data     = [];
    private int   $size     = 0;
    private int   $capacity;

    public function __construct(int $initialCapacity = 4)
    {
        $this->capacity = max(1, $initialCapacity);
    }

    // ── Taille ──────────────────────────────────────────────────

    public function size(): int   { return $this->size; }
    public function isEmpty(): bool { return $this->size === 0; }

    // ── Accès ───────────────────────────────────────────────────

    /** @return mixed */
    public function get(int $index): mixed
    {
        $this->checkIndex($index);
        return $this->data[$index];
    }

    /** @param mixed $value */
    public function set(int $index, mixed $value): void
    {
        $this->checkIndex($index);
        $this->data[$index] = $value;
    }

    // ── Ajout ───────────────────────────────────────────────────

    /** Ajoute en fin — O(1) amorti */
    public function add(mixed $value): void
    {
        if ($this->size === $this->capacity) {
            $this->resize();
        }
        $this->data[$this->size++] = $value;
    }

    /** Insère à l'index i — O(n) */
    public function insertAt(int $index, mixed $value): void
    {
        if ($index < 0 || $index > $this->size) {
            throw new OutOfRangeException("Index $index hors limites");
        }
        if ($this->size === $this->capacity) {
            $this->resize();
        }
        // Décaler les éléments vers la droite
        for ($i = $this->size; $i > $index; $i--) {
            $this->data[$i] = $this->data[$i - 1];
        }
        $this->data[$index] = $value;
        $this->size++;
    }

    // ── Suppression ─────────────────────────────────────────────

    /** Supprime et retourne le dernier élément — O(1) */
    public function removeLast(): mixed
    {
        if ($this->isEmpty()) {
            throw new UnderflowException("Liste vide");
        }
        $value = $this->data[--$this->size];
        unset($this->data[$this->size]);
        return $value;
    }

    /** Supprime l'élément à l'index i — O(n) */
    public function removeAt(int $index): mixed
    {
        $this->checkIndex($index);
        $value = $this->data[$index];
        // Décaler les éléments vers la gauche
        for ($i = $index; $i < $this->size - 1; $i++) {
            $this->data[$i] = $this->data[$i + 1];
        }
        unset($this->data[--$this->size]);
        return $value;
    }

    // ── Recherche ───────────────────────────────────────────────

    /** Recherche linéaire — O(n) */
    public function indexOf(mixed $value): int
    {
        for ($i = 0; $i < $this->size; $i++) {
            if ($this->data[$i] === $value) return $i;
        }
        return -1;
    }

    public function contains(mixed $value): bool
    {
        return $this->indexOf($value) !== -1;
    }

    // ── Utilitaires ─────────────────────────────────────────────

    public function toArray(): array
    {
        return array_slice($this->data, 0, $this->size);
    }

    public function __toString(): string
    {
        return '[' . implode(', ', $this->toArray()) . ']';
    }

    // ── Interne ─────────────────────────────────────────────────

    /** Doublement de capacité — O(n) mais O(1) amorti sur add() */
    private function resize(): void
    {
        $this->capacity *= 2;
        // En PHP, les tableaux sont déjà dynamiques,
        // mais on simule le comportement pour l'apprentissage
    }

    private function checkIndex(int $index): void
    {
        if ($index < 0 || $index >= $this->size) {
            throw new OutOfRangeException("Index $index hors limites [0, {$this->size})");
        }
    }
}
```

---

## 2.4 Démonstration

```php
<?php
declare(strict_types=1);

$list = new ArrayList();

// Ajout en fin — O(1)
$list->add(10);
$list->add(20);
$list->add(30);
$list->add(40);
echo $list;          // [10, 20, 30, 40]

// Insertion au milieu — O(n)
$list->insertAt(2, 25);
echo $list;          // [10, 20, 25, 30, 40]

// Accès et modification — O(1)
echo $list->get(1);  // 20
$list->set(1, 99);
echo $list;          // [10, 99, 25, 30, 40]

// Suppression — O(n)
$removed = $list->removeAt(1);
echo "Retiré : $removed\n";  // Retiré : 99
echo $list;                   // [10, 25, 30, 40]

// Recherche — O(n)
echo $list->indexOf(30);  // 2
echo $list->contains(99) ? 'oui' : 'non';  // non
```

---

## 2.5 Manipulation de tableaux PHP natifs

PHP fournit une bibliothèque riche de fonctions. Voici les plus utiles avec leur complexité :

```php
<?php
declare(strict_types=1);

$arr = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3];

// ── Tri ─────────────────────────────────────────────────────
sort($arr);              // O(n log n) — tri croissant, réindexe
rsort($arr);             // O(n log n) — tri décroissant
asort($arr);             // O(n log n) — préserve les clés
ksort($arr);             // O(n log n) — tri par clé
usort($arr, fn($a, $b) => $a <=> $b);  // tri personnalisé

// ── Recherche ───────────────────────────────────────────────
$pos = array_search(5, $arr);    // O(n) — retourne la clé ou false
$ok  = in_array(9, $arr);        // O(n)
$ok  = isset($arr[2]);           // O(1) — vérification par index

// ── Transformation ──────────────────────────────────────────
$doubled  = array_map(fn($v) => $v * 2, $arr);          // O(n)
$evens    = array_filter($arr, fn($v) => $v % 2 === 0); // O(n)
$sum      = array_reduce($arr, fn($c, $v) => $c + $v, 0); // O(n)

// ── Découpage / fusion ──────────────────────────────────────
$slice    = array_slice($arr, 1, 3);       // O(k) où k = longueur
$merged   = array_merge($arr, [7, 8]);     // O(n + m)
$unique   = array_unique($arr);            // O(n log n)
$flipped  = array_flip($arr);             // O(n) — échange clé/valeur
$reversed = array_reverse($arr);           // O(n)

// ── Pile / File ─────────────────────────────────────────────
array_push($arr, 99);     // O(1) — ajoute en fin
$top = array_pop($arr);   // O(1) — retire en fin
array_unshift($arr, 0);   // O(n) — ajoute en début
$front = array_shift($arr); // O(n) — retire en début

// ── Set operations ──────────────────────────────────────────
$a = [1, 2, 3, 4];
$b = [3, 4, 5, 6];
$inter = array_intersect($a, $b);     // [3, 4] — O(n·m)
$diff  = array_diff($a, $b);          // [1, 2] — O(n·m)
$union = array_unique(array_merge($a, $b)); // [1..6] — O(n+m)
```

---

## 2.6 Tableaux multidimensionnels

```php
<?php
declare(strict_types=1);

// Matrice 3×3
$matrice = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
];

// Accès O(1)
echo $matrice[1][2]; // 6

// Transposée O(n²)
function transpose(array $matrix): array
{
    $rows = count($matrix);
    $cols = count($matrix[0]);
    $result = [];

    for ($j = 0; $j < $cols; $j++) {
        for ($i = 0; $i < $rows; $i++) {
            $result[$j][$i] = $matrix[$i][$j];
        }
    }
    return $result;
}

// Multiplication matricielle O(n³)
function matMul(array $A, array $B): array
{
    $n = count($A);
    $result = array_fill(0, $n, array_fill(0, $n, 0));

    for ($i = 0; $i < $n; $i++) {
        for ($j = 0; $j < $n; $j++) {
            for ($k = 0; $k < $n; $k++) {
                $result[$i][$j] += $A[$i][$k] * $B[$k][$j];
            }
        }
    }
    return $result;
}

// Recherche dans une matrice triée O(n + m)
// (chaque ligne et colonne triée croissante)
function searchMatrix(array $matrix, int $target): bool
{
    $row = 0;
    $col = count($matrix[0]) - 1;

    while ($row < count($matrix) && $col >= 0) {
        if ($matrix[$row][$col] === $target) return true;
        if ($matrix[$row][$col] > $target)   $col--;
        else                                  $row++;
    }
    return false;
}
```

---

## 2.7 SPL — SplFixedArray

`SplFixedArray` est un tableau de taille fixe plus économe en mémoire que les tableaux PHP natifs (pas de hash table interne).

```php
<?php
declare(strict_types=1);

// Tableau de taille fixe — utile pour les gros volumes
$fixed = new SplFixedArray(1000);

$fixed[0]   = 'premier';
$fixed[999] = 'dernier';

echo $fixed->getSize();  // 1000
echo $fixed[0];          // premier

// Création depuis un tableau
$fixed2 = SplFixedArray::fromArray([10, 20, 30]);
echo $fixed2[1]; // 20

// Itération
foreach ($fixed2 as $i => $v) {
    echo "$i: $v\n";
}
```

!!! info "SplFixedArray vs array"
    - `SplFixedArray` : accès O(1), mémoire ~3× moins qu'un `array` PHP
    - `array` PHP : flexible, dynamique, mais consomme plus de mémoire par élément

---

## Exercices

!!! exercise "Exercice 2.1 — Rotation de tableau"
    Écrivez une fonction qui fait tourner un tableau de `k` positions vers la droite, en O(n) temps et O(1) espace supplémentaire.
    ```
    [1, 2, 3, 4, 5], k=2  →  [4, 5, 1, 2, 3]
    ```
    ??? success "Réponse"
        ```php
        function rotate(array &$arr, int $k): void
        {
            $n  = count($arr);
            $k %= $n;
            reverse($arr, 0, $n - 1);
            reverse($arr, 0, $k - 1);
            reverse($arr, $k, $n - 1);
        }

        function reverse(array &$arr, int $l, int $r): void
        {
            while ($l < $r) {
                [$arr[$l], $arr[$r]] = [$arr[$r], $arr[$l]];
                $l++; $r--;
            }
        }
        ```

!!! exercise "Exercice 2.2 — Deux sommes"
    Étant donné un tableau d'entiers et une cible `$target`, retournez les indices des deux éléments dont la somme vaut `$target`. Solution en O(n).
    ```
    [2, 7, 11, 15], target=9  →  [0, 1]
    ```
    ??? success "Réponse"
        ```php
        function twoSum(array $nums, int $target): array
        {
            $map = [];
            foreach ($nums as $i => $v) {
                $complement = $target - $v;
                if (isset($map[$complement])) {
                    return [$map[$complement], $i];
                }
                $map[$v] = $i;
            }
            return [];
        }
        ```

!!! exercise "Exercice 2.3 — Maximum de sous-tableau (Kadane)"
    Trouvez le sous-tableau contigu dont la somme est maximale. Algorithme de Kadane en O(n).
    ```
    [-2, 1, -3, 4, -1, 2, 1, -5, 4]  →  6  (sous-tableau [4,-1,2,1])
    ```
