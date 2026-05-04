# Chapitre 5 — Tables de hachage

## 5.1 Concept

Une **table de hachage** (hash table) associe des clés à des valeurs avec des opérations en O(1) amorti. C'est la structure derrière les `array` PHP associatifs, les `dict` Python, les `HashMap` Java.

```
clé "alice"  ──▶  h("alice") = 3  ──▶  bucket[3]: [("alice", 42)]
clé "bob"    ──▶  h("bob")   = 7  ──▶  bucket[7]: [("bob", 99)]
clé "charlie"──▶  h("charlie")= 3  ──▶  bucket[3]: [("alice",42),("charlie",5)]
                                                                  ↑ collision !
```

---

## 5.2 Fonctions de hachage

Une bonne fonction de hachage doit :

1. **Être déterministe** — même clé → même hash
2. **Distribuer uniformément** — éviter les clusters
3. **Être rapide** — O(1) ou O(|clé|)

```php
<?php
declare(strict_types=1);

// ── Hachage polynomial pour les chaînes ─────────────────────
// H(s) = s[0]*p^(n-1) + s[1]*p^(n-2) + ... + s[n-1]
// p = nombre premier (31), m = taille de la table (premier)
function hashChaine(string $cle, int $taille): int
{
    $hash = 0;
    $p    = 31;
    $m    = $taille;
    $puissance = 1;

    for ($i = 0; $i < strlen($cle); $i++) {
        $hash      = ($hash + ord($cle[$i]) * $puissance) % $m;
        $puissance = ($puissance * $p) % $m;
    }
    return $hash;
}

// ── Hachage djb2 (Daniel J. Bernstein) ──────────────────────
function djb2(string $cle, int $taille): int
{
    $hash = 5381;
    for ($i = 0; $i < strlen($cle); $i++) {
        $hash = (($hash << 5) + $hash + ord($cle[$i])) & 0x7FFFFFFF;
    }
    return $hash % $taille;
}

// PHP natif
echo crc32("alice");   // hash 32 bits
echo hash("xxh3", "alice");  // xxHash3 très rapide
```

---

## 5.3 Gestion des collisions

### 5.3.1 Chaînage (Separate Chaining)

Chaque bucket contient une liste de paires (clé, valeur).

```php
<?php
declare(strict_types=1);

class HashMapChainee
{
    private array $buckets;
    private int   $capacite;
    private int   $taille = 0;
    private float $facteurCharge = 0.75;

    public function __construct(int $capacite = 16)
    {
        $this->capacite = $capacite;
        $this->buckets  = array_fill(0, $capacite, []);
    }

    // ── Hash ────────────────────────────────────────────────────

    private function hash(string $cle): int
    {
        $h = 0;
        for ($i = 0; $i < strlen($cle); $i++) {
            $h = ($h * 31 + ord($cle[$i])) & 0x7FFFFFFF;
        }
        return $h % $this->capacite;
    }

    // ── Opérations CRUD ────────────────────────────────────────

    /** Insertion / mise à jour — O(1) amorti */
    public function put(string $cle, mixed $valeur): void
    {
        $index = $this->hash($cle);

        foreach ($this->buckets[$index] as &$paire) {
            if ($paire[0] === $cle) {
                $paire[1] = $valeur;  // mise à jour
                return;
            }
        }

        $this->buckets[$index][] = [$cle, $valeur];
        $this->taille++;

        if ($this->taille / $this->capacite > $this->facteurCharge) {
            $this->rehacher();
        }
    }

    /** Lecture — O(1) amorti */
    public function get(string $cle): mixed
    {
        $index = $this->hash($cle);

        foreach ($this->buckets[$index] as $paire) {
            if ($paire[0] === $cle) return $paire[1];
        }
        return null;
    }

    /** Suppression — O(1) amorti */
    public function supprimer(string $cle): bool
    {
        $index = $this->hash($cle);

        foreach ($this->buckets[$index] as $i => $paire) {
            if ($paire[0] === $cle) {
                array_splice($this->buckets[$index], $i, 1);
                $this->taille--;
                return true;
            }
        }
        return false;
    }

    public function contient(string $cle): bool
    {
        return $this->get($cle) !== null;
    }

    public function taille(): int { return $this->taille; }

    // ── Rehachage ───────────────────────────────────────────────
    // Déclenché quand facteur de charge > seuil — O(n)

    private function rehacher(): void
    {
        $anciensBuckets  = $this->buckets;
        $this->capacite *= 2;
        $this->buckets   = array_fill(0, $this->capacite, []);
        $this->taille    = 0;

        foreach ($anciensBuckets as $bucket) {
            foreach ($bucket as [$cle, $valeur]) {
                $this->put($cle, $valeur);
            }
        }
    }
}
```

### 5.3.2 Adressage ouvert (Open Addressing) — sondage linéaire

Au lieu de chaîner, on cherche la prochaine case libre dans le tableau.

