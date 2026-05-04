# Chapitre 4 — Listes chaînées

## 4.1 Concept

Une **liste chaînée** est une suite de nœuds reliés par des pointeurs. Contrairement aux tableaux, les éléments ne sont pas contigus en mémoire : chaque nœud contient la donnée et une référence vers le nœud suivant.

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  val: 10 │───▶│  val: 20 │───▶│  val: 30 │───▶│  val: 40 │───▶ null
└──────────┘    └──────────┘    └──────────┘    └──────────┘
    head
```

### Comparaison avec les tableaux

| Opération | Tableau | Liste chaînée |
|-----------|---------|---------------|
| Accès par index | O(1) | O(n) |
| Insertion en tête | O(n) | O(1) |
| Insertion en queue | O(1)* | O(n) ou O(1)** |
| Insertion au milieu | O(n) | O(1) avec pointeur |
| Suppression en tête | O(n) | O(1) |
| Recherche | O(n) | O(n) |
| Mémoire | Contigu | Fragmenté (+pointeur) |

*amorti ; **avec pointeur `tail`

---

## 4.2 Liste simplement chaînée

```php
<?php
declare(strict_types=1);

class Noeud
{
    public mixed $valeur;
    public ?Noeud $suivant = null;

    public function __construct(mixed $valeur)
    {
        $this->valeur = $valeur;
    }
}

class ListeSimple
{
    private ?Noeud $tete  = null;
    private ?Noeud $queue = null;
    private int    $taille = 0;

    // ── Taille ──────────────────────────────────────────────────

    public function taille(): int    { return $this->taille; }
    public function estVide(): bool  { return $this->tete === null; }

    // ── Ajout ───────────────────────────────────────────────────

    /** Insertion en tête — O(1) */
    public function ajouterEnTete(mixed $valeur): void
    {
        $noeud          = new Noeud($valeur);
        $noeud->suivant = $this->tete;
        $this->tete     = $noeud;

        if ($this->queue === null) {
            $this->queue = $noeud;
        }
        $this->taille++;
    }

    /** Insertion en queue — O(1) grâce au pointeur tail */
    public function ajouterEnQueue(mixed $valeur): void
    {
        $noeud = new Noeud($valeur);

        if ($this->queue === null) {
            $this->tete  = $noeud;
            $this->queue = $noeud;
        } else {
            $this->queue->suivant = $noeud;
            $this->queue          = $noeud;
        }
        $this->taille++;
    }

    /** Insertion à l'index i — O(n) */
    public function insererA(int $index, mixed $valeur): void
    {
        if ($index < 0 || $index > $this->taille) {
            throw new OutOfRangeException("Index $index invalide");
        }
        if ($index === 0) {
            $this->ajouterEnTete($valeur);
            return;
        }
        if ($index === $this->taille) {
            $this->ajouterEnQueue($valeur);
            return;
        }

        $courant = $this->tete;
        for ($i = 0; $i < $index - 1; $i++) {
            $courant = $courant->suivant;
        }
        $noeud          = new Noeud($valeur);
        $noeud->suivant = $courant->suivant;
        $courant->suivant = $noeud;
        $this->taille++;
    }

    // ── Suppression ─────────────────────────────────────────────

    /** Suppression en tête — O(1) */
    public function supprimerEnTete(): mixed
    {
        if ($this->estVide()) throw new UnderflowException("Liste vide");

        $valeur     = $this->tete->valeur;
        $this->tete = $this->tete->suivant;
        if ($this->tete === null) $this->queue = null;
        $this->taille--;
        return $valeur;
    }

    /** Suppression d'un nœud par valeur — O(n) */
    public function supprimer(mixed $valeur): bool
    {
        if ($this->estVide()) return false;

        // Cas : nœud en tête
        if ($this->tete->valeur === $valeur) {
            $this->supprimerEnTete();
            return true;
        }

        $courant = $this->tete;
        while ($courant->suivant !== null) {
            if ($courant->suivant->valeur === $valeur) {
                if ($courant->suivant === $this->queue) {
                    $this->queue = $courant;
                }
                $courant->suivant = $courant->suivant->suivant;
                $this->taille--;
                return true;
            }
            $courant = $courant->suivant;
        }
        return false;
    }

    // ── Accès ───────────────────────────────────────────────────

