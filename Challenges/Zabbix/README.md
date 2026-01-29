# Atelier-Zabbix
# Challenge SB3E03 – Zabbix

## 🎯 Objectif du challenge

Mettre en place une **alarme Zabbix** qui se déclenche **uniquement si le fichier `test.txt` n’existe pas** sur les machines Windows surveillées.

---

## 🧪 Étape 1 – Création du fichier de test

Sur la machine Windows surveillée, créer un fichier :

- Emplacement : `C:\`
- Nom : `test.txt`

Ce fichier servira de référence pour tester le déclenchement de l’alarme.
<img width="719" height="237" alt="image" src="https://github.com/user-attachments/assets/4ecd9812-a201-4fd3-9f07-aa859cbf92ad" />


---

## 📡 Étape 2 – Création de l’item Zabbix

Créer (ou utiliser) un **item Zabbix** permettant de vérifier l’existence du fichier.

### Chemin dans l’interface Zabbix
- `Data collection` → `Hosts`
- Sélectionner la machine **Windows10**
- Onglet **Items**
- Cliquer sur **Create item**

### Paramètres de l’item
- **Name** : fichier test.txt  
- **Type** : Zabbix agent (passif)  
- **Key** : clé interrogeant l’agent sur la présence du fichier  
- **Type of information** : Numeric (unsigned)  
  - `1` = fichier existant  
  - `0` = fichier absent  
- **Update interval** : 30s  

Valider avec **Add**.
<img width="1040" height="777" alt="image" src="https://github.com/user-attachments/assets/9a717cfb-dad9-4026-b6ad-292e258a1246" />


### Vérification
- Aller dans `Monitoring` → `Latest data`
- Vérifier que la valeur retournée est `1` lorsque le fichier existe
<img width="1561" height="493" alt="image" src="https://github.com/user-attachments/assets/0f062d8d-e242-4d59-8cb3-492b931207e8" />


---

## 🚨 Étape 3 – Création du trigger

### Chemin dans l’interface
- `Data collection` → `Hosts`
- Sélectionner **Windows10**
- Onglet **Triggers**
- Cliquer sur **Create trigger**

### Paramètres du trigger
- **Name** : Alerte absence fichier test.txt  
- **Severity** : High
<img width="1037" height="771" alt="image" src="https://github.com/user-attachments/assets/86e488e8-beb4-4a4f-b7aa-53b892566174" />
  
### Expression
- Ajouter l’item créé précédemment
- Déclenchement lorsque la valeur est `0`
<img width="673" height="264" alt="image" src="https://github.com/user-attachments/assets/568de1cc-a87c-422d-93ba-84f3737515c7" />


Le trigger doit apparaître avec le statut **Enabled**.
<img width="1691" height="97" alt="image" src="https://github.com/user-attachments/assets/4a5b66a1-190b-45ae-8f59-38f7d56f52e2" />


---

## ❌ Étape 4 – Test de déclenchement

1. Supprimer le fichier `test.txt`
2. Aller dans `Monitoring` → `Problems`
3. Vérifier l’apparition de l’alarme
<img width="1713" height="432" alt="image" src="https://github.com/user-attachments/assets/629d8b8b-2c90-46b2-bade-4fbfdea598de" />


---

## ✅ Étape 5 – Résolution automatique

1. Recréer le fichier `test.txt`
2. Vérifier dans `Monitoring` → `Problems` que le statut passe à **RESOLVED**
<img width="1685" height="399" alt="image" src="https://github.com/user-attachments/assets/8452c280-b08c-40fd-8f49-14fca5890166" />


---

## ⭐ Bonus – Affichage dans le Dashboard

### Ajout du widget
- `Dashboards` → **Edit dashboard** → **Add**
- **Type** : Item value  
- **Item** : mon fichier test.txt
<img width="703" height="740" alt="image" src="https://github.com/user-attachments/assets/ef1a6765-aa50-466a-bee0-6464a3e84e0e" />


### Thresholds
- `0` → Rouge  
- `1` → Vert  
<img width="693" height="771" alt="image" src="https://github.com/user-attachments/assets/57383b9b-0433-485f-b9cc-ff8d0eb9411e" />

Enregistrer avec **Save changes**.



---

## 🟢🔴 Résultat final

- 🟢 Vert : fichier présent
<img width="1060" height="591" alt="image" src="https://github.com/user-attachments/assets/74920778-5395-49e2-a3f6-de67ec1af893" />

- 🔴 Rouge : fichier absent
<img width="1155" height="579" alt="image" src="https://github.com/user-attachments/assets/73a945a4-3abc-407f-8e9a-7f1c205ffcdd" />


---

✅ **Challenge Zabbix SB3E03 validé**
