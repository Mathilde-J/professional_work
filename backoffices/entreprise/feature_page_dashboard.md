# 🔧 Page de Dashboard

> Page de dashboard qui permet aux porteurs de projets* et aux équipes internes d'avoir des visuels clairs sur les statistiques des joueurs du groupe.

*Les porteurs de projets sont les personnes, chez les clients, chargées de piloter le jeu au sein de leur structure.

---

## 🖼️ Visuels

![Image locale](/screenshots/dashboard_1.png)
![Image locale](/screenshots/dashboard_2.png)

---

## 📌 Contexte

- **Projet** : Backoffice entreprise
- **Équipe** : développée en autonomie
- **Période** : Sprint de 2 semaines en Octobre/Novembre 2025

Cette page de dashboard contient plusieurs graphiques afin de visualiser les statistiques générales de l'édition.

Dans une première section on trouve les chiffres de l'édition de la structure en elle-même :
- Le nombre de ligues
- Le nombre de joueurs inscrits
- Le nombre de défis validés
- Le nombre de défis par joueur

Dans la section suivante, on trouve les chiffres liés à l'impact écologique de l'édition :
- Le nombre de kg de CO2 évités
- Le nombre de litres d'eau économisés
- Le nombre de kg de déchets évités
- Le nombre d'heures de formation

On trouve ensuite les graphiques suivants :
- Le taux d'activité
- Les performances des défis personnalisables
- L'objectif de défis
- Le classement des défis préférés
- Le classement des défis les plus validés

Enfin, l'ensemble de la page est téléchargeable au format PDF ou PNG.

---

## 🎯 Objectif

Toutes ces données permettent aux utilisateurs (équipes internes et clients) de visualiser rapidement et facilement l'évolution d'une édition.

Pour les équipes internes, cela les aide à conseiller les clients sur la suite de l'édition et à identifier les actions à mener.

Pour les clients, cela leur permet non seulement d'avoir un appui visuel pour dynamiser et suivre l'évolution de l'édition, mais aussi de disposer de visuels en interne pour porter le projet auprès des différentes équipes, grâce au contenu exportable.

---

## 🗺️ Flow / Structure

1. L'utilisateur clique sur le lien "tableau de bord", on affiche la page de dashboard.
2. Le système déclenche le fetch des données dans un `useEffect`. Ce `useEffect` appelle les fonctions de fetch des différentes données les unes après les autres. Il se déclenche dès que l'id de l'édition ou du groupe est modifiée.
3. C'est le `StatsStore`, un store MobX, qui gère les différents appels à la base de données, en appelant les fonctions dans les fichiers Database, pour récupérer les données et met à jour le state du store. Si un des appels échoue, on conserve une erreur dans le state concernant la donnée en question et on affiche un message d'erreur dans l'encadré du graphique.
4. Les graphiques apparaissent les uns après les autres, ce qui était un prérequis pour cette page, sans pour autant bloquer toute l'exécution.

---

## 🏗️ Architecture & choix techniques

- **Stack utilisée** : Pour cette page, j'ai utilisé Recharts, une librairie React qui fournit des composants pour créer des graphiques avec de grandes possibilités de personnalisation.
- **Structure des données** :
  - Pour les données des stats du groupe, les stats d'impact et le graphique de taux d'activité : 3 RPC différentes lisent simplement une vue/table `group_edition_stats` déjà agrégée -> lecture directe avec calcul de moyenne à la volée.
  - Pour le graphique de l'objectif de défis : on calcule le nombre de défis à réaliser en fonction du nombre de défis proposés pour l'édition, le nombre total de joueurs et le nombre de défis déjà réalisés.
  - Pour le graphique des défis personnalisables : on retourne en un seul JSON les données jour par jour du graphique des défis "choix du public" (via `get_group_custom_challenges_data`), accompagnées du nombre de validations de la semaine en cours et d'un résumé par niveau de difficulté (easy / medium / hard / total).
  - Pour le graphique des défis les plus réalisés : on retourne le nombre de défis validés par catégorie, en ne comptant que les défis qui rapportent des points ou un boost, et en incluant les défis collectifs seulement s'ils ont été atteints.

- **Choix notable** : Lors du Backlog Refinement, les maquettes Figma étaient déjà prêtes. En discutant ensemble, nous avons conclu qu'il faudrait utiliser une librairie pour les graphiques (taux d'activité, performances des défis personnalisables et objectif des défis), l'enjeu étant de pouvoir les customiser pour qu'ils correspondent au design Figma.

  Nous avons donc créé un ticket dont le but était de choisir la librairie et de préparer chaque composant graphique avec de fausses données structurées. Une fois ce travail fait, il n'y avait plus qu'à brancher les appels back en suivant la structure de données établie. C'est moi qui ai effectué ce travail préparatoire, ainsi que la finalisation des graphiques taux d'activité, performances des défis (sauf la partie back), les statistiques du groupe et le classement des défis préférées.

  Pour choisir la librairie, j'ai d'abord fait des recherches pour identifier les noms qui revenaient souvent et vérifier que la customisation et la prise en main correspondaient à nos besoins. J'ai ensuite utilisé ChatGPT pour obtenir d'autres options et établir un comparatif. J'ai choisi Recharts car c'est une librairie connue, légère, bien customisable et régulièrement maintenue.

  Pour les statistiques du client en haut de page, les composants existaient déjà puisqu'ils faisaient parti du design system défini par les product owner. A partir de ce design system, nous avions déjà créé une librairie interne de composants. Pour les classements en bas, j'ai créé les composants (directement dans le repo du backoffice car ils n'ont pas vocation à être utilisé ailleurs), un pour les lignes et un pour l'encadré global.

---

## ⚠️ Difficultés rencontrées

- **Structure des données du graphique de performances des défis personnalisables** : la structure devait correspondre à celle attendue par le composant dela librairie. On voulait ajouter un tooltip apparaissant au survol des lignes du graphique, avec les autres lignes en transparence. J'ai expérimenté plusieurs solutions avec la librairie avant de trouver la bonne : il suffisait de gérer un state local pour récupérer le point survolé à l'entrée sur une ligne, et le remettre à `null` à la sortie.

---

## ✅ Résultat

- Le développement de cette page a été moins complexe que prévu grâce à la librairie qui était très flexible pour le design.
- Un autre développeur a pu finaliser facilement un des graphiques à partir du travail préparatoire.
- Des modifications de design ont pu être apportées récemment sans difficulté.

---

## 📎 Liens utiles *(optionnel)*