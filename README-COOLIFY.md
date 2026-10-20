# Déployer Postiz sur Coolify

Configuration dédiée à **Postiz v2.25.1** pour **https://postiz.oraclelumira.com**.

La configuration officielle `docker-compose.yaml` reste intacte. Utilisez **`docker-compose.coolify.yml`**, basé sur les services du Compose officiel et adapté aux déploiements Git de Coolify.

## Avant le déploiement

- Le serveur possède suffisamment de mémoire : Postiz documente un minimum de 2 Go sur un serveur dédié léger et recommande 8 Go pour une petite équipe ; si d'autres applications fonctionnent sur le serveur, vérifier la RAM disponible (`free -h`), l'espace disque (`df -h`) et la charge avant de lancer Temporal + Elasticsearch.
- Le DNS A `postiz.oraclelumira.com` doit pointer sur l'adresse IPv4 publique du serveur Coolify. Corriger tout AAAA incorrect.
- Le proxy Traefik de Coolify doit être opérationnel sur les ports 80/443.

## Créer une application depuis Git

1. Dans le projet Coolify vide : **+ New > Application > Public Repository**.
2. URL du dépôt : `https://github.com/tachfineamnay/postiz-docker-compose`.
3. Branche : `main`. Build Pack / Build Strategy : **Docker Compose**.
4. Base Directory : `/`. Docker Compose Location : `/docker-compose.coolify.yml` (ou `docker-compose.coolify.yml` si le champ attend un chemin relatif).
5. Enregistrer, puis vérifier les 6 services analysés et les volumes.
6. Dans **Domains**, associer **uniquement** le service `postiz` au domaine HTTPS `postiz.oraclelumira.com`, avec port interne **5000** et chemin **/**. Suivant le formulaire, la valeur affichée pourra être `https://postiz.oraclelumira.com:5000` : le suffixe est le port interne, et le navigateur ouvre **https://postiz.oraclelumira.com** sans port.
7. Dans **Environment Variables**, laisser Coolify générer automatiquement les 4 valeurs : `SERVICE_HEX_64_POSTIZJWT`, `SERVICE_PASSWORD_64_POSTIZDB`, `SERVICE_PASSWORD_64_POSTIZREDIS`, `SERVICE_PASSWORD_64_TEMPORALDB`. Vérifier qu'aucune valeur n'est vide. Ne pas recopier les secrets en clair dans GitHub.
8. Lancer **Deploy**, puis lire les logs des 6 services. Le démarrage initial peut être long, surtout pour PostgreSQL, Elasticsearch et Temporal.
9. Ouvrir `https://postiz.oraclelumira.com`. Vérifier que le certificat TLS est valide et que la connexion / inscription fonctionne.

## Sécuriser après la première connexion

- L'inscription est ouverte pour permettre la création du premier compte : `POSTIZ_DISABLE_REGISTRATION=false` par défaut. **Dès que l'administrateur est créé**, définir `POSTIZ_DISABLE_REGISTRATION=true` sous Environment Variables puis **Redeploy**. Cela évite les inscriptions non sollicitées.
- Ajouter les identifiants OAuth des réseaux sociaux à la configuration `postiz.environment` **en référant les secrets Coolify** (jamais en dur dans GitHub), et suivre la documentation de chaque fournisseur.
- Configurer des sauvegardes récurrentes des volumes PostgreSQL, uploads Postiz, et Temporal. Vérifier leur restauration.
- Garder le JWT stable entre redéploiements ; sa rotation déconnecte les utilisateurs.
- Ne pas publier directement PostgreSQL, Redis, Elasticsearch ou Temporal sur le serveur.

## Détails d'architecture

Le Compose dédié utilise un **montage de fichier inline `content:` propre à Coolify** pour le fichier de configuration dynamique de Temporal. Ce champ est prévu par la documentation Coolify ; le fichier `docker-compose.coolify.yml` n'est donc **pas** destiné à une exécution directe en dehors de Coolify sans adaptation de ce montage.

Les labels `traefik.enable=false` empêchent le proxy d'exposer les cinq conteneurs internes. Uniquement le service `postiz` reçoit un domaine dans la configuration Coolify.

## Documentation

- https://docs.postiz.com/self-host/installation/docker-compose
- https://docs.postiz.com/self-host/installation/system-requirements
- https://coolify.io/docs/applications/builds/docker-compose
- https://coolify.io/docs/core/persistent-storage/storage-mounts/file-mounts
