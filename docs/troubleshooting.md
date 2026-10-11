# Incidents et dépannage

## DNS public initialement incomplet

### Symptôme

Les noms utilisés pendant la préparation ne se résolvaient pas depuis un
résolveur public ou la vérification DNS expirait.

### Analyse

Le DNS interne nécessaire aux agents et le DNS public requis pour un challenge
ACME sont deux sujets distincts. Une entrée locale pfSense ne peut pas être vue
par l'autorité de certification.

### Résolution

- créer les noms internes pour les agents ;
- publier uniquement le TXT ACME demandé dans la zone autoritaire ;
- vérifier le TXT depuis un résolveur public ;
- attendre la propagation avant de poursuivre.

## Échec du challenge DNS-01

### Symptôme

Certbot retourne `NXDOMAIN` pour `_acme-challenge`.

### Causes observées ou probables

- nom saisi avec le domaine ajouté deux fois par l'interface DNS ;
- ancienne valeur TXT encore présente ;
- propagation non terminée ;
- validation lancée avant la visibilité publique.

### Résolution

Vérifier le nom complet produit par le fournisseur DNS. Certaines interfaces
attendent uniquement `_acme-challenge.homelab` et ajoutent automatiquement le
domaine racine. Ne jamais republier dans Git la valeur TXT temporaire.

## Erreur Certbot `AttributeError: can't set attribute`

### Symptôme

La création du certificat a échoué malgré la présence du TXT.

### Traitement appliqué au POC

L'installation a été reprise avec un certificat wildcard valide et ses chemins
`fullchain.pem` / `privkey.pem`, puis l'installateur TRMM a été lancé avec
`--use-own-cert`.

### Bonne pratique

Vérifier la version et la méthode d'installation de Certbot, consulter
`/var/log/letsencrypt/letsencrypt.log` et tester d'abord en environnement de
préproduction. Ne pas multiplier les demandes de production afin d'éviter les limites
de l'autorité de certification.

## Installation TRMM interrompue

### Risque

Une interruption partielle peut laisser des services, bases ou configurations
Nginx incohérents. Relancer aveuglément l'installateur peut aggraver l'état.

### Résolution appliquée

La VM a été restaurée depuis une sauvegarde ou une copie QCOW2 prise avant
l'installation, puis la procédure a été reprise sur un état propre.

### Enseignement

Prendre un snapshot cohérent avant l'installation et avant chaque mise à jour
majeure. Tester également la restauration, pas seulement la création de la sauvegarde.

## Agent Windows bloqué

### Symptômes et contrôles

- installateur bloqué ou supprimé par l'antivirus ;
- agent installé mais absent du Dashboard ;
- MeshCentral non opérationnel ;
- restrictions McAfee ou Smart App Control.

### Methode

1. Valider la signature de l'installateur officiel.
2. Contrôler la résolution de `rmm`, `api` et `mesh`.
3. Tester TCP 443 et la chaîne TLS.
4. Consulter les journaux de l'agent et de l'antivirus.
5. Créer une exception minimale seulement après validation du risque.
6. Éviter toute désactivation permanente de l'antivirus.

## Agent Linux non disponible

L'interface a retourné une erreur indiquant que les agents Linux/macOS exigent
la signature de code. La documentation officielle précise que les agents signés,
dont Linux et macOS, sont accessibles aux sponsors disposant du niveau requis.

Le POC ne contourne pas cette restriction. Les machines Linux sont donc
supervisées temporairement par des checks réseau, tandis que Wazuh conserve sa
télémétrie de sécurité.

## Échec d'un script PowerShell

Un test de script a signalé l'absence de PowerShell 7 au chemin attendu. Cet
essai concernait une automatisation annexe et ne valide pas le déploiement du
module testé.

Avant toute automatisation :

- déclarer les prérequis du script ;
- vérifier l'interpréteur et son chemin ;
- tester sur un endpoint pilote ;
- limiter les privilèges ;
- définir un délai d'expiration et un code retour explicite ;
- documenter le retour arrière.