    /** Accès par index — O(n) */
    public function obtenir(int $index): mixed
    {
        $this->verifierIndex($index);
        $courant = $this->tete;
        for ($i = 0; $i < $index; $i++) {
            $courant = $courant->suivant;
        }
        return $courant->valeur;
    }

    // ── Recherche ───────────────────────────────────────────────

    public function contient(mixed $valeur): bool
    {
        $courant = $this->tete;
        while ($courant !== null) {
            if ($courant->valeur === $valeur) return true;
            $courant = $courant->suivant;
        }
        return false;
    }

    // ── Transformation ──────────────────────────────────────────

    /** Inversion en place — O(n) */
    public function inverser(): void
    {
        $precedent = null;
        $courant   = $this->tete;
        $this->queue = $this->tete;

        while ($courant !== null) {
            $suivant          = $courant->suivant;
            $courant->suivant = $precedent;
            $precedent        = $courant;
            $courant          = $suivant;
        }
        $this->tete = $precedent;
    }

    /** Conversion en tableau — O(n) */
    public function versTableau(): array
    {
        $arr     = [];
        $courant = $this->tete;
        while ($courant !== null) {
            $arr[]   = $courant->valeur;
            $courant = $courant->suivant;
        }
        return $arr;
    }

    public function __toString(): string
    {
        return implode(' → ', $this->versTableau()) . ' → null';
    }

    private function verifierIndex(int $index): void
    {
        if ($index < 0 || $index >= $this->taille) {
            throw new OutOfRangeException("Index $index invalide");
        }
    }
}
```

---

## 4.3 Démonstration

```php
<?php
declare(strict_types=1);

$liste = new ListeSimple();

$liste->ajouterEnQueue(10);
$liste->ajouterEnQueue(20);
$liste->ajouterEnQueue(30);
$liste->ajouterEnTete(5);
echo $liste; // 5 → 10 → 20 → 30 → null

$liste->insererA(2, 15);
echo $liste; // 5 → 10 → 15 → 20 → 30 → null

$liste->supprimer(15);
echo $liste; // 5 → 10 → 20 → 30 → null

$liste->inverser();
echo $liste; // 30 → 20 → 10 → 5 → null

echo $liste->obtenir(1); // 20
```

---

## 4.4 Liste doublement chaînée

Chaque nœud a un pointeur `suivant` ET un pointeur `precedent`.

```
null ◀── ┌──────┐ ◀──▶ ┌──────┐ ◀──▶ ┌──────┐ ──▶ null
         │  10  │      │  20  │      │  30  │
         └──────┘      └──────┘      └──────┘
           head                       tail
```

```php
<?php
declare(strict_types=1);

class NoeudDouble
{
    public mixed        $valeur;
    public ?NoeudDouble $suivant   = null;
    public ?NoeudDouble $precedent = null;

    public function __construct(mixed $valeur)
    {
        $this->valeur = $valeur;
    }
}

class ListeDouble
{
    private ?NoeudDouble $tete  = null;
    private ?NoeudDouble $queue = null;
    private int          $taille = 0;

    public function taille(): int   { return $this->taille; }
    public function estVide(): bool { return $this->tete === null; }

    // ── Ajout ───────────────────────────────────────────────────

    public function ajouterEnTete(mixed $valeur): void
    {
        $noeud = new NoeudDouble($valeur);

        if ($this->tete === null) {
            $this->tete  = $noeud;
            $this->queue = $noeud;
        } else {
            $noeud->suivant      = $this->tete;
            $this->tete->precedent = $noeud;
            $this->tete          = $noeud;
        }
        $this->taille++;
    }

    public function ajouterEnQueue(mixed $valeur): void
    {
        $noeud = new NoeudDouble($valeur);

        if ($this->queue === null) {
            $this->tete  = $noeud;
            $this->queue = $noeud;
        } else {
            $this->queue->suivant = $noeud;
            $noeud->precedent     = $this->queue;
            $this->queue          = $noeud;
        }
        $this->taille++;
    }

    // ── Suppression ─────────────────────────────────────────────

    public function supprimerEnTete(): mixed
    {
        if ($this->estVide()) throw new UnderflowException("Liste vide");

        $valeur = $this->tete->valeur;
        $this->tete = $this->tete->suivant;

        if ($this->tete !== null) {
            $this->tete->precedent = null;
        } else {
            $this->queue = null;
        }
        $this->taille--;
        return $valeur;
    }

