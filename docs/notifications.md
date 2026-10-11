# Notifications Dashboard et e-mail

Cette etape repond au besoin d'envoyer le `Warning` d'un check a l'utilisateur
concerne. Elle est documentee d'apres les fonctions officielles de Tactical RMM,
mais l'envoi SMTP n'a pas encore ete valide dans ce POC.

## Fonctionnement

La severite est definie sur le check. La diffusion de l'alerte est geree par un
`Alert Template`. Tactical RMM prend notamment en charge :

- e-mail ;
- notification Dashboard sous forme de pop-up ;
- SMS via Twilio ;
- webhook ;
- action par script lors de l'echec ou de la resolution.

## 1. Configurer SMTP

Dans `Settings > Global Settings > Email Alerts` :

| Champ | Recommandation |
|---|---|
| Recipients | boite nominative ou groupe d'astreinte |
| From | adresse technique dediee au RMM |
| Host | relais SMTP du fournisseur |
| Port | `587` avec STARTTLS, selon le fournisseur |
| Authentication | activee si le relais l'exige |
| Username | compte SMTP dedie |
| Password | secret unique stocke hors Git |

Pour Gmail ou Microsoft 365, utiliser une methode d'authentification compatible
avec la politique du tenant. Ne pas employer le mot de passe personnel d'un
administrateur. Privilegier un compte applicatif, un mot de passe d'application
si autorise, ou un relais explicitement restreint.

## 2. Creer le modele d'alerte

Dans `Settings > Alerts Manager > New` :

1. Nommer le modele `HomeLab - Warning Notifications`.
2. Ajouter les destinataires concernes ou conserver ceux des Global Settings.
3. Dans la section des checks, activer `Email`.
4. Activer `Dashboard` pour obtenir le pop-up dans l'interface.
5. Configurer `Alert on severity` sur `Warning`.
6. Activer la notification de resolution si elle est utile.
7. Definir une relance uniquement si l'alerte reste non resolue.
8. Enregistrer.

Le parametre `Alert on severity` est indispensable pour la notification des
checks et des taches.

## 3. Affecter le modele

Le modele peut etre applique :

- globalement dans `Settings > Global Settings` ;
- au client du HomeLab ;
- au site `HomeLab - Infrastructure` ;
- par une Automation Policy.

Pour ce POC, l'affectation au site limite les notifications au laboratoire. Les
templates les plus proches de l'agent sont prioritaires sur le template global.

## 4. Verifier les indicateurs du check

Dans la liste des checks, les colonnes avec les icones enveloppe et cloche
doivent refleter l'activation de l'e-mail et du Dashboard. Un niveau `Warning`
sur le check ne suffit pas, a lui seul, a envoyer un message.

## 5. Tester sans couper pfSense

Ne pas arreter le pare-feu principal. Creer temporairement un check de recette
vers une adresse privee inutilisee, par exemple dans un sous-reseau de test :

1. definir un intervalle court uniquement pendant la recette ;
2. demander un seul echec avant alerte ;
3. verifier le pop-up Dashboard ;
4. verifier la reception et les en-tetes de l'e-mail ;
5. remettre la cible joignable ou supprimer le check ;
6. confirmer la notification de resolution ;
7. restaurer les seuils de production du laboratoire.

## Criteres d'acceptation

| Test | Resultat attendu |
|---|---|
| Echec de check `Warning` | alerte visible dans le Dashboard |
| Distribution SMTP | e-mail recu par le destinataire configure |
| Contenu | agent, check, severite et horodatage identifiables |
| Relance | absence de spam, periodicite conforme |
| Resolution | alerte fermee et message de retour recu si active |
| Audit | evenement consultable dans Tactical RMM |

## References officielles

- [Alerting](https://docs.tacticalrmm.com/functions/alerting/)
- [Email and SMS Alerts](https://docs.tacticalrmm.com/functions/emailsms_alert/)
- [Global Settings](https://docs.tacticalrmm.com/functions/global_settings/)
