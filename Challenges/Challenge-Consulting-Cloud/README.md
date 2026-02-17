# 📂 Case Study : Transformation Cloud - MediCare+

Ce document constitue le livrable de conseil pour la modernisation de l'infrastructure IT de la PME **MediCare+**.

---

## 1. Énoncé du Challenge (Le Contexte)

### 🏢 Profil de l'entreprise
* **Activité :** Services de santé (données sensibles, hors dossiers médicaux complets).
* **Effectif :** 50 employés.
* **Implantation :** Siège à Lyon, agences à Marseille et Paris.
* **Équipe IT :** Un administrateur système à mi-temps.

### ⚙️ État des lieux technique (100% On-premises)
* **Identité :** Active Directory + DNS/DHCP sur Windows Server.
* **Métier :** Application PHP/MySQL + stockage de fichiers sur Windows Server.
* **Web :** Site vitrine WordPress sur serveur Linux.
* **Backup :** NAS physique avec sauvegardes manuelles.
* **Réseau :** VPN inter-sites pour lier les agences.
* **Coût annuel estimé :** ~46 000 €.

### ⚠️ Problématiques identifiées
1. **Obsolescence :** Matériel à renouveler et coûts de maintenance élevés.
2. **Disponibilité :** Sauvegardes peu fiables et accès distant (télétravail) complexe.
3. **Évolutivité :** Montée en charge difficile pour accompagner la croissance.
4. **Conformité :** Gouvernance RGPD insuffisante pour le secteur santé.

---

## 2. Architecture Cible Recommandée

Nous préconisons une bascule vers un modèle **SaaS** pour la collaboration et **PaaS** pour les briques applicatives afin de libérer l'administrateur des tâches de bas niveau (patching, hardware).

| Composant | Proposition | Modèle | Provider | Justification |
| :--- | :--- | :--- | :--- | :--- |
| **Identités** | Microsoft Entra ID | SaaS | Azure | Suppression de l'AD physique. Authentification forte (MFA). |
| **Bureautique** | Microsoft 365 Business | SaaS | Microsoft | Messagerie cloud et outils collaboratifs sécurisés. |
| **Fichiers** | SharePoint / OneDrive | SaaS | Microsoft | Remplace le NAS. Gestion fine des droits et accès distant. |
| **App Métier** | Azure App Service | PaaS | Azure | Hébergement PHP managé (Haute disponibilité native). |
| **Base de Données** | Azure DB for MySQL | PaaS | Azure | SQL managé, backups automatiques et chiffrement. |
| **Sauvegardes** | Azure Backup | PaaS | Azure | Externalisation automatique des backups (Ransomware-proof). |
| **Site Web** | Hébergement Web Pro | PaaS | OVHcloud | Simplicité, coût minime et souveraineté française. |

### Schéma Logique de la Solution
`Utilisateurs (Lyon / Paris / Marseille) ➔ Connexion Internet (MFA)`
`  ↳ Accès aux ressources M365 (Mails/Docs)`
`  ↳ Accès App Métier (Azure Web Apps) ⟷ Stockage (Azure MySQL)`

---

### 2️⃣ Choix du provider cloud

| Critère | Azure | AWS | OVHcloud |
|------|------|-----|---------|
| Localisation France | Oui | Oui | Oui |
| Services managés | Très complets | Très complets | Plus limités |
| Simplicité | Élevée | Moyenne | Élevée |
| Intégration Microsoft | Excellente | Limitée | Limitée |
| Coût estimé | Modéré | Modéré à élevé | Plus bas |

#### Choix retenu : **Azure**

Azure est retenu car :
- l’infrastructure actuelle est majoritairement Windows,
- les services SaaS (Microsoft 365, Azure AD) simplifient fortement l’administration,
- les données peuvent être hébergées en France,
- la prise en main est adaptée à une PME avec peu de ressources IT.

---

## 4. Estimation Budgétaire (Ordre de grandeur)

L'objectif est de transformer le CapEx (achat serveur) en OpEx (abonnement).

* **Abonnements SaaS (M365) :** ~1 000 € / mois (Licences Business Premium avec sécurité).
* **Consommation Azure (App + DB) :** ~800 € / mois (Instance réservée pour économies).
* **Sécurité & Sauvegardes :** ~200 € / mois.
* **Total mensuel : ~2 000 € HT** (soit **24 000 € / an**).

**Impact financier :** Une réduction de près de **45% du coût annuel** par rapport à l'existant, tout en améliorant radicalement le niveau de service.

---

## 5. Points de Vigilance & Risques

1. **Migration des données :** Risque de perte ou corruption lors du transfert vers le Cloud.
   * *Solution :* Procédure "Lift-and-Shift" assistée par l'outil Azure Migration Service.
2. **Confidentialité (RGPD) :** Données de santé sensibles.
   * *Solution :* Choix de la région "France Central" uniquement. Chiffrement AES-256 activé par défaut.
3. **Accompagnement Technique :** L'admin à mi-temps doit changer ses habitudes.
   * *Solution :* Accompagnement par TechConseil sur les 3 premiers mois (transfert de compétences).
4. **Dépendance Réseau :** Risque de coupure internet.
   * *Solution :* Mise en place de double accès WAN (Fibre + Back-up 4G/5G) sur chaque site.
  
## ✅ Conclusion

La solution proposée permet :
- une réduction significative des coûts,
- une amélioration de la disponibilité et de la sécurité,
- une simplification de l’administration,
- une meilleure conformité RGPD,
- une architecture évolutive adaptée à la croissance de l’entreprise.

Cette approche privilégie la **simplicité**, la **cohérence** et l’**adaptation aux contraintes d’une PME**, conformément aux objectifs du challenge AIS.

---
*Livrable produit par l'équipe TechConseil pour MediCare+ - 2026*
