# Incidents et depannage

## DNS public initialement incomplet

### Symptome

Les noms utilises pendant la preparation ne se resolvaient pas depuis un
resolveur public ou la verification DNS expirait.

### Analyse

Le DNS interne necessaire aux agents et le DNS public requis pour un challenge
ACME sont deux sujets distincts. Une entree locale pfSense ne peut pas etre vue
par l'autorite de certification.

### Resolution

- creer les noms internes pour les agents ;
- publier uniquement le TXT ACME demande dans la zone autoritaire ;
- verifier le TXT depuis un resolveur public ;
- attendre la propagation avant de poursuivre.

## Echec du challenge DNS-01

### Symptome

Certbot retourne `NXDOMAIN` pour `_acme-challenge`.

### Causes observees ou probables

- nom saisi avec le domaine ajoute deux fois par l'interface DNS ;
- ancienne valeur TXT encore presente ;
- propagation non terminee ;
- validation lancee avant la visibilite publique.

### Resolution

Verifier le nom complet produit par le fournisseur DNS. Certaines interfaces
attendent uniquement `_acme-challenge.homelab` et ajoutent automatiquement le
domaine racine. Ne jamais republier dans Git la valeur TXT temporaire.

## Erreur Certbot `AttributeError: can't set attribute`

### Symptome

La creation du certificat a echoue malgre la presence du TXT.

### Traitement du POC

L'installation a ete reprise avec un certificat wildcard valide et ses chemins
`fullchain.pem` / `privkey.pem`, puis l'installateur TRMM a ete lance avec
`--use-own-cert`.

### Bonne pratique

Verifier la version et la methode d'installation de Certbot, consulter
`/var/log/letsencrypt/letsencrypt.log` et tester d'abord en environnement de
staging. Ne pas multiplier les demandes de production afin d'eviter les limites
de l'autorite de certification.

## Installation TRMM interrompue

### Risque

Une interruption partielle peut laisser des services, bases ou configurations
Nginx incoherents. Relancer aveuglement l'installateur peut aggraver l'etat.

### Resolution appliquee

La VM a ete restauree depuis une sauvegarde ou copie QCOW2 prise avant
l'installation, puis la procedure a ete reprise sur un etat propre.

### Enseignement

Prendre un snapshot coherent avant l'installation et avant chaque mise a jour
majeure. Tester egalement la restauration, pas seulement la creation du backup.

## Agent Windows bloque

### Symptomes et controles

- installateur bloque ou supprime par l'antivirus ;
- agent installe mais absent du Dashboard ;
- MeshCentral non operationnel ;
- restrictions McAfee ou Smart App Control.

### Methode

1. Valider la signature de l'installateur officiel.
2. Controler la resolution de `rmm`, `api` et `mesh`.
3. Tester TCP 443 et la chaine TLS.
4. Consulter les journaux de l'agent et de l'antivirus.
5. Creer une exception minimale seulement apres validation du risque.
6. Eviter toute desactivation permanente de l'antivirus.

## Agent Linux non disponible

L'interface a retourne une erreur indiquant que les agents Linux/macOS exigent
la signature de code. La documentation officielle precise que les agents signes,
dont Linux et macOS, sont accessibles aux sponsors disposant du niveau requis.

Le POC ne contourne pas cette restriction. Les machines Linux sont donc
supervisees temporairement par checks reseau, tandis que Wazuh conserve sa
telemetrie de securite.

## Echec d'un script PowerShell

Un test de script a signale l'absence de PowerShell 7 au chemin attendu. Cet
essai concernait une automatisation annexe et ne valide pas le deploiement du
module teste.

Avant toute automatisation :

- declarer les prerequis du script ;
- verifier l'interpreteur et son chemin ;
- tester sur un endpoint pilote ;
- limiter les privileges ;
- definir un timeout et un code retour explicite ;
- documenter le retour arriere.