```php
<?php
declare(strict_types=1);

class HashMapOuverte
{
    private array  $cles;
    private array  $valeurs;
    private array  $etats;    // 0=vide, 1=occupé, 2=supprimé(tombstone)
    private int    $capacite;
    private int    $taille = 0;

    private const VIDE       = 0;
    private const OCCUPE     = 1;
    private const SUPPRIME   = 2;

    public function __construct(int $capacite = 16)
    {
        $this->capacite = $capacite;
        $this->cles     = array_fill(0, $capacite, null);
        $this->valeurs  = array_fill(0, $capacite, null);
        $this->etats    = array_fill(0, $capacite, self::VIDE);
    }

    private function hash(string $cle): int
    {
        return crc32($cle) % $this->capacite;
    }

    /** Sondage linéaire — cherche la prochaine case disponible */
    public function put(string $cle, mixed $valeur): void
    {
        if ($this->taille / $this->capacite >= 0.5) {
            $this->rehacher();
        }

        $index = abs($this->hash($cle));

        while ($this->etats[$index] === self::OCCUPE && $this->cles[$index] !== $cle) {
            $index = ($index + 1) % $this->capacite;  // sondage linéaire
        }

        if ($this->etats[$index] !== self::OCCUPE) {
            $this->taille++;
        }
        $this->cles[$index]   = $cle;
        $this->valeurs[$index] = $valeur;
        $this->etats[$index]  = self::OCCUPE;
    }

    public function get(string $cle): mixed
    {
        $index = abs($this->hash($cle));

        while ($this->etats[$index] !== self::VIDE) {
            if ($this->etats[$index] === self::OCCUPE && $this->cles[$index] === $cle) {
                return $this->valeurs[$index];
            }
            $index = ($index + 1) % $this->capacite;
        }
        return null;
    }

    public function supprimer(string $cle): bool
    {
        $index = abs($this->hash($cle));

        while ($this->etats[$index] !== self::VIDE) {
            if ($this->etats[$index] === self::OCCUPE && $this->cles[$index] === $cle) {
                $this->etats[$index] = self::SUPPRIME;  // tombstone
                $this->taille--;
                return true;
            }
            $index = ($index + 1) % $this->capacite;
        }
        return false;
    }

    private function rehacher(): void
    {
        $anciennesCles    = $this->cles;
        $anciennesValeurs = $this->valeurs;
        $ancienEtats      = $this->etats;

        $this->capacite *= 2;
        $this->cles     = array_fill(0, $this->capacite, null);
        $this->valeurs  = array_fill(0, $this->capacite, null);
        $this->etats    = array_fill(0, $this->capacite, self::VIDE);
        $this->taille   = 0;

        for ($i = 0; $i < count($ancienEtats); $i++) {
            if ($ancienEtats[$i] === self::OCCUPE) {
                $this->put($anciennesCles[$i], $anciennesValeurs[$i]);
            }
        }
    }
}
```

---

## 5.4 L'array PHP comme HashMap natif

Le tableau associatif PHP est déjà une table de hachage très optimisée.

```php
<?php
declare(strict_types=1);

// ── Dictionnaire de mots ─────────────────────────────────────
$frequences = [];
$texte = "le chat et le chien et le chat";

foreach (explode(' ', $texte) as $mot) {
    $frequences[$mot] = ($frequences[$mot] ?? 0) + 1;
}
// ['le'=>3, 'chat'=>2, 'et'=>2, 'chien'=>1]

arsort($frequences);  // tri par fréquence décroissante
print_r($frequences);

// ── Cache mémoïsation ────────────────────────────────────────
$cache = [];
function fib(int $n, array &$cache): int
{
    if ($n <= 1) return $n;
    if (isset($cache[$n])) return $cache[$n];

    return $cache[$n] = fib($n - 1, $cache) + fib($n - 2, $cache);
}

// ── Ensemble (Set) ───────────────────────────────────────────
$ensemble = [];
$ensemble['alice']   = true;
$ensemble['bob']     = true;
$ensemble['charlie'] = true;

$contient = isset($ensemble['bob']);   // O(1)
unset($ensemble['bob']);               // O(1)
$tous = array_keys($ensemble);         // O(n)

// ── Comptage d'éléments uniques ──────────────────────────────
function compterUniques(array $arr): int
{
    return count(array_unique($arr));  // O(n log n) à cause du tri interne
}

// Meilleure version O(n) :
function compterUniquesFast(array $arr): int
{
    $vus = [];
    foreach ($arr as $v) $vus[$v] = true;
    return count($vus);
}
```

---

## 5.5 Algorithmes classiques avec HashMap

### 5.5.1 Deux sommes (Two Sum)

```php
<?php
declare(strict_types=1);

// O(n) — une seule passe
function deuxSommes(array $nums, int $cible): array
{
    $vus = [];
    foreach ($nums as $i => $v) {
        $complement = $cible - $v;
        if (isset($vus[$complement])) {
            return [$vus[$complement], $i];
        }
        $vus[$v] = $i;
    }
    return [];
}

echo implode(', ', deuxSommes([2, 7, 11, 15], 9)); // 0, 1
```

