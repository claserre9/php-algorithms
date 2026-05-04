# Chapitre 3 — Piles, files & deques

## 3.1 La pile (Stack)

Une **pile** (ou *stack*) est une structure **LIFO** — Last In, First Out. Pensez à une pile d'assiettes : on pose toujours sur le dessus et on retire du dessus.

```
  ┌───┐
  │ 3 │  ← sommet (top)
  ├───┤
  │ 2 │
  ├───┤
  │ 1 │  ← base
  └───┘

push(4) →   pop() →
  ┌───┐       ┌───┐
  │ 4 │       │ 3 │  ← top
  ├───┤       ├───┤
  │ 3 │       │ 2 │
  ├───┤       ├───┤
  │ 2 │       │ 1 │
  ├───┤       └───┘
  │ 1 │
  └───┘
```

### Opérations

| Opération | Description | Complexité |
|-----------|-------------|-----------|
| `push(v)` | Empile v au sommet | O(1) |
| `pop()` | Dépile et retourne le sommet | O(1) |
| `peek()` | Lit le sommet sans dépiler | O(1) |
| `isEmpty()` | Vrai si la pile est vide | O(1) |
| `size()` | Nombre d'éléments | O(1) |

---

## 3.2 Implémentation de la Stack

```php
<?php
declare(strict_types=1);

/**
 * Pile générique LIFO.
 * Implémentée avec un tableau PHP (push/pop en fin = O(1)).
 */
class Stack
{
    private array $items = [];

    public function push(mixed $value): void
    {
        $this->items[] = $value;
    }

    public function pop(): mixed
    {
        if ($this->isEmpty()) {
            throw new UnderflowException("Stack vide — impossible de pop");
        }
        return array_pop($this->items);
    }

    public function peek(): mixed
    {
        if ($this->isEmpty()) {
            throw new UnderflowException("Stack vide");
        }
        return end($this->items);
    }

    public function isEmpty(): bool { return empty($this->items); }
    public function size(): int     { return count($this->items); }

    public function __toString(): string
    {
        return 'Stack(top→bas): [' . implode(', ', array_reverse($this->items)) . ']';
    }
}
```

---

## 3.3 Applications de la pile

### 3.3.1 Vérification des parenthèses

```php
<?php
declare(strict_types=1);

function parenthesesValides(string $s): bool
{
    $pile = new Stack();
    $paires = [')' => '(', '}' => '{', ']' => '['];

    for ($i = 0; $i < strlen($s); $i++) {
        $c = $s[$i];

        if (in_array($c, ['(', '{', '['])) {
            $pile->push($c);
        } elseif (isset($paires[$c])) {
            if ($pile->isEmpty() || $pile->pop() !== $paires[$c]) {
                return false;
            }
        }
    }
    return $pile->isEmpty();
}

var_dump(parenthesesValides("({[]})"));  // true
var_dump(parenthesesValides("([)]"));   // false
var_dump(parenthesesValides("{[}"));    // false
```

### 3.3.2 Évaluation d'expression postfixée (RPN)

```php
<?php
declare(strict_types=1);

// Expression : "3 4 + 2 * 7 /" = (3+4)*2/7
function evaluerRPN(string $expression): float
{
    $pile   = new Stack();
    $tokens = explode(' ', trim($expression));

    foreach ($tokens as $token) {
        if (is_numeric($token)) {
            $pile->push((float) $token);
        } else {
            $b = $pile->pop();
            $a = $pile->pop();
            $pile->push(match ($token) {
                '+'  => $a + $b,
                '-'  => $a - $b,
                '*'  => $a * $b,
                '/'  => $a / $b,
                '**' => $a ** $b,
                default => throw new InvalidArgumentException("Opérateur inconnu: $token"),
            });
        }
    }
    return $pile->pop();
}

echo evaluerRPN("3 4 + 2 * 7 /");  // 2
echo evaluerRPN("5 1 2 + 4 * + 3 -");  // 14
```

### 3.3.3 Historique de navigation (undo/redo)

```php
<?php
declare(strict_types=1);

class Editeur
{
    private Stack $historique;
    private Stack $futur;
    private string $texte = '';

    public function __construct()
    {
        $this->historique = new Stack();
        $this->futur      = new Stack();
    }

    public function ecrire(string $texte): void
    {
        $this->historique->push($this->texte);  // sauvegarde état
        $this->texte .= $texte;
        // Toute nouvelle action efface le futur
        $this->futur = new Stack();
    }

    public function annuler(): void   // Ctrl+Z
    {
        if ($this->historique->isEmpty()) return;
        $this->futur->push($this->texte);
        $this->texte = $this->historique->pop();
    }

    public function retablir(): void  // Ctrl+Y
    {
        if ($this->futur->isEmpty()) return;
        $this->historique->push($this->texte);
        $this->texte = $this->futur->pop();
    }

    public function getTexte(): string { return $this->texte; }
}

$ed = new Editeur();
$ed->ecrire("Hello");
$ed->ecrire(" World");
echo $ed->getTexte();   // Hello World

$ed->annuler();
echo $ed->getTexte();   // Hello

$ed->retablir();
echo $ed->getTexte();   // Hello World
```