    public function supprimerEnQueue(): mixed
    {
        if ($this->estVide()) throw new UnderflowException("Liste vide");

        $valeur = $this->queue->valeur;
        $this->queue = $this->queue->precedent;

        if ($this->queue !== null) {
            $this->queue->suivant = null;
        } else {
            $this->tete = null;
        }
        $this->taille--;
        return $valeur;
    }

    public function supprimerNoeud(NoeudDouble $noeud): void
    {
        if ($noeud->precedent !== null) {
            $noeud->precedent->suivant = $noeud->suivant;
        } else {
            $this->tete = $noeud->suivant;
        }

        if ($noeud->suivant !== null) {
            $noeud->suivant->precedent = $noeud->precedent;
        } else {
            $this->queue = $noeud->precedent;
        }
        $this->taille--;
    }

    public function __toString(): string
    {
        $parts   = [];
        $courant = $this->tete;
        while ($courant !== null) {
            $parts[] = $courant->valeur;
            $courant = $courant->suivant;
        }
        return 'null ⟵ ' . implode(' ⟺ ', $parts) . ' ⟶ null';
    }
}
```

---

## 4.5 Liste circulaire

Le dernier nœud pointe vers le premier. Utile pour les tourniquet, planificateurs (round-robin), etc.

```php
<?php
declare(strict_types=1);

class ListeCirculaire
{
    private ?Noeud $queue  = null;  // on garde un pointeur sur la queue
    private int    $taille = 0;

    /** Ajout en queue — O(1). La queue.suivant pointe sur la tête. */
    public function ajouter(mixed $valeur): void
    {
        $noeud = new Noeud($valeur);

        if ($this->queue === null) {
            $noeud->suivant = $noeud;    // pointe sur lui-même
        } else {
            $noeud->suivant       = $this->queue->suivant;  // nouveau → tête
            $this->queue->suivant = $noeud;                 // ancienne queue → nouveau
        }
        $this->queue = $noeud;
        $this->taille++;
    }

    /** Supprime la tête (premier entré) — O(1) */
    public function supprimerTete(): mixed
    {
        if ($this->queue === null) throw new UnderflowException("Liste vide");

        $tete = $this->queue->suivant;
        $valeur = $tete->valeur;

        if ($tete === $this->queue) {
            $this->queue = null;                            // un seul élément
        } else {
            $this->queue->suivant = $tete->suivant;        // queue → nouveau_tête
        }
        $this->taille--;
        return $valeur;
    }

    /** Rotation — avance la tête d'un cran, O(1) */
    public function tourner(): void
    {
        if ($this->queue !== null) {
            $this->queue = $this->queue->suivant;
        }
    }

    public function afficher(): void
    {
        if ($this->queue === null) { echo "[vide]\n"; return; }

        $courant = $this->queue->suivant;  // tête
        $parts = [];
        do {
            $parts[] = $courant->valeur;
            $courant = $courant->suivant;
        } while ($courant !== $this->queue->suivant);

        echo implode(' → ', $parts) . " → (retour tête)\n";
    }
}
```

---

## 4.6 Algorithmes classiques sur les listes chaînées

```php
<?php
declare(strict_types=1);

// ── Détecter un cycle (algorithme de Floyd) ─────────────────
// Complexité : O(n) temps, O(1) espace
function detecterCycle(?Noeud $tete): bool
{
    $lent   = $tete;
    $rapide = $tete;

    while ($rapide !== null && $rapide->suivant !== null) {
        $lent   = $lent->suivant;           // avance d'1
        $rapide = $rapide->suivant->suivant; // avance de 2

        if ($lent === $rapide) return true;  // cycle détecté
    }
    return false;
}

// ── Trouver le milieu d'une liste ───────────────────────────
// O(n) — tortue et lièvre
function milieu(?Noeud $tete): ?Noeud
{
    $lent   = $tete;
    $rapide = $tete;

    while ($rapide !== null && $rapide->suivant !== null) {
        $lent   = $lent->suivant;
        $rapide = $rapide->suivant->suivant;
    }
    return $lent;
}

