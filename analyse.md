# BobApp — Document d'analyse CI/CD

## 1. Présentation du projet

BobApp est une application de blagues du jour développée par Bob. 
Face à la croissance du nombre d'utilisateurs et aux difficultés 
de gestion manuelle des déploiements, une pipeline CI/CD complète 
a été mise en place pour automatiser les tests, l'analyse qualité 
et le déploiement.

## 2. Les étapes du workflow CI/CD

### Étape 1 — Tests et couverture back-end (Java / Spring Boot)

**Outil :** GitHub Actions + Maven + JaCoCo  
**Déclencheur :** chaque push ou pull request sur la branche main  
**Actions réalisées :**
- Compilation du projet avec Maven
- Exécution des tests unitaires
- Génération du rapport de couverture de code avec JaCoCo

### Étape 2 — Tests et couverture front-end (Angular)

**Outil :** GitHub Actions + Node.js + Karma  
**Actions réalisées :**
- Installation des dépendances npm
- Exécution des tests unitaires Angular
- Génération du rapport de couverture de code avec Karma

### Étape 3 — Analyse qualité du code (SonarCloud)

**Outil :** SonarCloud via GitHub Actions  
**Actions réalisées :**
- Analyse statique du code back-end
- Détection des bugs, code smells et Security Hotspots
- Génération du rapport qualité accessible sur SonarCloud

### Étape 4 — Déploiement des images Docker (Docker Hub)

**Outil :** GitHub Actions + Docker Hub  
**Condition :** uniquement si les étapes 1 et 2 sont validées  
**Actions réalisées :**
- Build de l'image Docker du back-end (Spring Boot)
- Build de l'image Docker du front-end (Angular)
- Push des deux images sur Docker Hub

## 3. KPI proposés

| KPI | Seuil minimum recommandé | Justification |
|-----|--------------------------|---------------|
| Taux de couverture de code (Coverage) | 80% | Garantir que la majorité du code est testée et limiter les régressions |
| Nouveaux bugs bloquants (New Blocker Issues) | 0 | Aucun bug critique ne doit être introduit dans le code |
| Security Hotspots résolus | 100% | Toutes les failles de sécurité détectées doivent être traitées |

## 4. Analyse des métriques SonarCloud

Après exécution de la pipeline, voici les métriques obtenues :

| Métrique | Valeur actuelle |
|----------|----------------|
| Security | A |
| Reliability | A |
| Maintainability | A |
| Coverage | < 80% (à améliorer) |
| Security Hotspots | 3 détectés |
| Duplications | 0% |

### Interprétation

- Les notes **A en Security, Reliability et Maintainability** sont positives 
  et indiquent un code globalement sain.
- Le **taux de couverture est insuffisant** — il y a très peu de tests 
  dans le projet. C'est le point prioritaire à améliorer.
- Les **3 Security Hotspots** détectés doivent être analysés et traités 
  par l'équipe de développement.

## 5. Analyse des avis utilisateurs

| Avis | Problème identifié | Priorité |
|------|-------------------|----------|
| "Impossible de poster une suggestion de blague, le bouton tourne et fait planter mon navigateur" | Bug front-end sur le bouton de suggestion | Haute |
| "J'ai remonté un bug sur le post de vidéo il y a deux semaines et il est encore présent" | Bug non corrigé sur le post de vidéo | Haute |
| "Ça fait une semaine que je ne reçois plus rien" | Problème de notifications ou de flux | Haute |
| "J'ai supprimé ce site de mes favoris" | Perte d'utilisateur due à la mauvaise qualité | Critique |

### Conclusion

Les avis utilisateurs révèlent des bugs critiques non corrigés. 
La mise en place de la CI/CD va permettre à Bob de :
- Détecter les régressions automatiquement avant chaque déploiement
- Réduire le temps de correction grâce aux rapports automatisés
- Encourager les contributions extérieures grâce à un pipeline fiable