### 3.3.4 Conversion décimal → binaire

```php
<?php
declare(strict_types=1);

function decimalVersBinaire(int $n): string
{
    if ($n === 0) return '0';

    $pile = new Stack();
    while ($n > 0) {
        $pile->push($n % 2);
        $n = intdiv($n, 2);
    }

    $binaire = '';
    while (!$pile->isEmpty()) {
        $binaire .= $pile->pop();
    }
    return $binaire;
}

echo decimalVersBinaire(13);  // 1101
echo decimalVersBinaire(255); // 11111111
```

---

## 3.4 La file (Queue)

Une **file** (ou *queue*) est une structure **FIFO** — First In, First Out. Pensez à une file d'attente au supermarché.

```
Entrée →  [3][2][1] → Sortie

enqueue(4):  [4][3][2][1]
dequeue():   [4][3][2]    retourne 1
```

### Opérations

| Opération | Description | Complexité |
|-----------|-------------|-----------|
| `enqueue(v)` | Ajoute en queue | O(1) |
| `dequeue()` | Retire et retourne la tête | O(1) |
| `front()` | Lit la tête sans retirer | O(1) |
| `isEmpty()` | Vrai si vide | O(1) |
| `size()` | Nombre d'éléments | O(1) |

---

## 3.5 Implémentation de la Queue

```php
<?php
declare(strict_types=1);

/**
 * File FIFO optimisée avec SplQueue.
 * SplDoublyLinkedList en dessous : enqueue/dequeue O(1).
 */
class Queue
{
    private SplQueue $items;

    public function __construct()
    {
        $this->items = new SplQueue();
    }

    public function enqueue(mixed $value): void
    {
        $this->items->enqueue($value);  // ajout en queue
    }

    public function dequeue(): mixed
    {
        if ($this->isEmpty()) {
            throw new UnderflowException("Queue vide");
        }
        return $this->items->dequeue();  // retrait en tête
    }

    public function front(): mixed
    {
        if ($this->isEmpty()) {
            throw new UnderflowException("Queue vide");
        }
        return $this->items->bottom();  // premier entré
    }

    public function isEmpty(): bool { return $this->items->isEmpty(); }
    public function size(): int     { return $this->items->count(); }
}
```

### Implémentation manuelle avec tableau circulaire

```php
<?php
declare(strict_types=1);

/**
 * Queue en tableau circulaire.
 * Évite le O(n) de array_shift — head/tail bougent, le tableau reste fixe.
 */
class CircularQueue
{
    private array $data;
    private int   $head     = 0;
    private int   $tail     = 0;
    private int   $count    = 0;
    private int   $capacity;

    public function __construct(int $capacity = 8)
    {
        $this->capacity = $capacity;
        $this->data     = array_fill(0, $capacity, null);
    }

    public function enqueue(mixed $value): void
    {
        if ($this->count === $this->capacity) {
            throw new OverflowException("Queue pleine");
        }
        $this->data[$this->tail] = $value;
        $this->tail = ($this->tail + 1) % $this->capacity;  // wrap-around
        $this->count++;
    }

    public function dequeue(): mixed
    {
        if ($this->isEmpty()) {
            throw new UnderflowException("Queue vide");
        }
        $value      = $this->data[$this->head];
        $this->head = ($this->head + 1) % $this->capacity;  // wrap-around
        $this->count--;
        return $value;
    }

    public function front(): mixed
    {
        if ($this->isEmpty()) throw new UnderflowException("Queue vide");
        return $this->data[$this->head];
    }

    public function isEmpty(): bool { return $this->count === 0; }
    public function size(): int     { return $this->count; }
}
```

---

## 3.6 Applications de la file

### 3.6.1 BFS — Parcours en largeur d'un graphe

```php
<?php
declare(strict_types=1);

function bfs(array $graph, int $depart): array
{
    $visite = [];
    $ordre  = [];
    $queue  = new Queue();

    $queue->enqueue($depart);
    $visite[$depart] = true;

    while (!$queue->isEmpty()) {
        $noeud  = $queue->dequeue();
        $ordre[] = $noeud;

        foreach ($graph[$noeud] as $voisin) {
            if (!isset($visite[$voisin])) {
                $visite[$voisin] = true;
                $queue->enqueue($voisin);
            }
        }
    }
    return $ordre;
}

$graphe = [
    0 => [1, 2],
    1 => [0, 3, 4],
    2 => [0, 5],
    3 => [1],
    4 => [1],
    5 => [2],
];

print_r(bfs($graphe, 0)); // [0, 1, 2, 3, 4, 5]
```

### 3.6.2 Simulation de file d'attente

