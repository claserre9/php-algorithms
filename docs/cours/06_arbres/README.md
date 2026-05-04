# Chapitre 6 — Arbres binaires & BST

## 6.1 Terminologie

```
             10          ← racine (root)
           /    \
          5      15      ← nœuds internes
         / \    /  \
        3   7  12   20   ← feuilles (leaves)

profondeur de 7 = 2    (distance à la racine)
hauteur de l'arbre = 3 (nœuds sur le chemin le plus long)
degré d'un nœud = nombre d'enfants
```

| Terme | Définition |
|-------|-----------|
| Racine | Nœud sans parent |
| Feuille | Nœud sans enfant |
| Hauteur | Longueur du chemin le plus long racine → feuille |
| Profondeur | Distance d'un nœud à la racine |
| Arbre complet | Tous les niveaux remplis sauf peut-être le dernier |
| Arbre parfait | Tous les niveaux entièrement remplis |

---

## 6.2 Nœud et arbre binaire

```php
<?php
declare(strict_types=1);

class NoeudArbre
{
    public mixed        $valeur;
    public ?NoeudArbre  $gauche = null;
    public ?NoeudArbre  $droite = null;

    public function __construct(mixed $valeur)
    {
        $this->valeur = $valeur;
    }
}

class ArbreBinaire
{
    public ?NoeudArbre $racine = null;

    // ── Hauteur ─────────────────────────────────────────────────
    // O(n) — on visite chaque nœud
    public function hauteur(?NoeudArbre $noeud = null): int
    {
        $noeud ??= $this->racine;
        if ($noeud === null) return -1;

        return 1 + max(
            $this->hauteur($noeud->gauche),
            $this->hauteur($noeud->droite)
        );
    }

    // ── Nombre de nœuds ─────────────────────────────────────────
    public function compter(?NoeudArbre $noeud = null): int
    {
        $noeud ??= $this->racine;
        if ($noeud === null) return 0;

        return 1 + $this->compter($noeud->gauche) + $this->compter($noeud->droite);
    }

    // ── Somme de tous les nœuds ──────────────────────────────────
    public function somme(?NoeudArbre $noeud = null): int
    {
        $noeud ??= $this->racine;
        if ($noeud === null) return 0;

        return $noeud->valeur + $this->somme($noeud->gauche) + $this->somme($noeud->droite);
    }

    // ── Test d'équilibre ─────────────────────────────────────────
    // Un arbre est équilibré si |hauteur(gauche) - hauteur(droite)| <= 1
    public function estEquilibre(?NoeudArbre $noeud = null): bool
    {
        $noeud ??= $this->racine;
        return $this->hauteurEquilibre($noeud) !== -2;
    }

    private function hauteurEquilibre(?NoeudArbre $n): int
    {
        if ($n === null) return -1;

        $hg = $this->hauteurEquilibre($n->gauche);
        if ($hg === -2) return -2;

        $hd = $this->hauteurEquilibre($n->droite);
        if ($hd === -2) return -2;

        if (abs($hg - $hd) > 1) return -2;  // -2 = sentinelle "déséquilibré"
        return 1 + max($hg, $hd);
    }
}
```

---

## 6.3 Parcours d'arbres

