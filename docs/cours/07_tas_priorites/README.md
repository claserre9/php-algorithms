# Chapitre 7 — Tas & files de priorité

## 7.1 Le tas (Heap)

Un **tas** (heap) est un arbre binaire complet satisfaisant la **propriété de tas** :

- **Min-heap** : le parent est toujours **≤** ses enfants → la racine est le minimum
- **Max-heap** : le parent est toujours **≥** ses enfants → la racine est le maximum

```
  Min-heap          Max-heap
     1                 9
   /   \             /   \
  3     2           7     8
 / \   /           / \   /
8   5 4           2   5 3
```

### Représentation en tableau

Un arbre binaire complet peut être stocké dans un tableau sans pointeurs :

```
Index:  0   1   2   3   4   5
Valeur: 1   3   2   8   5   4

Parent de i      = (i-1) / 2
Enfant gauche    = 2*i + 1
Enfant droit     = 2*i + 2
```

| Opération | Complexité |
|-----------|-----------|
| Insérer | O(log n) |
| Extraire min/max | O(log n) |
| Consulter min/max | O(1) |
| Construire depuis tableau | O(n) |
| Recherche arbitraire | O(n) |

---

## 7.2 Implémentation du Min-Heap

```php
<?php
declare(strict_types=1);

class MinHeap
{
    /** @var array<int, int|float> */
    private array $data = [];

    public function taille(): int   { return count($this->data); }
    public function estVide(): bool { return empty($this->data); }

    // ── Consultation du minimum ──────────────────────────────────

    public function min(): int|float
    {
        if ($this->estVide()) throw new UnderflowException("Heap vide");
        return $this->data[0];
    }

    // ── Insertion — O(log n) ─────────────────────────────────────

    public function inserer(int|float $valeur): void
    {
        $this->data[] = $valeur;          // ajouter à la fin
        $this->remonter(count($this->data) - 1);  // remonter (sift up)
    }

    /** Remonte l'élément à l'index i tant qu'il est inférieur à son parent */
    private function remonter(int $i): void
    {
        while ($i > 0) {
            $parent = intdiv($i - 1, 2);

            if ($this->data[$parent] <= $this->data[$i]) break;  // propriété respectée

            $this->echanger($i, $parent);
            $i = $parent;
        }
    }

    // ── Extraction du minimum — O(log n) ─────────────────────────

    public function extraireMin(): int|float
    {
        if ($this->estVide()) throw new UnderflowException("Heap vide");

        $min  = $this->data[0];
        $last = array_pop($this->data);

        if (!$this->estVide()) {
            $this->data[0] = $last;         // placer le dernier à la racine
            $this->descendre(0);            // faire descendre (sift down)
        }
        return $min;
    }

    /** Descend l'élément à l'index i tant qu'il est supérieur à un enfant */
    private function descendre(int $i): void
    {
        $n = count($this->data);

        while (true) {
            $gauche = 2 * $i + 1;
            $droite = 2 * $i + 2;
            $min    = $i;

            if ($gauche < $n && $this->data[$gauche] < $this->data[$min]) {
                $min = $gauche;
            }
            if ($droite < $n && $this->data[$droite] < $this->data[$min]) {
                $min = $droite;
            }

            if ($min === $i) break;  // déjà à la bonne place

            $this->echanger($i, $min);
            $i = $min;
        }
    }

    // ── Construction depuis un tableau — O(n) ────────────────────
    // Plus efficace que n insertions individuelles (O(n log n))

    public static function depuisTableau(array $arr): self
    {
        $heap = new self();
        $heap->data = $arr;

        // Appliquer heapify depuis le dernier parent vers la racine
        $n = count($arr);
        for ($i = intdiv($n, 2) - 1; $i >= 0; $i--) {
            $heap->descendre($i);
        }
        return $heap;
    }

    // ── Modification de priorité ─────────────────────────────────

    public function diminuerCle(int $i, int|float $nouvelleValeur): void
    {
        if ($nouvelleValeur > $this->data[$i]) {
            throw new InvalidArgumentException("La nouvelle valeur doit être inférieure");
        }
        $this->data[$i] = $nouvelleValeur;
        $this->remonter($i);
    }

    // ── Utilitaires ──────────────────────────────────────────────

    private function echanger(int $i, int $j): void
    {
        [$this->data[$i], $this->data[$j]] = [$this->data[$j], $this->data[$i]];
    }

    public function versTableau(): array { return $this->data; }

    public function __toString(): string
    {
        return 'MinHeap[' . implode(', ', $this->data) . ']';
    }
}
```

---

## 7.3 Max-Heap

Le Max-Heap est identique au Min-Heap, les comparaisons sont inversées :

