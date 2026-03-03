---
title: "Rust pour l'ingénieur IA : au-delà du hype"
date: 2025-12-05
draft: false
tags: ["Rust", "Performance", "Python", "HPC", "Production"]
summary: "Python pour prototyper, Rust pour déployer. Retour pragmatique sur l'adoption de Rust dans un contexte d'IA industrielle."
---

## Python n'est pas le problème

Soyons clairs : Python reste le meilleur choix pour le prototypage en IA. L'écosystème (PyTorch, NumPy, scikit-learn) est inégalé, et la vélocité de développement est imbattable pour l'expérimentation.

Le problème arrive quand le prototype doit passer en production avec des contraintes de performance, de mémoire ou de fiabilité.

## Pourquoi Rust

J'ai migré un modèle de vision par ordinateur de MATLAB vers C++/CUDA il y a quelques années. Le gain de performance était spectaculaire (×50), mais le prix à payer en complexité de code, en gestion mémoire et en bugs subtils était élevé.

Rust offre le même niveau de performance que C++ avec trois avantages décisifs :

### 1. Sécurité mémoire à la compilation

Pas de segfault, pas de use-after-free, pas de data race. Le borrow checker est exigeant, mais il élimine des classes entières de bugs *avant* l'exécution.

```rust
// Le compilateur refuse ce code : emprunt mutable + immutable simultané
let mut data = vec![1, 2, 3];
let ref_data = &data;
data.push(4); // Erreur de compilation
println!("{:?}", ref_data);
```

### 2. Concurrence sans terreur

En C++, la programmation concurrente est un champ de mines. En Rust, le système de types garantit l'absence de data races à la compilation. On peut paralléliser agressivement sans craindre les bugs intermittents.

### 3. Interopérabilité Python via PyO3

On n'a pas besoin de tout réécrire. PyO3 permet d'exposer des fonctions Rust comme des modules Python natifs :

```rust
use pyo3::prelude::*;

#[pyfunction]
fn process_batch(data: Vec<f64>) -> Vec<f64> {
    data.iter().map(|x| x * x + 1.0).collect()
}

#[pymodule]
fn my_module(m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(process_batch, m)?)?;
    Ok(())
}
```

Côté Python, c'est transparent :

```python
import my_module
result = my_module.process_batch([1.0, 2.0, 3.0])
```

## Cas d'usage concrets

| Contexte | Avant | Après (Rust) |
|---|---|---|
| Moteur de règles ontologiques | Python, 2.3s/requête | Rust, 85ms/requête |
| Parsing de documents techniques | Python, 450 docs/min | Rust, 8 200 docs/min |
| Inférence de graphe (custom) | Python/NetworkX | Rust/petgraph, ×35 |

## Quand ne pas utiliser Rust

- **Prototypage et expérimentation** : Python reste plus rapide à écrire
- **Scripts one-shot** : le overhead du système de types ne se justifie pas
- **Écosystème ML pur** : PyTorch n'a pas d'équivalent en Rust (pour l'instant)

La stratégie qui fonctionne : prototyper en Python, identifier les goulots d'étranglement, réécrire ces composants en Rust et les exposer via PyO3.

## Pour commencer

Si vous venez de Python, le choc initial est réel. Quelques ressources qui m'ont aidé :

- **The Rust Book** : incontournable, lisez-le en entier
- **Rust by Example** : pour les impatients
- **PyO3 User Guide** : pour l'intégration Python

Le temps investi dans l'apprentissage de Rust se rembourse dès le premier composant critique déployé en production.

---

*Un futur article détaillera l'utilisation de Rust + CUDA pour le traitement d'image haute performance.*