```php
<?php
declare(strict_types=1);

class ParcourArbre
{
    // ── Parcours en profondeur (DFS) ────────────────────────────

    /** Préordre : Racine → Gauche → Droite */
    public static function preordre(?NoeudArbre $n): array
    {
        if ($n === null) return [];
        return [
            $n->valeur,
            ...self::preordre($n->gauche),
            ...self::preordre($n->droite),
        ];
    }

    /** Inordre : Gauche → Racine → Droite (donne le tri pour un BST) */
    public static function inordre(?NoeudArbre $n): array
    {
        if ($n === null) return [];
        return [
            ...self::inordre($n->gauche),
            $n->valeur,
            ...self::inordre($n->droite),
        ];
    }

    /** Postordre : Gauche → Droite → Racine */
    public static function postordre(?NoeudArbre $n): array
    {
        if ($n === null) return [];
        return [
            ...self::postordre($n->gauche),
            ...self::postordre($n->droite),
            $n->valeur,
        ];
    }

    // ── Parcours itératif (évite le stack overflow) ─────────────

    /** Inordre itératif avec une pile explicite */
    public static function inordreIteratif(?NoeudArbre $racine): array
    {
        $pile    = [];
        $resultat = [];
        $courant  = $racine;

        while ($courant !== null || !empty($pile)) {
            // Aller le plus à gauche possible
            while ($courant !== null) {
                $pile[]  = $courant;
                $courant = $courant->gauche;
            }
            // Traiter le nœud
            $courant     = array_pop($pile);
            $resultat[]  = $courant->valeur;
            $courant     = $courant->droite;
        }
        return $resultat;
    }

    // ── Parcours en largeur (BFS / Level-order) ─────────────────
    // O(n) temps et espace
    public static function largeur(?NoeudArbre $racine): array
    {
        if ($racine === null) return [];

        $queue    = new SplQueue();
        $resultat = [];
        $queue->enqueue($racine);

        while (!$queue->isEmpty()) {
            $noeud    = $queue->dequeue();
            $resultat[] = $noeud->valeur;

            if ($noeud->gauche !== null) $queue->enqueue($noeud->gauche);
            if ($noeud->droite !== null) $queue->enqueue($noeud->droite);
        }
        return $resultat;
    }

    /** Parcours par niveaux — retourne un tableau de tableaux */
    public static function parNiveaux(?NoeudArbre $racine): array
    {
        if ($racine === null) return [];

        $queue    = new SplQueue();
        $resultat = [];
        $queue->enqueue($racine);

        while (!$queue->isEmpty()) {
            $niveau = [];
            $taille = $queue->count();

            for ($i = 0; $i < $taille; $i++) {
                $noeud = $queue->dequeue();
                $niveau[] = $noeud->valeur;

                if ($noeud->gauche !== null) $queue->enqueue($noeud->gauche);
                if ($noeud->droite !== null) $queue->enqueue($noeud->droite);
            }
            $resultat[] = $niveau;
        }
        return $resultat;
    }
}
```

### Démonstration des parcours

```php
<?php
declare(strict_types=1);

//         10
//        /  \
//       5    15
//      / \     \
//     3   7    20

$racine           = new NoeudArbre(10);
$racine->gauche   = new NoeudArbre(5);
$racine->droite   = new NoeudArbre(15);
$racine->gauche->gauche = new NoeudArbre(3);
$racine->gauche->droite = new NoeudArbre(7);
$racine->droite->droite = new NoeudArbre(20);

echo implode(', ', ParcourArbre::preordre($racine));  // 10, 5, 3, 7, 15, 20
echo implode(', ', ParcourArbre::inordre($racine));   //  3, 5, 7, 10, 15, 20
echo implode(', ', ParcourArbre::postordre($racine)); //  3, 7, 5, 20, 15, 10
echo implode(', ', ParcourArbre::largeur($racine));   // 10, 5, 15, 3, 7, 20

print_r(ParcourArbre::parNiveaux($racine));
// [[10], [5, 15], [3, 7, 20]]
```

---

## 6.4 Arbre Binaire de Recherche (BST)

**Propriété BST** : pour tout nœud N,
- tous les nœuds du sous-arbre **gauche** ont une valeur **< N**
- tous les nœuds du sous-arbre **droit** ont une valeur **> N**

