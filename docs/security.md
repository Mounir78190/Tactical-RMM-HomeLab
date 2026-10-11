# Sécurité de la publication

Classification : **TLP:CLEAR**.

## Principes appliqués

- serveur Tactical RMM isolé dans une VM dédiée ;
- administration réservée aux réseaux privés et à WireGuard ;
- HTTPS avec une chaîne de confiance valide ;
- trois noms DNS distincts pour le frontend, l'API et MeshCentral ;
- authentification multifacteur recommandée ;
- comptes nominatifs et droits minimaux ;
- sauvegarde avant installation et changements majeurs ;
- maintien de Wazuh comme couche de détection indépendante.

## Éléments exclus du dépôt

- mots de passe et adresses e-mail privées ;
- clés Wi-Fi apparues dans une sortie Netplan ;
- jetons d'enrôlement et installateurs d'agents ;
- valeur du challenge ACME DNS-01 ;
- certificats, clés privées et fichiers PEM ;
- identifiants MeshCentral et comptes d'urgence ;
- adresses IP publiques et configuration WireGuard ;
- sauvegardes QCOW2, bases de données et journaux bruts ;
- scripts tiers non validés pendant le POC.

Les secrets qui ont été affichés dans l'échange de travail doivent être
considérés comme compromis et remplacés avant toute publication, même s'ils ne
sont pas présents dans ce dépôt.

## Captures retenues

Les preuves publiées sont recadrées pour retirer le navigateur et la barre des
tâches. Les colonnes contenant le nom de session Windows sont masquées. Les
adresses RFC 1918 et les noms fonctionnels utiles à la compréhension sont
conservés.

Les captures montrant une adresse e-mail, une IP publique, une valeur TXT ACME
ou un secret de configuration sont exclues.

## Sécurité de l'instance

Avant une utilisation autre qu'un laboratoire :

1. Activer TOTP pour tous les comptes interactifs.
2. Restreindre TCP 443 aux sources nécessaires si l'usage le permet.
3. Restreindre SSH aux postes d'administration et utiliser des clés.
4. Désactiver ou limiter les scripts côté serveur.
5. Revoir les permissions donnant accès au terminal et au contrôle distant.
6. Journaliser les actions d'administration.
7. Mettre à jour Tactical RMM selon la documentation officielle.
8. Sauvegarder la plateforme et tester régulièrement la restauration.
9. Surveiller l'hôte et la VM depuis un système indépendant.
10. Révoquer immédiatement tout jeton ou secret suspecté d'exposition.

## Vérification avant publication

```text
password | passwd | token | secret | api_key | Authorization
BEGIN PRIVATE KEY | privkey.pem | _acme-challenge
```

Chaque occurrence doit être revue manuellement. Une mention documentaire est
acceptable ; une valeur réelle ne l'est pas.