### 5.5.2 Détection d'anagrammes

```php
<?php
declare(strict_types=1);

function sontAnagrammes(string $a, string $b): bool
{
    if (strlen($a) !== strlen($b)) return false;

    $compte = [];
    for ($i = 0; $i < strlen($a); $i++) {
        $compte[$a[$i]] = ($compte[$a[$i]] ?? 0) + 1;
        $compte[$b[$i]] = ($compte[$b[$i]] ?? 0) - 1;
    }

    foreach ($compte as $c) {
        if ($c !== 0) return false;
    }
    return true;
}

var_dump(sontAnagrammes("listen", "silent"));  // true
var_dump(sontAnagrammes("hello", "world"));    // false
```

### 5.5.3 Sous-tableau de somme k

```php
<?php
declare(strict_types=1);

// Combien de sous-tableaux ont une somme égale à k ? O(n)
function compterSousTableaux(array $nums, int $k): int
{
    $compte    = 0;
    $somme     = 0;
    $prefixSum = [0 => 1];  // la somme 0 existe une fois (tableau vide)

    foreach ($nums as $v) {
        $somme  += $v;
        $compte += $prefixSum[$somme - $k] ?? 0;
        $prefixSum[$somme] = ($prefixSum[$somme] ?? 0) + 1;
    }
    return $compte;
}

echo compterSousTableaux([1, 1, 1], 2);     // 2
echo compterSousTableaux([1, 2, 3], 3);     // 2 ([1,2] et [3])
```

### 5.5.4 LRU Cache

```php
<?php
declare(strict_types=1);

/**
 * Cache LRU avec get/put en O(1).
 * HashMap + liste doublement chaînée.
 */
class LRUCache
{
    private array      $map;  // clé => nœud
    private ListeDouble $liste;
    private int        $capacite;

    public function __construct(int $capacite)
    {
        $this->capacite = $capacite;
        $this->map      = [];
        $this->liste    = new ListeDouble();
    }

    public function get(int $cle): int
    {
        if (!isset($this->map[$cle])) return -1;

        $noeud = $this->map[$cle];
        // Déplacer en tête (most recently used)
        $this->liste->supprimerNoeud($noeud);
        $this->liste->ajouterEnTete($noeud->valeur);
        $this->map[$cle] = $this->getTete();

        return $noeud->valeur[1];
    }

    public function put(int $cle, int $valeur): void
    {
        if (isset($this->map[$cle])) {
            $this->liste->supprimerNoeud($this->map[$cle]);
        } elseif ($this->liste->taille() === $this->capacite) {
            // Évincer le moins récemment utilisé (queue)
            $lru = $this->liste->supprimerEnQueue();
            unset($this->map[$lru[0]]);
        }

        $this->liste->ajouterEnTete([$cle, $valeur]);
        $this->map[$cle] = $this->getTete();
    }

    private function getTete(): NoeudDouble
    {
        // Accès au nœud tête de la liste
        // (implémentation simplifiée)
        return new NoeudDouble([0, 0]);
    }
}
```

---

## 5.6 Complexités récapitulatives

| Opération | Cas moyen | Pire cas (toutes collisions) |
|-----------|-----------|------------------------------|
| `put` | O(1) | O(n) |
| `get` | O(1) | O(n) |
| `delete` | O(1) | O(n) |
| `contains` | O(1) | O(n) |
| Itération | O(n) | O(n) |

!!! info "Facteur de charge"
    - **α < 0.75** : performances proches de O(1)
    - **α > 0.75** : les collisions augmentent, rehachage nécessaire
    - PHP redimensionne automatiquement ses arrays

---

## Exercices

!!! exercise "Exercice 5.1 — Premier caractère unique"
    Trouvez le premier caractère non répété d'une chaîne en O(n).
    ```
    "loveleetcode" → 'v'  (index 2)
    ```
    ??? success "Réponse"
        ```php
        function premierUnique(string $s): string
        {
            $compte = [];
            for ($i = 0; $i < strlen($s); $i++) {
                $compte[$s[$i]] = ($compte[$s[$i]] ?? 0) + 1;
            }
            for ($i = 0; $i < strlen($s); $i++) {
                if ($compte[$s[$i]] === 1) return $s[$i];
            }
            return '';
        }
        ```

!!! exercise "Exercice 5.2 — Grouper les anagrammes"
    Étant donné un tableau de mots, groupez les anagrammes ensemble.
    ```
    ["eat","tea","tan","ate","nat","bat"]
    → [["bat"],["nat","tan"],["ate","eat","tea"]]
    ```

!!! exercise "Exercice 5.3 — Sous-chaîne sans répétition"
    Trouvez la longueur de la plus longue sous-chaîne sans caractères répétés. O(n) avec fenêtre glissante + HashMap.
    ```
    "abcabcbb" → 3 ("abc")
    "pwwkew"   → 3 ("wke")
    ```
