# Securite de la publication

Classification : **TLP:CLEAR**.

## Principes appliques

- serveur Tactical RMM isole dans une VM dediee ;
- administration reservee aux reseaux prives et a WireGuard ;
- HTTPS avec une chaine de confiance valide ;
- trois noms DNS distincts pour le frontend, l'API et MeshCentral ;
- authentification multifacteur recommandee ;
- comptes nominatifs et droits minimaux ;
- sauvegarde avant installation et changements majeurs ;
- maintien de Wazuh comme couche de detection independante.

## Elements exclus du depot

- mots de passe et adresses e-mail privees ;
- cles Wi-Fi apparues dans une sortie Netplan ;
- jetons d'enrolement et installateurs d'agents ;
- valeur du challenge ACME DNS-01 ;
- certificats, cles privees et fichiers PEM ;
- identifiants MeshCentral et comptes d'urgence ;
- adresses IP publiques et configuration WireGuard ;
- sauvegardes QCOW2, bases de donnees et journaux bruts ;
- scripts tiers non valides pendant le POC.

Les secrets qui ont ete affiches dans l'echange de travail doivent etre
consideres compromis et remplaces avant toute publication, meme s'ils ne sont
pas presents dans ce depot.

## Captures retenues

Les preuves publiees sont recadrees pour retirer le navigateur et la barre des
taches. Les colonnes contenant le nom de session Windows sont masquees. Les
adresses RFC 1918 et les noms fonctionnels utiles a la comprehension sont
conserves.

Les captures montrant une adresse e-mail, une IP publique, une valeur TXT ACME
ou un secret de configuration sont exclues.

## Securite de l'instance

Avant une utilisation autre qu'un laboratoire :

1. Activer TOTP pour tous les comptes interactifs.
2. Restreindre TCP 443 aux sources necessaires si l'usage le permet.
3. Restreindre SSH aux postes d'administration et utiliser des cles.
4. Desactiver ou limiter les scripts cote serveur.
5. Revoir les permissions donnant acces au terminal et au controle distant.
6. Journaliser les actions d'administration.
7. Mettre a jour Tactical RMM selon la documentation officielle.
8. Sauvegarder la plateforme et tester regulierement la restauration.
9. Surveiller l'hote et la VM depuis un systeme independant.
10. Revoquer immediatement tout jeton ou secret suspecte d'exposition.

## Verification avant publication

```text
password | passwd | token | secret | api_key | Authorization
BEGIN PRIVATE KEY | privkey.pem | _acme-challenge
```

Chaque occurrence doit etre revue manuellement. Une mention documentaire est
acceptable ; une valeur reelle ne l'est pas.
