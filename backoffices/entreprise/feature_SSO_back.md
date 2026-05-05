# 🔧 Connexion SSO

> Cette feature permet de se connecter en SSO à notre application Flutter et notre backoffice

---

## 📌 Contexte

Cette feature a été développée car plusieurs clients avaient comme prérequis la possibilité de se connecter en SSO à nos produits avant de signer des devis. L'intérêt étant de sécuriser la connexion des collaborateurs, nous avons décidé d'implémenter cette fonctionnalité, en utilisant WorkOS, sur notre application Ma Petite Planète et le back-office entreprise.

- **Projet** : Backoffice entreprise / App mobile
- **Équipe** : Développée en autonomie
- **Période** : Sprint de 2 semaines + 1/2 sprint pour la recherche

Pour cette feature, j'ai effectué la recherche sur la mise en place du SSO, l'implémentation du back nécessaire avec la connexion à WorkOS, ainsi que l'implémentation front pour l'application mobile.

Avant de pouvoir implémenter la connexion en SSO, nous avons, avec le product owner, modifié le processus d'inscription/connexion.

---

## 🎯 Objectif

L'objectif est de sécuriser la connexion pour les clients et de débloquer des participations au challenge.

---

## 🗺️ Flow / Structure

1. Pour pouvoir se connecter en passant par WorkOS, il nous faut récupérer un `connection_id`, fourni par WorkOS sur le dashboard. Afin de vérifier si l'utilisateur qui tente de se connecter doit utiliser le SSO, j'ai créé une table `sso_domains` qui mappe un domaine email à un `workos_connection_id`. Il faut donc, en premier lieu, renseigner les valeurs dans la table pour que la connexion fonctionne.
2. Lors de la connexion, l'utilisateur renseigne son adresse mail, ce qui déclenche une edge function.
3. La edge function reçoit l'email, un timestamp et une signature (dans le but de sécuriser l'appel à cette fonction cf. partie plus bas 🙂) et, avec ces valeurs, on va chercher le `connection_id` correspondant au domaine de l'email envoyé, s'il existe.
4. Si un `connection_id` existe, il est renvoyé côté front, dans notre application, où il est utilisé grâce à un provider WorkOS avec Supabase.
5. L'utilisateur est renvoyé vers le bon IdP afin de se connecter.
6. Une fois la connexion effectuée, l'utilisateur est renvoyé sur l'application et, dans les coulisses, WorkOS et Supabase ont effectué un échange de code pour créer une session côté Supabase. L'utilisateur est donc connecté et s'il n'a pas encore de compte, celui-ci est créé automatiquement dans la table `auth` de Supabase.

---

## 🏗️ Architecture & choix techniques

Cette vérification ayant lieu avant la connexion de l'utilisateur, l'endpoint est public (pas d'auth utilisateur), ce qui le rend potentiellement exposé à des abus ou à de l'énumération de domaines. Plusieurs mesures sont en place pour mitiger cela :

- **CORS restreint** — seules les origines autorisées peuvent appeler l'endpoint depuis un navigateur
- **HMAC** — garantit que la requête vient bien de notre frontend et non d'un client arbitraire ; le timestamp empêche les attaques par rejeu
- **Rate limiting à 10 req/min par IP** — limite l'énumération de domaines et les attaques par force brute
- **Rejet des providers publics** — pas de lookup SSO pour Gmail, Outlook, etc. (inutile et réduit la surface d'attaque)
- **Validation stricte de l'email** — format et longueur du domaine vérifiés avant toute requête en base
- **Réponse peu explicite** de la fonction, afin de ne pas divulguer si le `connection_id` existe ou non

Pour la partie rate limiting : j'ai créé une table `rate_limits` qui sert de compteur de requêtes par IP sur une fenêtre d'une minute. La fonction `increment_rate_limit(ip)` upserte le compteur et le remet à zéro si la fenêtre est expirée. Elle renvoie ensuite le nombre de tentatives enregistrées et si il y a 10 ou plus tentatives, on renvoie une réponse 429.

Enfin, pour ne pas conserver de données inutiles dans la table `rate_limits`, j'ai créé un cron `cleanup_expired_rate_limits`. Ce dernier tourne chaque nuit à minuit pour purger les lignes expirées et éviter que la table grossisse indéfiniment.

- **Stack utilisée** : Supabase Edge Functions, Flutter, WorkOS
- **Choix notables** : Le choix le plus structurant concerne la sécurité évoquée plus haut. L'endpoint ne pouvant pas être soumis à des vérifications d'identité classiques, il était important de renforcer au maximum les protections afin de restreindre les possibilités d'appel non autorisé. C'est pourquoi j'ai choisi de durcir la configuration CORS et d'ajouter une signature HMAC.

Un autre choix notable est celui d'utiliser WorkOS. Souhaitant proposer la connexion SSO à nos clients sans connaître à l'avance leur IdP, nous avons proposé, avec un collègue, d'utiliser WorkOS en tant que broker SSO, afin de répondre techniquement à plusieurs demandes clients de manière flexible. En prenant en compte les ressources à notre disposition (équipe et budget), nous avons conclu qu'implémenter une solution uniquement via Supabase aurait été plus fastidieux et potentiellement insuffisant pour couvrir l'ensemble des demandes clients.

---

## ⚠️ Difficultés rencontrées

L'une des difficultés rencontrées a été l'intégration dans notre CI/CD de toutes les valeurs attendues (liens de redirection, secret pour le HMAC, etc.). N'étant pas à l'origine de sa mise en place, il m'a fallu quelques tentatives avant que tout fonctionne correctement.

Une autre difficulté a été d'expliquer l'ensemble du cheminement et du fonctionnement du SSO à l'équipe. J'ai d'abord présenté le système de base sans mentionner WorkOS, en m'appuyant sur des schémas pour permettre à l'équipe de visualiser les interactions. Après avoir abordé la problématique des différents IdP, j'ai expliqué le choix de WorkOS, puis l'implémentation concrète. Cette passation était d'autant plus importante que je ne pourrais pas voir le résultat de cette fonctionnalité en production, quittant les effectifs peu après.

---

## ✅ Résultat

Certains clients ont signé un devis pour participer à des éditions alors qu'ils refusaient de le faire auparavant, le SSO étant un prérequis bloquant pour eux.