// ── Fusionner deux listes triées ────────────────────────────
// O(n + m)
function fusionnerTriees(?Noeud $l1, ?Noeud $l2): ?Noeud
{
    $factice   = new Noeud(0);  // nœud sentinelle
    $courant   = $factice;

    while ($l1 !== null && $l2 !== null) {
        if ($l1->valeur <= $l2->valeur) {
            $courant->suivant = $l1;
            $l1               = $l1->suivant;
        } else {
            $courant->suivant = $l2;
            $l2               = $l2->suivant;
        }
        $courant = $courant->suivant;
    }

    $courant->suivant = $l1 ?? $l2;
    return $factice->suivant;
}

// ── Le k-ième nœud depuis la fin ────────────────────────────
// O(n) — deux pointeurs
function kiemeDepuisLaFin(?Noeud $tete, int $k): mixed
{
    $avance = $tete;
    $retard = $tete;

    // Avancer le premier pointeur de k pas
    for ($i = 0; $i < $k; $i++) {
        if ($avance === null) return null;
        $avance = $avance->suivant;
    }

    // Avancer les deux jusqu'à la fin
    while ($avance !== null) {
        $avance = $avance->suivant;
        $retard = $retard->suivant;
    }
    return $retard?->valeur;
}

// ── Retirer les doublons d'une liste triée ──────────────────
// O(n), O(1) espace
function supprimerDoublons(?Noeud $tete): ?Noeud
{
    $courant = $tete;
    while ($courant !== null && $courant->suivant !== null) {
        if ($courant->valeur === $courant->suivant->valeur) {
            $courant->suivant = $courant->suivant->suivant;
        } else {
            $courant = $courant->suivant;
        }
    }
    return $tete;
}

// ── Vérifier si une liste est palindrome ────────────────────
// O(n) temps, O(1) espace : trouve le milieu, inverse la moitié droite, compare
function estPalindrome(?Noeud $tete): bool
{
    if ($tete === null || $tete->suivant === null) return true;

    // 1. Trouver le milieu
    $milieu = milieu($tete);

    // 2. Inverser la seconde moitié
    $secondeMotie = inverserListe($milieu);
    $copie        = $secondeMotie;

    // 3. Comparer
    $resultat = true;
    $p1 = $tete;
    while ($secondeMotie !== null) {
        if ($p1->valeur !== $secondeMotie->valeur) {
            $resultat = false;
            break;
        }
        $p1           = $p1->suivant;
        $secondeMotie = $secondeMotie->suivant;
    }

    // 4. Restaurer (optionnel)
    inverserListe($copie);
    return $resultat;
}

function inverserListe(?Noeud $tete): ?Noeud
{
    $precedent = null;
    $courant   = $tete;
    while ($courant !== null) {
        $suivant          = $courant->suivant;
        $courant->suivant = $precedent;
        $precedent        = $courant;
        $courant          = $suivant;
    }
    return $precedent;
}
```

---

## 4.7 Comparaison des types de listes

| | Simplement chaînée | Doublement chaînée | Circulaire |
|--|---|---|---|
| Mémoire par nœud | data + 1 ptr | data + 2 ptr | data + 1 ptr |
| Insertion tête | O(1) | O(1) | O(1) |
| Insertion queue | O(1)* | O(1)* | O(1)* |
| Suppression tête | O(1) | O(1) | O(1) |
| Suppression queue | O(n) | O(1) | O(n) |
| Parcours arrière | Non | Oui | Non |
| Usage typique | Stack, Queue | Deque, LRU cache | Round-robin |

*avec pointeur `tail`

---

## Exercices

!!! exercise "Exercice 4.1 — Inversion par groupes"
    Inversez une liste chaînée par groupes de k nœuds.
    ```
    [1→2→3→4→5→6], k=2  →  [2→1→4→3→6→5]
    [1→2→3→4→5→6], k=3  →  [3→2→1→6→5→4]
    ```

!!! exercise "Exercice 4.2 — LRU Cache"
    Implémentez un cache LRU (Least Recently Used) de capacité n avec `get` et `put` en O(1). Utilisez une liste doublement chaînée + une table de hachage.

!!! exercise "Exercice 4.3 — Tri par fusion sur liste chaînée"
    Implémentez le tri par fusion directement sur une liste simplement chaînée (sans la convertir en tableau). Complexité : O(n log n) temps, O(log n) espace (pile de récursion).