```php
<?php
declare(strict_types=1);

class MaxHeap extends MinHeap
{
    // On inverse la propriété : parent >= enfants

    private array $data = [];

    public function inserer(int|float $valeur): void
    {
        $this->data[] = $valeur;
        $this->remonter(count($this->data) - 1);
    }

    private function remonter(int $i): void
    {
        while ($i > 0) {
            $parent = intdiv($i - 1, 2);
            if ($this->data[$parent] >= $this->data[$i]) break;  // max-heap
            [$this->data[$i], $this->data[$parent]] = [$this->data[$parent], $this->data[$i]];
            $i = $parent;
        }
    }

    public function max(): int|float { return $this->data[0]; }

    public function extraireMax(): int|float
    {
        $max  = $this->data[0];
        $last = array_pop($this->data);
        if (!empty($this->data)) {
            $this->data[0] = $last;
            $this->descendre(0);
        }
        return $max;
    }

    private function descendre(int $i): void
    {
        $n = count($this->data);
        while (true) {
            $gauche = 2 * $i + 1;
            $droite = 2 * $i + 2;
            $max    = $i;

            if ($gauche < $n && $this->data[$gauche] > $this->data[$max]) $max = $gauche;
            if ($droite < $n && $this->data[$droite] > $this->data[$max]) $max = $droite;

            if ($max === $i) break;
            [$this->data[$i], $this->data[$max]] = [$this->data[$max], $this->data[$i]];
            $i = $max;
        }
    }
}
```

---

## 7.4 File de priorité (Priority Queue)

Une **file de priorité** donne toujours accès à l'élément de plus haute (ou basse) priorité.

```php
<?php
declare(strict_types=1);

class ElementPrioritaire
{
    public function __construct(
        public readonly mixed $valeur,
        public readonly int   $priorite
    ) {}
}

class FilePrioritaire
{
    /** @var array<int, ElementPrioritaire> */
    private array $data = [];

    public function estVide(): bool { return empty($this->data); }
    public function taille(): int   { return count($this->data); }

    /** Enqueue — O(log n) */
    public function enfiler(mixed $valeur, int $priorite): void
    {
        $this->data[] = new ElementPrioritaire($valeur, $priorite);
        $this->remonter(count($this->data) - 1);
    }

    /** Dequeue — retourne l'élément de priorité maximale — O(log n) */
    public function defiler(): ElementPrioritaire
    {
        if ($this->estVide()) throw new UnderflowException("File vide");

        $max  = $this->data[0];
        $last = array_pop($this->data);

        if (!$this->estVide()) {
            $this->data[0] = $last;
            $this->descendre(0);
        }
        return $max;
    }

    /** Consulter sans retirer — O(1) */
    public function sommet(): ElementPrioritaire
    {
        if ($this->estVide()) throw new UnderflowException("File vide");
        return $this->data[0];
    }

    private function remonter(int $i): void
    {
        while ($i > 0) {
            $parent = intdiv($i - 1, 2);
            if ($this->data[$parent]->priorite >= $this->data[$i]->priorite) break;
            [$this->data[$i], $this->data[$parent]] = [$this->data[$parent], $this->data[$i]];
            $i = $parent;
        }
    }

    private function descendre(int $i): void
    {
        $n = count($this->data);
        while (true) {
            $g   = 2 * $i + 1;
            $d   = 2 * $i + 2;
            $max = $i;

            if ($g < $n && $this->data[$g]->priorite > $this->data[$max]->priorite) $max = $g;
            if ($d < $n && $this->data[$d]->priorite > $this->data[$max]->priorite) $max = $d;

            if ($max === $i) break;
            [$this->data[$i], $this->data[$max]] = [$this->data[$max], $this->data[$i]];
            $i = $max;
        }
    }
}
```

---

## 7.5 SplMinHeap et SplMaxHeap (PHP natif)

```php
<?php
declare(strict_types=1);

// ── SplMinHeap ───────────────────────────────────────────────
$minHeap = new SplMinHeap();
$minHeap->insert(5);
$minHeap->insert(3);
$minHeap->insert(8);
$minHeap->insert(1);

while (!$minHeap->isEmpty()) {
    echo $minHeap->extract() . ' ';  // 1 3 5 8 — toujours le minimum
}

// ── SplMaxHeap ───────────────────────────────────────────────
$maxHeap = new SplMaxHeap();
$maxHeap->insert(5);
$maxHeap->insert(3);
$maxHeap->insert(8);
$maxHeap->insert(1);

while (!$maxHeap->isEmpty()) {
    echo $maxHeap->extract() . ' ';  // 8 5 3 1 — toujours le maximum
}

// ── SplPriorityQueue ─────────────────────────────────────────
$pq = new SplPriorityQueue();
$pq->insert("basse priorité", 1);
$pq->insert("urgente",        10);
$pq->insert("normale",        5);

while (!$pq->isEmpty()) {
    echo $pq->extract() . "\n";
    // urgente
    // normale
    // basse priorité
}
```

