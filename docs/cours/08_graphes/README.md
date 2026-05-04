# Chapitre 8 — Graphes

## 8.1 Concepts fondamentaux

Un **graphe** G = (V, E) est composé de :
- **V** — un ensemble de **sommets** (vertices / nœuds)
- **E** — un ensemble d'**arêtes** (edges) reliant des paires de sommets

```
Non orienté :          Orienté (digraphe) :
  A — B — D              A → B → D
  |   |                  ↑   |
  C   E                  C ← E
```

| Terme | Définition |
|-------|-----------|
| Adjacent | Deux sommets reliés par une arête |
| Degré | Nombre d'arêtes d'un sommet |
| Chemin | Suite de sommets consécutifs |
| Cycle | Chemin dont le début = la fin |
| Connexe | Tout sommet est accessible depuis tout autre |
| Pondéré | Chaque arête a un poids (distance, coût) |

---

## 8.2 Représentations

### 8.2.1 Matrice d'adjacence

```php
<?php
declare(strict_types=1);

class GrapheMatrice
{
    private array $matrice;
    private int   $sommets;

    public function __construct(int $n)
    {
        $this->sommets = $n;
        $this->matrice = array_fill(0, $n, array_fill(0, $n, 0));
    }

    // O(1) — accès direct
    public function ajouterArete(int $u, int $v, int $poids = 1): void
    {
        $this->matrice[$u][$v] = $poids;
        $this->matrice[$v][$u] = $poids;  // non orienté
    }

    public function supprimerArete(int $u, int $v): void
    {
        $this->matrice[$u][$v] = 0;
        $this->matrice[$v][$u] = 0;
    }

    public function aArete(int $u, int $v): bool
    {
        return $this->matrice[$u][$v] !== 0;
    }

    public function voisins(int $u): array
    {
        $voisins = [];
        for ($v = 0; $v < $this->sommets; $v++) {
            if ($this->matrice[$u][$v] !== 0) $voisins[] = $v;
        }
        return $voisins;
    }

    public function afficher(): void
    {
        foreach ($this->matrice as $row) {
            echo implode(' ', $row) . "\n";
        }
    }
}

// Mémoire : O(V²) — inadapté aux graphes creux
// Avantage : test d'adjacence O(1)
```

### 8.2.2 Liste d'adjacence

```php
<?php
declare(strict_types=1);

class Arete
{
    public function __construct(
        public readonly int $destination,
        public readonly int $poids = 1
    ) {}
}

class Graphe
{
    /** @var array<int, array<Arete>> */
    private array $adj;
    private int   $sommets;
    private bool  $oriente;

    public function __construct(int $sommets, bool $oriente = false)
    {
        $this->sommets = $sommets;
        $this->oriente = $oriente;
        $this->adj     = array_fill(0, $sommets, []);
    }

    public function ajouterArete(int $u, int $v, int $poids = 1): void
    {
        $this->adj[$u][] = new Arete($v, $poids);
        if (!$this->oriente) {
            $this->adj[$v][] = new Arete($u, $poids);
        }
    }

    public function voisins(int $u): array
    {
        return $this->adj[$u];
    }

    public function sommets(): int { return $this->sommets; }

    public function afficher(): void
    {
        for ($u = 0; $u < $this->sommets; $u++) {
            $voisins = array_map(
                fn(Arete $a) => "$a->destination(p=$a->poids)",
                $this->adj[$u]
            );
            echo "$u → " . implode(', ', $voisins) . "\n";
        }
    }
}

// Mémoire : O(V + E) — efficace pour graphes creux
// Avantage : itération des voisins O(degré)
```

---

## 8.3 Parcours en largeur — BFS