```php
<?php
declare(strict_types=1);

class BST
{
    private ?NoeudArbre $racine = null;

    // ── Insertion — O(log n) moy, O(n) pire ─────────────────────

    public function inserer(int $valeur): void
    {
        $this->racine = $this->insererRec($this->racine, $valeur);
    }

    private function insererRec(?NoeudArbre $noeud, int $valeur): NoeudArbre
    {
        if ($noeud === null) return new NoeudArbre($valeur);

        if ($valeur < $noeud->valeur) {
            $noeud->gauche = $this->insererRec($noeud->gauche, $valeur);
        } elseif ($valeur > $noeud->valeur) {
            $noeud->droite = $this->insererRec($noeud->droite, $valeur);
        }
        // Doublon ignoré
        return $noeud;
    }

    // ── Recherche — O(log n) moy, O(n) pire ─────────────────────

    public function rechercher(int $valeur): bool
    {
        return $this->rechercherRec($this->racine, $valeur);
    }

    private function rechercherRec(?NoeudArbre $n, int $v): bool
    {
        if ($n === null)        return false;
        if ($v === $n->valeur)  return true;
        if ($v < $n->valeur)    return $this->rechercherRec($n->gauche, $v);
        return                         $this->rechercherRec($n->droite, $v);
    }

    // Itératif — plus performant (pas de pile d'appel)
    public function rechercherIteratif(int $valeur): bool
    {
        $courant = $this->racine;
        while ($courant !== null) {
            if ($valeur === $courant->valeur) return true;
            $courant = $valeur < $courant->valeur ? $courant->gauche : $courant->droite;
        }
        return false;
    }

    // ── Suppression — O(log n) moy ───────────────────────────────

    public function supprimer(int $valeur): void
    {
        $this->racine = $this->supprimerRec($this->racine, $valeur);
    }

    private function supprimerRec(?NoeudArbre $n, int $v): ?NoeudArbre
    {
        if ($n === null) return null;

        if ($v < $n->valeur) {
            $n->gauche = $this->supprimerRec($n->gauche, $v);
        } elseif ($v > $n->valeur) {
            $n->droite = $this->supprimerRec($n->droite, $v);
        } else {
            // Nœud trouvé — 3 cas :
            if ($n->gauche === null) return $n->droite;  // cas 1 : pas d'enfant gauche
            if ($n->droite === null) return $n->gauche;  // cas 2 : pas d'enfant droit

            // Cas 3 : deux enfants → remplacer par le successeur in-ordre (min du sous-arbre droit)
            $successeur    = $this->minimum($n->droite);
            $n->valeur     = $successeur->valeur;
            $n->droite     = $this->supprimerRec($n->droite, $successeur->valeur);
        }
        return $n;
    }

    // ── Min / Max ────────────────────────────────────────────────

    public function minimum(?NoeudArbre $noeud = null): ?NoeudArbre
    {
        $noeud ??= $this->racine;
        while ($noeud?->gauche !== null) {
            $noeud = $noeud->gauche;
        }
        return $noeud;
    }

    public function maximum(?NoeudArbre $noeud = null): ?NoeudArbre
    {
        $noeud ??= $this->racine;
        while ($noeud?->droite !== null) {
            $noeud = $noeud->droite;
        }
        return $noeud;
    }

    // ── Validation BST ───────────────────────────────────────────

    public function estBST(): bool
    {
        return $this->validerBST($this->racine, PHP_INT_MIN, PHP_INT_MAX);
    }

    private function validerBST(?NoeudArbre $n, int $min, int $max): bool
    {
        if ($n === null) return true;
        if ($n->valeur <= $min || $n->valeur >= $max) return false;

        return $this->validerBST($n->gauche, $min, $n->valeur)
            && $this->validerBST($n->droite, $n->valeur, $max);
    }

    // ── Plus proche ancêtre commun (LCA) ─────────────────────────

    public function lca(int $p, int $q): ?int
    {
        return $this->lcaRec($this->racine, $p, $q)?->valeur;
    }

    private function lcaRec(?NoeudArbre $n, int $p, int $q): ?NoeudArbre
    {
        if ($n === null) return null;

        if ($p < $n->valeur && $q < $n->valeur) {
            return $this->lcaRec($n->gauche, $p, $q);
        }
        if ($p > $n->valeur && $q > $n->valeur) {
            return $this->lcaRec($n->droite, $p, $q);
        }
        return $n;  // un dans chaque sous-arbre = c'est le LCA
    }

    // ── Affichage ────────────────────────────────────────────────

    public function afficher(): void
    {
        $this->afficherRec($this->racine, '', true);
    }

    private function afficherRec(?NoeudArbre $n, string $prefix, bool $estDroite): void
    {
        if ($n === null) return;

        echo $prefix . ($estDroite ? "└── " : "├── ") . $n->valeur . "\n";
        $nouveauPrefix = $prefix . ($estDroite ? "    " : "│   ");
        $this->afficherRec($n->droite, $nouveauPrefix, false);
        $this->afficherRec($n->gauche, $nouveauPrefix, true);
    }
}
```

---

## 6.5 Complexités du BST

| Opération | Cas moyen | Pire cas (liste) |
|-----------|-----------|-----------------|
| Recherche | O(log n) | O(n) |
| Insertion | O(log n) | O(n) |
| Suppression | O(log n) | O(n) |
| Min / Max | O(log n) | O(n) |
| Inordre (tri) | O(n) | O(n) |

!!! warning "BST dégénéré"
    Si on insère des données déjà triées (1, 2, 3, 4...), le BST devient une liste chaînée et toutes les opérations passent en O(n). C'est pourquoi les arbres **AVL** et **rouge-noir** garantissent l'équilibre.

