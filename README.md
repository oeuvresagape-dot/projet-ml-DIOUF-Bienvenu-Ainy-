# Projet 2 — Détecter des attaques réseau avec le Machine Learning

**Nom / Prénom :** DIOUF Bienvenu Ainy

## Utilisation
```bash
pip install -r requirements.txt
python src/train.py      # compare les modèles, sauvegarde models/model.joblib
python src/predict.py    # exemple de prédiction
```

## Données et méthode
- 3000 flux, 9 variables, cible `attaque` (27,8 % d'attaques → classes déséquilibrées).
- Seule `duration` a des valeurs manquantes (90, soit 3 %) → imputation par la médiane.
- Pipeline : imputation + `StandardScaler` (variables numériques), imputation + `OneHotEncoder` (`protocol`), puis classifieur.
- Validation : séparation 75 % / 25 % **stratifiée** (`random_state=42`), évaluation sur la classe « attaque ».
- Modèles à classes pondérées (`class_weight="balanced"`) quand c'est possible.

## Résultats

**Nom / Prénom :** DIOUF Bienvenu Ainy

Jeu de validation : 750 flux (208 attaques).

| Modèle | F1 (attaque) | Rappel (attaque) | Précision (attaque) |
|---|---|---|---|
| Régression logistique | 0,919 | 0,899 | 0,940 |
| Arbre de décision (profondeur 6) | 0,959 | 0,957 | 0,961 |
| **Forêt aléatoire (300 arbres)** | **0,973** | **0,971** | **0,976** |
| Gradient Boosting | 0,973 | 0,966 | 0,980 |

Matrices de confusion (validation) :

| Modèle | Vrais négatifs | Faux positifs | Faux négatifs | Vrais positifs |
|---|---|---|---|---|
| Régression logistique | 530 | 12 | 21 | 187 |
| Arbre de décision | 534 | 8 | 9 | 199 |
| Forêt aléatoire | 537 | 5 | 6 | 202 |
| Gradient Boosting | 538 | 4 | 7 | 201 |

**Modèle retenu et pourquoi :** la **forêt aléatoire**. Elle a le meilleur F1 (0,973, à égalité avec le Gradient Boosting) et surtout le meilleur **rappel** (0,971), le plus important en sécurité. Les modèles à base d'arbres captent des règles non linéaires (ex. beaucoup d'échecs de login sur le port 22/23, `syn_rate` élevé avec beaucoup de connexions vers le même hôte) que la régression logistique, linéaire, ne représente pas bien. Le modèle final est ré-entraîné sur les 3000 flux et sauvegardé dans `models/model.joblib` (Pipeline complet).

**Analyse des erreurs (faux négatifs / faux positifs) :** la forêt aléatoire rate **6 attaques sur 208** (faux négatifs, ≈ 2,9 %) et déclenche **5 fausses alertes** sur 542 flux normaux (faux positifs, ≈ 0,9 %). Le faux négatif est le **pire cas** en sécurité : une attaque non détectée passe inaperçue (intrusion, vol de données, indisponibilité du service) et le coût est élevé, alors qu'une fausse alerte coûte seulement un peu de temps d'analyse. Les attaques manquées ressemblent probablement à du trafic normal (peu de paquets, faible `syn_rate`, ex. force brute lente). Pistes d'amélioration : abaisser le seuil de décision pour augmenter le rappel, ajouter des variables (ratio `src_bytes`/`dst_bytes`, octets par paquet), régler les hyperparamètres par validation croisée.