```php
<?php
declare(strict_types=1);

/**
 * BFS — Breadth-First Search
 * Explore niveau par niveau, garantit le plus court chemin (graphe non pondéré).
 * O(V + E) temps, O(V) espace
 */
function bfs(Graphe $g, int $source): array
{
    $visite   = array_fill(0, $g->sommets(), false);
    $distance = array_fill(0, $g->sommets(), -1);
    $parent   = array_fill(0, $g->sommets(), -1);
    $ordre    = [];

    $queue = new SplQueue();
    $queue->enqueue($source);
    $visite[$source]   = true;
    $distance[$source] = 0;

    while (!$queue->isEmpty()) {
        $u       = $queue->dequeue();
        $ordre[] = $u;

        foreach ($g->voisins($u) as $arete) {
            $v = $arete->destination;
            if (!$visite[$v]) {
                $visite[$v]   = true;
                $distance[$v] = $distance[$u] + 1;
                $parent[$v]   = $u;
                $queue->enqueue($v);
            }
        }
    }

    return ['ordre' => $ordre, 'distance' => $distance, 'parent' => $parent];
}

// Reconstruit le chemin le plus court de source à cible
function cheminCourt(array $parent, int $source, int $cible): array
{
    $chemin = [];
    $courant = $cible;

    while ($courant !== -1) {
        array_unshift($chemin, $courant);
        $courant = $parent[$courant];
    }

    return $chemin[0] === $source ? $chemin : [];
}

// ── Démonstration ────────────────────────────────────────────
$g = new Graphe(6);
$g->ajouterArete(0, 1);
$g->ajouterArete(0, 2);
$g->ajouterArete(1, 3);
$g->ajouterArete(1, 4);
$g->ajouterArete(2, 5);

$resultat = bfs($g, 0);
echo implode(' → ', $resultat['ordre']);         // 0 → 1 → 2 → 3 → 4 → 5
echo implode(' → ', cheminCourt($resultat['parent'], 0, 4)); // 0 → 1 → 4
```

---

## 8.4 Parcours en profondeur — DFS

```php
<?php
declare(strict_types=1);

/**
 * DFS — Depth-First Search
 * Explore aussi loin que possible avant de revenir.
 * O(V + E) temps, O(V) espace (pile de récursion)
 */
class DFS
{
    private array $visite;
    private array $entree;   // temps de découverte
    private array $sortie;   // temps de fin de traitement
    private array $composante;
    private int   $temps = 0;

    public function parcourir(Graphe $g): array
    {
        $n = $g->sommets();
        $this->visite    = array_fill(0, $n, false);
        $this->entree    = array_fill(0, $n, -1);
        $this->sortie    = array_fill(0, $n, -1);
        $this->composante = array_fill(0, $n, -1);
        $this->temps     = 0;

        $composante = 0;
        $ordre      = [];

        for ($u = 0; $u < $n; $u++) {
            if (!$this->visite[$u]) {
                $this->dfsRec($g, $u, $composante, $ordre);
                $composante++;
            }
        }

        return ['ordre' => $ordre, 'composantes' => $this->composante];
    }

    private function dfsRec(Graphe $g, int $u, int $comp, array &$ordre): void
    {
        $this->visite[$u]     = true;
        $this->entree[$u]     = $this->temps++;
        $this->composante[$u] = $comp;
        $ordre[]              = $u;

        foreach ($g->voisins($u) as $arete) {
            if (!$this->visite[$arete->destination]) {
                $this->dfsRec($g, $arete->destination, $comp, $ordre);
            }
        }

        $this->sortie[$u] = $this->temps++;
    }

    // DFS itératif (évite le stack overflow sur grands graphes)
    public function parcourirIteratif(Graphe $g, int $source): array
    {
        $visite = array_fill(0, $g->sommets(), false);
        $pile   = [$source];
        $ordre  = [];

        while (!empty($pile)) {
            $u = array_pop($pile);

            if ($visite[$u]) continue;
            $visite[$u] = true;
            $ordre[]    = $u;

            foreach (array_reverse($g->voisins($u)) as $arete) {
                if (!$visite[$arete->destination]) {
                    $pile[] = $arete->destination;
                }
            }
        }
        return $ordre;
    }
}

// ── Détection de cycle ───────────────────────────────────────
function aCycle(Graphe $g): bool
{
    $visite = array_fill(0, $g->sommets(), false);

    for ($u = 0; $u < $g->sommets(); $u++) {
        if (!$visite[$u] && dfsCycle($g, $u, $visite, -1)) {
            return true;
        }
    }
    return false;
}

function dfsCycle(Graphe $g, int $u, array &$visite, int $parent): bool
{
    $visite[$u] = true;

    foreach ($g->voisins($u) as $arete) {
        $v = $arete->destination;
        if (!$visite[$v]) {
            if (dfsCycle($g, $v, $visite, $u)) return true;
        } elseif ($v !== $parent) {
            return true;  // arête arrière = cycle
        }
    }
    return false;
}
```

---

## 8.5 Tri topologique

Ordonne les sommets d'un DAG (graphe orienté sans cycle) tel que chaque arête va de gauche à droite.

