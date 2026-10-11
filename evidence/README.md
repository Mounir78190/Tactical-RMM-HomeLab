# Inventaire des preuves

Les captures ont ete recadrees et anonymisees pour une publication
`TLP:CLEAR`. Les originaux contenant des donnees inutiles ou sensibles ne sont
pas distribues.

Les empreintes SHA-256 sont publiees dans
[`checksums.sha256`](checksums.sha256).

| Fichier | Preuve | Traitement |
|---|---|---|
| `01-windows-agents.png` | Deux agents Windows en ligne dans le site HomeLab | navigateur retire, utilisateurs masques |
| `02-pfsense-check-ok.png` | Ping Check pfSense reussi, 3/3 paquets | navigateur retire, utilisateurs masques |
| `03-pfsense-check-settings.png` | Cible, severite, seuil et intervalle du check | navigateur retire, utilisateurs masques |

## Captures volontairement exclues

- erreurs DNS ou Certbot contenant des valeurs ACME ;
- interface du fournisseur DNS ;
- terminal affichant une adresse e-mail ;
- vue agent contenant une IP publique ;
- essais de scripts PowerShell sans rapport direct avec le resultat du POC ;
- sorties Netplan contenant des cles Wi-Fi en clair.

## Regles de conservation

1. Conserver les originaux uniquement dans un espace prive chiffre.
2. Ne publier que la version recadree et revue.
3. Regenerer les sommes SHA-256 apres toute modification.
4. Verifier en taille reelle les barres d'adresse, terminaux et infobulles.
5. Revoquer tout secret accidentellement expose avant de supprimer une capture.