---

## 6.6 Arbre AVL (auto-équilibré)

Un **arbre AVL** maintient un facteur d'équilibre `[-1, 0, 1]` à chaque nœud. Dès qu'une insertion/suppression déséquilibre, une **rotation** rééquilibre en O(1).

```php
<?php
declare(strict_types=1);

class NoeudAVL
{
    public int      $valeur;
    public int      $hauteur  = 1;
    public ?NoeudAVL $gauche  = null;
    public ?NoeudAVL $droite  = null;

    public function __construct(int $valeur) { $this->valeur = $valeur; }
}

class AVL
{
    private ?NoeudAVL $racine = null;

    private function hauteur(?NoeudAVL $n): int
    {
        return $n?->hauteur ?? 0;
    }

    private function facteurEquilibre(NoeudAVL $n): int
    {
        return $this->hauteur($n->gauche) - $this->hauteur($n->droite);
    }

    private function majHauteur(NoeudAVL $n): void
    {
        $n->hauteur = 1 + max($this->hauteur($n->gauche), $this->hauteur($n->droite));
    }

    // ── Rotations ────────────────────────────────────────────────

    /**
     *   y            x
     *  / \          / \
     * x   T3  →  T1   y
     * / \              / \
     * T1 T2           T2  T3
     * Rotation droite (cas gauche-gauche)
     */
    private function rotationDroite(NoeudAVL $y): NoeudAVL
    {
        $x    = $y->gauche;
        $T2   = $x->droite;

        $x->droite = $y;
        $y->gauche = $T2;

        $this->majHauteur($y);
        $this->majHauteur($x);

        return $x;  // nouvelle racine
    }

    /**
     *   x                y
     *  / \              / \
     * T1   y    →      x   T3
     *     / \         / \
     *    T2  T3      T1  T2
     * Rotation gauche (cas droite-droite)
     */
    private function rotationGauche(NoeudAVL $x): NoeudAVL
    {
        $y    = $x->droite;
        $T2   = $y->gauche;

        $y->gauche  = $x;
        $x->droite  = $T2;

        $this->majHauteur($x);
        $this->majHauteur($y);

        return $y;
    }

    private function equilibrer(NoeudAVL $n): NoeudAVL
    {
        $this->majHauteur($n);
        $fe = $this->facteurEquilibre($n);

        // Cas gauche-gauche
        if ($fe > 1 && $this->facteurEquilibre($n->gauche) >= 0) {
            return $this->rotationDroite($n);
        }
        // Cas gauche-droite
        if ($fe > 1 && $this->facteurEquilibre($n->gauche) < 0) {
            $n->gauche = $this->rotationGauche($n->gauche);
            return $this->rotationDroite($n);
        }
        // Cas droite-droite
        if ($fe < -1 && $this->facteurEquilibre($n->droite) <= 0) {
            return $this->rotationGauche($n);
        }
        // Cas droite-gauche
        if ($fe < -1 && $this->facteurEquilibre($n->droite) > 0) {
            $n->droite = $this->rotationDroite($n->droite);
            return $this->rotationGauche($n);
        }

        return $n;  // déjà équilibré
    }

    public function inserer(int $valeur): void
    {
        $this->racine = $this->insererRec($this->racine, $valeur);
    }

    private function insererRec(?NoeudAVL $n, int $v): NoeudAVL
    {
        if ($n === null) return new NoeudAVL($v);

        if ($v < $n->valeur)      $n->gauche = $this->insererRec($n->gauche, $v);
        elseif ($v > $n->valeur)  $n->droite = $this->insererRec($n->droite, $v);
        else return $n;  // doublon

        return $this->equilibrer($n);
    }
}
```

---

## Exercices

!!! exercise "Exercice 6.1 — Vue de droite"
    Retournez les valeurs visibles depuis la droite (dernier nœud de chaque niveau).
    ```
    Arbre:        10
                 /  \
                5    15
               / \
              3   7
    → [10, 15, 7]
    ```

!!! exercise "Exercice 6.2 — Chemin de somme maximale"
    Trouvez la somme maximale d'un chemin racine → feuille dans un arbre binaire.

!!! exercise "Exercice 6.3 — Sérialisation / Désérialisation"
    Implémentez `serialiser(ArbreBinaire): string` et `deserialiser(string): ArbreBinaire` pour sauvegarder et restaurer un arbre.