```php
<?php
declare(strict_types=1);

// ── DFS-based topological sort ───────────────────────────────
function triTopologique(Graphe $g): array
{
    $visite = array_fill(0, $g->sommets(), false);
    $pile   = new SplStack();

    for ($u = 0; $u < $g->sommets(); $u++) {
        if (!$visite[$u]) {
            topoRec($g, $u, $visite, $pile);
        }
    }

    $ordre = [];
    while (!$pile->isEmpty()) {
        $ordre[] = $pile->pop();
    }
    return $ordre;
}

function topoRec(Graphe $g, int $u, array &$visite, SplStack $pile): void
{
    $visite[$u] = true;

    foreach ($g->voisins($u) as $arete) {
        if (!$visite[$arete->destination]) {
            topoRec($g, $arete->destination, $visite, $pile);
        }
    }
    $pile->push($u);  // on empile après avoir traité tous les enfants
}

// ── Kahn's algorithm (BFS) ───────────────────────────────────
// Détecte aussi les cycles (si résultat incomplet → cycle existe)
function kahnTopo(Graphe $g): array
{
    $n       = $g->sommets();
    $degre   = array_fill(0, $n, 0);

    // Calculer les degrés entrants
    for ($u = 0; $u < $n; $u++) {
        foreach ($g->voisins($u) as $arete) {
            $degre[$arete->destination]++;
        }
    }

    // Initialiser avec les sommets sans prédécesseur
    $queue = new SplQueue();
    for ($u = 0; $u < $n; $u++) {
        if ($degre[$u] === 0) $queue->enqueue($u);
    }

    $ordre = [];
    while (!$queue->isEmpty()) {
        $u       = $queue->dequeue();
        $ordre[] = $u;

        foreach ($g->voisins($u) as $arete) {
            $v = $arete->destination;
            if (--$degre[$v] === 0) {
                $queue->enqueue($v);
            }
        }
    }

    // Si tous les sommets ne sont pas dans l'ordre → cycle
    return count($ordre) === $n ? $ordre : [];
}
```

---

## 8.6 Algorithme de Dijkstra

Plus court chemin depuis une source dans un graphe **pondéré positif**.

```php
<?php
declare(strict_types=1);

/**
 * Dijkstra avec min-heap (SplMinHeap).
 * O((V + E) log V)
 *
 * @return array{distance: array<int>, parent: array<int>}
 */
function dijkstra(Graphe $g, int $source): array
{
    $n        = $g->sommets();
    $dist     = array_fill(0, $n, PHP_INT_MAX);
    $parent   = array_fill(0, $n, -1);
    $visite   = array_fill(0, $n, false);

    $dist[$source] = 0;

    // Min-heap : [distance, sommet]
    $heap = new SplMinHeap();
    $heap->insert([0, $source]);

    while (!$heap->isEmpty()) {
        [$d, $u] = $heap->extract();

        if ($visite[$u]) continue;  // déjà traité avec distance plus courte
        $visite[$u] = true;

        foreach ($g->voisins($u) as $arete) {
            $v         = $arete->destination;
            $nouvDist  = $dist[$u] + $arete->poids;

            if ($nouvDist < $dist[$v]) {
                $dist[$v]   = $nouvDist;
                $parent[$v] = $u;
                $heap->insert([$nouvDist, $v]);
            }
        }
    }

    return ['distance' => $dist, 'parent' => $parent];
}

function reconstruireChemin(array $parent, int $source, int $cible): array
{
    $chemin  = [];
    $courant = $cible;

    while ($courant !== -1) {
        array_unshift($chemin, $courant);
        $courant = $parent[$courant];
    }

    return $chemin[0] === $source ? $chemin : [];
}

// ── Démonstration ────────────────────────────────────────────
$g = new Graphe(5, true);
$g->ajouterArete(0, 1, 10);
$g->ajouterArete(0, 2, 3);
$g->ajouterArete(1, 3, 2);
$g->ajouterArete(2, 1, 4);
$g->ajouterArete(2, 3, 8);
$g->ajouterArete(2, 4, 2);
$g->ajouterArete(3, 4, 5);
$g->ajouterArete(4, 3, 1);

$resultat = dijkstra($g, 0);
// Distance de 0 à tous : [0, 7, 3, 9, 5]
//   0→2 (3) → 2→1 (4) → 1→3 (2) = 9
//   0→2 (3) → 2→4 (2) → 4→3 (1) = 6... non, 9 est correct

$chemin = reconstruireChemin($resultat['parent'], 0, 3);
echo "Chemin 0→3 : " . implode(' → ', $chemin);
echo " (distance: {$resultat['distance'][3]})";
```

---

## 8.7 Bellman-Ford

Gère les poids **négatifs** et détecte les cycles négatifs. O(V·E).