```php
<?php
declare(strict_types=1);

class Caisse
{
    private Queue $file;
    private string $nom;

    public function __construct(string $nom)
    {
        $this->file = new Queue();
        $this->nom  = $nom;
    }

    public function arriver(string $client): void
    {
        $this->file->enqueue($client);
        echo "[$this->nom] $client rejoint la file ({$this->file->size()} en attente)\n";
    }

    public function servir(): void
    {
        if ($this->file->isEmpty()) {
            echo "[$this->nom] Personne en attente.\n";
            return;
        }
        $client = $this->file->dequeue();
        echo "[$this->nom] Service de $client\n";
    }
}

$caisse = new Caisse("Caisse 1");
$caisse->arriver("Alice");
$caisse->arriver("Bob");
$caisse->arriver("Charlie");
$caisse->servir();  // Alice
$caisse->servir();  // Bob
$caisse->arriver("Diane");
$caisse->servir();  // Charlie
$caisse->servir();  // Diane
```

---

## 3.7 Le deque (Double-Ended Queue)

Un **deque** permet d'ajouter et de retirer des deux côtés en O(1).

```php
<?php
declare(strict_types=1);

class Deque
{
    private SplDoublyLinkedList $list;

    public function __construct()
    {
        $this->list = new SplDoublyLinkedList();
    }

    // ── Côté avant (tête) ───────────────────────────────────
    public function addFront(mixed $value): void { $this->list->unshift($value); }
    public function removeFront(): mixed
    {
        if ($this->isEmpty()) throw new UnderflowException("Deque vide");
        return $this->list->shift();
    }
    public function peekFront(): mixed { return $this->list->bottom(); }

    // ── Côté arrière (queue) ────────────────────────────────
    public function addBack(mixed $value): void  { $this->list->push($value); }
    public function removeBack(): mixed
    {
        if ($this->isEmpty()) throw new UnderflowException("Deque vide");
        return $this->list->pop();
    }
    public function peekBack(): mixed { return $this->list->top(); }

    public function isEmpty(): bool { return $this->list->isEmpty(); }
    public function size(): int     { return $this->list->count(); }
}
```

### Application : fenêtre glissante maximum

```php
<?php
declare(strict_types=1);

/**
 * Pour chaque fenêtre de taille k, retourne le maximum.
 * Algorithme O(n) avec un deque d'indices.
 */
function maxGlissant(array $nums, int $k): array
{
    $deque  = new SplDoublyLinkedList();
    $result = [];

    for ($i = 0; $i < count($nums); $i++) {
        // Retirer les éléments hors de la fenêtre
        while (!$deque->isEmpty() && $deque->bottom() < $i - $k + 1) {
            $deque->shift();
        }
        // Retirer les éléments plus petits que le courant
        while (!$deque->isEmpty() && $nums[$deque->top()] < $nums[$i]) {
            $deque->pop();
        }
        $deque->push($i);

        // La fenêtre est pleine
        if ($i >= $k - 1) {
            $result[] = $nums[$deque->bottom()];
        }
    }
    return $result;
}

print_r(maxGlissant([1, 3, -1, -3, 5, 3, 6, 7], 3));
// [3, 3, 5, 5, 6, 7]
```

---

## 3.8 SPL intégré

PHP fournit des structures SPL natives :

```php
<?php
declare(strict_types=1);

// SplStack — pile LIFO
$stack = new SplStack();
$stack->push(1);
$stack->push(2);
$stack->push(3);
echo $stack->top();  // 3
echo $stack->pop();  // 3

// SplQueue — file FIFO
$queue = new SplQueue();
$queue->enqueue('a');
$queue->enqueue('b');
echo $queue->dequeue(); // a

// SplDoublyLinkedList — deque complet
$dll = new SplDoublyLinkedList();
$dll->push('milieu');
$dll->unshift('début');
$dll->push('fin');
// [début, milieu, fin]
```

---

## Résumé des complexités

| Structure | push/enqueue | pop/dequeue | peek | search |
|-----------|-------------|-------------|------|--------|
| Stack (tableau) | O(1) | O(1) | O(1) | O(n) |
| Queue (SplQueue) | O(1) | O(1) | O(1) | O(n) |
| Queue circulaire | O(1) | O(1) | O(1) | O(n) |
| Deque (SplDLL) | O(1) | O(1) | O(1) | O(n) |

---

## Exercices

!!! exercise "Exercice 3.1 — Tri par pile"
    Triez une pile sans utiliser d'autre structure que des piles et des variables locales. Complexité attendue : O(n²) temps, O(n) espace.

!!! exercise "Exercice 3.2 — Générer les nombres binaires"
    En utilisant une file, générez les n premiers nombres binaires :
    ```
    n=5 → ["1", "10", "11", "100", "101"]
    ```
    ??? success "Réponse"
        ```php
        function genererBinaires(int $n): array
        {
            $queue  = new Queue();
            $result = [];
            $queue->enqueue("1");

            for ($i = 0; $i < $n; $i++) {
                $s = $queue->dequeue();
                $result[] = $s;
                $queue->enqueue($s . "0");
                $queue->enqueue($s . "1");
            }
            return $result;
        }
        ```

!!! exercise "Exercice 3.3 — Palindrome avec deque"
    Utilisez un deque pour vérifier si une chaîne est un palindrome en O(n).
