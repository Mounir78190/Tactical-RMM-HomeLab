# Notifications Dashboard et e-mail

Cette étape répond au besoin d'envoyer le `Warning` d'un check à l'utilisateur
concerné. Elle est documentée d'après les fonctions officielles de Tactical RMM,
mais l'envoi SMTP n'a pas encore été validé dans ce POC.

## Fonctionnement

La sévérité est définie sur le check. La diffusion de l'alerte est gérée par un
`Alert Template`. Tactical RMM prend notamment en charge :

- e-mail ;
- notification Dashboard sous forme de pop-up ;
- SMS via Twilio ;
- webhook ;
- action par script lors de l'échec ou de la résolution.

## 1. Configurer SMTP

Dans `Settings > Global Settings > Email Alerts` :

| Champ | Recommandation |
|---|---|
| Recipients | boîte nominative ou groupe d'astreinte |
| From | adresse technique dédiée au RMM |
| Host | relais SMTP du fournisseur |
| Port | `587` avec STARTTLS, selon le fournisseur |
| Authentication | activée si le relais l'exige |
| Username | compte SMTP dédié |
| Password | secret unique stocke hors Git |

Pour Gmail ou Microsoft 365, utiliser une méthode d'authentification compatible
avec la politique de l'environnement. Ne pas employer le mot de passe personnel d'un
administrateur. Privilégier un compte applicatif, un mot de passe d'application
si autorisé, ou un relais explicitement restreint.

## 2. Créer le modèle d'alerte

Dans `Settings > Alerts Manager > New` :

1. Nommer le modèle `HomeLab - Warning Notifications`.
2. Ajouter les destinataires concernés ou conserver ceux des Global Settings.
3. Dans la section des checks, activer `Email`.
4. Activer `Dashboard` pour obtenir le pop-up dans l'interface.
5. Configurer `Alert on severity` sur `Warning`.
6. Activer la notification de résolution si elle est utile.
7. Définir une relance uniquement si l'alerte reste non résolue.
8. Enregistrer.

Le paramètre `Alert on severity` est indispensable pour la notification des
checks et des tâches.

## 3. Affecter le modèle

Le modèle peut être appliqué :

- globalement dans `Settings > Global Settings` ;
- au client du HomeLab ;
- au site `HomeLab - Infrastructure` ;
- par une Automation Policy.

Pour ce POC, l'affectation au site limite les notifications au laboratoire. Les
modèles les plus proches de l'agent sont prioritaires sur le modèle global.

## 4. Vérifier les indicateurs du check

Dans la liste des checks, les colonnes avec les icônes d'enveloppe et de cloche
doivent refléter l'activation de l'e-mail et du Dashboard. Un niveau `Warning`
sur le check ne suffit pas, à lui seul, à envoyer un message.

## 5. Tester sans couper pfSense

Ne pas arrêter le pare-feu principal. Créer temporairement un check de recette
vers une adresse privée inutilisée, par exemple dans un sous-réseau de test :

1. définir un intervalle court uniquement pendant la recette ;
2. demander un seul échec avant alerte ;
3. vérifier le pop-up Dashboard ;
4. vérifier la réception et les en-têtes de l'e-mail ;
5. remettre la cible joignable ou supprimer le check ;
6. confirmer la notification de résolution ;
7. restaurer les seuils de production du laboratoire.

## Critères d'acceptation

| Test | Résultat attendu |
|---|---|
| Échec de check `Warning` | alerte visible dans le Dashboard |
| Distribution SMTP | e-mail reçu par le destinataire configuré |
| Contenu | agent, check, sévérité et horodatage identifiables |
| Relance | absence de spam, périodicité conforme |
| Résolution | alerte fermée et message de retour reçu si activé |
| Audit | événement consultable dans Tactical RMM |

## Références officielles

- [Alerting](https://docs.tacticalrmm.com/functions/alerting/)
- [Email and SMS Alerts](https://docs.tacticalrmm.com/functions/emailsms_alert/)
- [Global Settings](https://docs.tacticalrmm.com/functions/global_settings/)