---

## 7.6 Algorithmes basés sur le tas

### 7.6.1 Tri par tas (Heap Sort)

```php
<?php
declare(strict_types=1);

// O(n log n) temps, O(1) espace supplémentaire
function heapSort(array $arr): array
{
    $n = count($arr);

    // Phase 1 : construire un max-heap in-place — O(n)
    for ($i = intdiv($n, 2) - 1; $i >= 0; $i--) {
        heapify($arr, $n, $i);
    }

    // Phase 2 : extraire les éléments un par un — O(n log n)
    for ($i = $n - 1; $i > 0; $i--) {
        [$arr[0], $arr[$i]] = [$arr[$i], $arr[0]];  // max en fin
        heapify($arr, $i, 0);  // rétablir le heap sur les i premiers
    }

    return $arr;
}

function heapify(array &$arr, int $n, int $i): void
{
    $max    = $i;
    $gauche = 2 * $i + 1;
    $droite = 2 * $i + 2;

    if ($gauche < $n && $arr[$gauche] > $arr[$max]) $max = $gauche;
    if ($droite < $n && $arr[$droite] > $arr[$max]) $max = $droite;

    if ($max !== $i) {
        [$arr[$i], $arr[$max]] = [$arr[$max], $arr[$i]];
        heapify($arr, $n, $max);
    }
}

print_r(heapSort([3, 1, 4, 1, 5, 9, 2, 6]));
// [1, 1, 2, 3, 4, 5, 6, 9]
```

### 7.6.2 K plus grands éléments

```php
<?php
declare(strict_types=1);

// Trouver les k plus grands éléments — O(n log k)
// Technique : min-heap de taille k
function kPlusGrands(array $nums, int $k): array
{
    $heap = new SplMinHeap();

    foreach ($nums as $v) {
        $heap->insert($v);
        if ($heap->count() > $k) {
            $heap->extract();  // éjecter le plus petit
        }
    }

    $result = [];
    while (!$heap->isEmpty()) {
        $result[] = $heap->extract();
    }
    return array_reverse($result);
}

print_r(kPlusGrands([3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5], 3));
// [9, 6, 5]
```

### 7.6.3 Médiane en temps réel (streaming)

```php
<?php
declare(strict_types=1);

/**
 * Calcule la médiane après chaque insertion en O(log n).
 * Deux heaps : max-heap pour la moitié basse, min-heap pour la moitié haute.
 */
class MedianeGlissante
{
    private SplMaxHeap $moitieBasse;   // éléments <= médiane
    private SplMinHeap $moitiéHaute;  // éléments > médiane

    public function __construct()
    {
        $this->moitieBasse  = new SplMaxHeap();
        $this->moitiéHaute  = new SplMinHeap();
    }

    public function ajouter(int|float $num): void
    {
        // Insérer dans la bonne moitié
        if ($this->moitieBasse->isEmpty() || $num <= $this->moitieBasse->top()) {
            $this->moitieBasse->insert($num);
        } else {
            $this->moitiéHaute->insert($num);
        }

        // Équilibrer les deux moitiés (différence max 1)
        if ($this->moitieBasse->count() > $this->moitiéHaute->count() + 1) {
            $this->moitiéHaute->insert($this->moitieBasse->extract());
        } elseif ($this->moitiéHaute->count() > $this->moitieBasse->count()) {
            $this->moitieBasse->insert($this->moitiéHaute->extract());
        }
    }

    public function mediane(): float
    {
        if ($this->moitieBasse->count() > $this->moitiéHaute->count()) {
            return $this->moitieBasse->top();
        }
        return ($this->moitieBasse->top() + $this->moitiéHaute->top()) / 2.0;
    }
}

$mg = new MedianeGlissante();
foreach ([2, 3, 4, 5, 1] as $v) {
    $mg->ajouter($v);
    echo $mg->mediane() . "\n";  // 2, 2.5, 3, 3.5, 3
}
```

---

## Exercices

!!! exercise "Exercice 7.1 — Fusion de k listes triées"
    Fusionnez k listes triées en une seule liste triée. Solution optimale en O(n log k) avec un min-heap.
    ```
    [[1,4,7], [2,5,8], [3,6,9]]  →  [1,2,3,4,5,6,7,8,9]
    ```

!!! exercise "Exercice 7.2 — Kième plus petit élément"
    Trouvez le kième plus petit élément dans un tableau non trié en O(n log k).

!!! exercise "Exercice 7.3 — Chemin minimum (Dijkstra)"
    Implémentez Dijkstra pour trouver le plus court chemin dans un graphe pondéré, en utilisant une file de priorité (min-heap). Voir aussi le chapitre 8.