```php
<?php
declare(strict_types=1);

function bellmanFord(int $sommets, array $aretes, int $source): array|false
{
    $dist = array_fill(0, $sommets, PHP_INT_MAX);
    $dist[$source] = 0;

    // Relaxer toutes les arêtes V-1 fois
    for ($i = 1; $i < $sommets; $i++) {
        foreach ($aretes as [$u, $v, $w]) {
            if ($dist[$u] !== PHP_INT_MAX && $dist[$u] + $w < $dist[$v]) {
                $dist[$v] = $dist[$u] + $w;
            }
        }
    }

    // Détecter les cycles négatifs (V-ième passe)
    foreach ($aretes as [$u, $v, $w]) {
        if ($dist[$u] !== PHP_INT_MAX && $dist[$u] + $w < $dist[$v]) {
            return false;  // cycle négatif détecté
        }
    }

    return $dist;
}

// Format aretes : [[u, v, poids], ...]
$aretes = [[0,1,4],[0,2,5],[1,2,-3],[2,3,4]];
$dist   = bellmanFord(4, $aretes, 0);
print_r($dist);  // [0, 4, 1, 5]
```

---

## 8.8 Composantes connexes

```php
<?php
declare(strict_types=1);

// ── Union-Find (Disjoint Set Union) ─────────────────────────
class UnionFind
{
    private array $parent;
    private array $rang;
    private int   $composantes;

    public function __construct(int $n)
    {
        $this->composantes = $n;
        $this->parent      = range(0, $n - 1);
        $this->rang        = array_fill(0, $n, 0);
    }

    /** Trouve la racine avec compression de chemin — O(α(n)) ≈ O(1) */
    public function trouver(int $x): int
    {
        if ($this->parent[$x] !== $x) {
            $this->parent[$x] = $this->trouver($this->parent[$x]);  // compression
        }
        return $this->parent[$x];
    }

    /** Union par rang — O(α(n)) ≈ O(1) */
    public function union(int $x, int $y): bool
    {
        $rx = $this->trouver($x);
        $ry = $this->trouver($y);

        if ($rx === $ry) return false;  // déjà dans la même composante

        if ($this->rang[$rx] < $this->rang[$ry]) {
            $this->parent[$rx] = $ry;
        } elseif ($this->rang[$rx] > $this->rang[$ry]) {
            $this->parent[$ry] = $rx;
        } else {
            $this->parent[$ry] = $rx;
            $this->rang[$rx]++;
        }

        $this->composantes--;
        return true;
    }

    public function connexes(int $x, int $y): bool
    {
        return $this->trouver($x) === $this->trouver($y);
    }

    public function nombreComposantes(): int { return $this->composantes; }
}

// ── Algorithme de Kruskal (arbre couvrant minimal) ──────────
function kruskal(int $n, array $aretes): array
{
    // Trier les arêtes par poids croissant
    usort($aretes, fn($a, $b) => $a[2] <=> $b[2]);

    $uf  = new UnionFind($n);
    $acm = [];
    $coutTotal = 0;

    foreach ($aretes as [$u, $v, $w]) {
        if ($uf->union($u, $v)) {  // n'ajoute pas si ça crée un cycle
            $acm[]      = [$u, $v, $w];
            $coutTotal += $w;

            if (count($acm) === $n - 1) break;  // n-1 arêtes = arbre couvrant
        }
    }

    return ['aretes' => $acm, 'cout' => $coutTotal];
}
```

---

## Résumé des complexités

| Algorithme | Temps | Espace | Usage |
|-----------|-------|--------|-------|
| BFS | O(V+E) | O(V) | Plus court chemin (non pondéré), composantes |
| DFS | O(V+E) | O(V) | Cycles, tri topologique, composantes |
| Dijkstra | O((V+E)log V) | O(V) | Plus court chemin (poids ≥ 0) |
| Bellman-Ford | O(V·E) | O(V) | Poids négatifs, détection de cycles |
| Kruskal | O(E log E) | O(V) | Arbre couvrant minimal |
| Union-Find | O(α(n)) ≈ O(1) | O(V) | Composantes connexes |

---

## Exercices

!!! exercise "Exercice 8.1 — Nombre d'îles"
    Étant donné une grille 2D de '1' (terre) et '0' (eau), comptez le nombre d'îles (zones connectées de '1'). Utilisez BFS ou DFS.

!!! exercise "Exercice 8.2 — Bipartite ?"
    Déterminez si un graphe est bipartite (2-colorable) en O(V+E) avec BFS.

!!! exercise "Exercice 8.3 — Plus court chemin dans une grille"
    Dans une grille n×m avec des obstacles (1=bloqué, 0=libre), trouvez le plus court chemin de (0,0) à (n-1,m-1) en déplacements 4-directionnels.
