# 🛠️ Atelier PowerShell – Gestion Active Directory

---

## 📌 Contexte

Vous êtes administrateur système chez **TechSecure**, une entreprise de 200 employés utilisant **Active Directory** pour la gestion des identités.

L’objectif de cet atelier est d’automatiser l’administration Active Directory à l’aide de **PowerShell**, en remplaçant les opérations manuelles réalisées via ADUC.

---

## 🧱 Prérequis

- Machine membre du domaine
- Droits administrateur Active Directory
- PowerShell 5.1+
- RSAT installés

---

# 🧪 Partie 1 – Découverte du module ActiveDirectory

## 1.1 Vérification et installation du module

```powershell
Get-Module -ListAvailable ActiveDirectory
```
<img width="1154" height="976" alt="1" src="https://github.com/user-attachments/assets/2b475a46-d0a6-4995-bcb9-9b57d686c580" />

## 1.2 Exploration du module

```powershell
(Get-Command -Module ActiveDirectory).Count
Get-Command Get-ADUser*
```
<img width="911" height="236" alt="2" src="https://github.com/user-attachments/assets/fb8a9d9a-7ea6-4b6d-a190-31b7c296ce8e" />

## 1.3 Infomrmation sur le domaine 

```powershell
Get-ADDomain | Select-Object NetBiosName, DomainMode
Get-ADDomainController -Filter * | Select-Object Name, IPv4Addres, OperatingSystem
```
<img width="1026" height="253" alt="3" src="https://github.com/user-attachments/assets/bd77c3ca-d051-4560-8344-412ea7986bc9" />

## 1.4 Information sur mon compte AD

```powershell
Get-ADUser -Identity Administrateur -Properties *
```
<img width="1165" height="974" alt="4" src="https://github.com/user-attachments/assets/621e6ff6-c0c2-4227-b9f5-0acdd79fa723" />

# 👤 PARTIE 2 – Gestion des utilisateurs

## 2.1 Création des utilisateurs 
- Création des utilisateurs dans l'OU de mon choix, ensuite je définis pour chaque user un mot de passe initial, Le compte doit être activé, et Le mot de passe doit être changé à la première connexion.
```powershell
$Pwd = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force
New-ADUser -Name "Alice Martin" `
```
<img width="922" height="634" alt="5" src="https://github.com/user-attachments/assets/ee6aad13-a723-42e5-a287-652ed43832ba" />

- Je retape les mêmes commandes pour les users Bob Dubois et Claire Bernard.

## 2.2 Recherche d’utilisateurs
```powershell
Get-ADUser -Identity amartin
Get-ADUser -Filter "Name -like 'B*'"
Get-ADUser -Filter "Title -like '*Administrateur*'" -Properties Title
(Get-ADUser -Filter *).Count
```
<img width="890" height="984" alt="7" src="https://github.com/user-attachments/assets/b7b645db-fe55-4a44-b85f-54fcab2757bc" />

## 2.3 Modification d’un utilisateur
- Pour l'utilisateur "amartin" :
1. Je Change son numéro de téléphone
2. J'ajoute une descirption
3. Je Change son titre en "Développeuse Senior"
```powershell
Set-ADUser amartin `
-OfficePhone "01 23 45 67 89" `
-Description "Membre de l'équipe développement" `
-Title "Développeuse Senior"
```
<img width="1417" height="203" alt="8" src="https://github.com/user-attachments/assets/099b6caf-a711-4698-bb13-4a49d5f484f8" />

## 2.4 Désactivation et suppression
- Pour l'utilisateur "bdubois" :
1. Je désactive son compte
2. Je vérifie qu'il est bien désactivé
3. Je supprime le compte "cbernard" avec une demande de confirmation
```powershell
Disable-ADAccount -Identity bdubois
Get-ADUser -Identity bdubois | Select-Object Name, Enabled
Remove-ADUser -Identity cbernard -Confirm
```
<img width="1066" height="309" alt="9" src="https://github.com/user-attachments/assets/55672838-1c10-460e-89bf-747f22fb81c9" />

# 👥 PARTIE 3 – Gestion des groupes
## 3.1 Création des groupes
```powershell
New-ADGroup GRP_Developpeurs -GroupScope Global -GroupCategory Security -Description "Équipe de développement"
New-ADGroup GRP_Admins_Systeme -GroupScope Global -GroupCategory Security -Description "Administrateurs système"
New-ADGroup GRP_Chefs_Projet -GroupScope Global -GroupCategory Security -Description "Chefs de projet"
New-ADGroup GRP_IT -GroupScope Global -GroupCategory Security -Description "Département IT"
```
<img width="614" height="346" alt="10" src="https://github.com/user-attachments/assets/4d4f9d63-3ac9-481a-8733-67583d547413" />

## 3.2 Ajouter des membres
1. J'ajoute "amartin" dans le groupe "GRP_Developpeurs"
2. J'ajoute "bdubois" dans le groupe "GRP_Admins_Systeme"
3. Je crée 2 nouveaux utilisateurs et ajoutez-les dans "GRP_Developpeurs"
4. J'ajoute tous les membres des trois premiers groupes dans "GRP_IT"
```powershell
Add-ADGroupMember -Identity "GRP_Developpeurs" -Members "amartin"
Add-ADGroupMember -Identity "GRP_Admins_Systeme" -Members "bdubois"
New-ADUser -Name "Jean Test" -SamAccountName "jtest" -AccountPassword $Password -Enable $true
New-ADUser -Name "Marc Test" -SamAccountName "mtest" -AccountPassword $Password -Enable $true
Add-ADGroupMember -Identity "GRP_Developpeurs" -Members "jtest", "mtest"
Add-ADGroupMember -Identity "GRP_IT" -Members "GRP_Developpeurs", "GRP_Admins_Systeme", "GRP_Chefs_Projet"
```
<img width="1238" height="189" alt="11" src="https://github.com/user-attachments/assets/ecb1518e-bcaa-4b45-b5b6-2dbef7429518" />

## 3.3 Lister les appartenances
1. J'affiche tous les membres du groupe "GRP_IT"
2. J'affiche tous les groupes dont "amartin" est membre
3. Je compte combien de membres a chaque groupe
```powershell
Get-ADGroupMember -Identity "GRP_IT" -Recursive | Select-Object Name, SamAccountName
Get-ADPrincipalGroupMembership -Identity amartin | Select-Object Name
"GRP_Developpeurs", "GRP_Admins_Systeme", "GRP_Ches_Projet", "GRP_IT" | ForEach-Object {
>>     $group = Get-ADGroup -Identity $_
>>     $count = (Get-ADGroupMember -identity $_).Count
>>     [PSCustomObject]@{
>>         GroupName = $_
>>         MemberCount = $count
>>     }
>>}
```
<img width="1080" height="699" alt="12" src="https://github.com/user-attachments/assets/34a40901-52b2-4b01-948d-a546ad50d2a4" />

## 3.4 - Retirer des membres
- Je retire "amartin" du groupe "GRP_IT"
```powershell
Remove-ADGroupMember -identity "GRP_IT" -Members "amartin" -Confirm
```

## 3.5 Groupes imbriqués
1. Je crée un groupe "GRP_Tous_Utilisateurs"
2. J'y Ajoute les groupes "GRP_IT"
3. Je Liste les membres (directs et récursifs) de "GRP_Tous_Utilisateurs"
```powershell
New-ADGroup -Name "GRP_Tous_Utilisateurs" ` -GroupScope Global -GroupCategory Security -Description "groupe incluant tous les utilisateurs via GRP_IT"
Add-ADGroupMember -Identity "GRP_Tous_Utilisateurs" -Members "GRP_IT"
Get-ADGroupMember -Identity "GRP_Tous_Utilisateurs" -Recursive | Select-Object Name, SameAccountName
```
<img width="1175" height="401" alt="14" src="https://github.com/user-attachments/assets/6017b6af-bb47-4f8f-8bdb-ce157f4af2ad" />

# 🗂️ PARTIE 4 – Unités Organisationnelles
## 4.1 Création des OU
```powershell
New-ADOrganizationalUnit -Name "TechSecure" -Path "DC=TECHSECURE, DC=LOCAL"
$rootpath = "OU=TechSecure, DC=TECHSECURE, DC=LOCAL"
New-ADOrganizationalUnit -Name "Utilisateurs" -Path $rootpath
New-ADOrganizationalUnit -Name "Groupes" -Path $rootpath
New-ADOrganizationalUnit -Name "Ordinateurs" -Path $rootpath
$userPath = OU=Utilisateurs, OU=TechSecure, DC=TECHSECURE, DC=LOCAL"
New-ADOrganizationalUnit -Name "Informatique" -Path $userPath
New-ADOrganizationalUnit -Name "RH" -Path $userPath
New-ADOrganizationalUnit -Name "commercial" -Path $userPath
$infoPath = "OU=Informatique, OU=Utilisateurs, OU=TechSecure, DC=TECHSECURE, DC=local"
New-ADOrganizationalUnit -Name "developpement" -Path $infoPath
New-ADOrganizationalUnit -Name "Infrastructure" -Path $infoPath
```
<img width="1033" height="265" alt="15" src="https://github.com/user-attachments/assets/1679c6e6-d0c1-4265-b11f-4f819bba67aa" />

## 4.2 Déplacement des objets
1. Je déplace l'utilisateur "amartin" dans l'OU "TechSecure/Utilisateurs/Informatique/Developpement"
2. Je Déplace tous vos groupes créés précédemment dans l'OU "TechSecure/Groupes"
3. Je Liste tous les utilisateurs présents dans l'OU "Informatique" (incluant les sous-OU)
```powershell
Move-ADObject -Identity (Get-ADUser amartin).DistinguishedName `
>> -TargetPath "OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr"
Get-ADGroup -Filter "Name -like 'GRP_*'" | `
>> Move-ADObject -TergetPath "OU=Groupes, OU=TechSecure, DC=TECHSECURE, DC=LOCAL"
Get-ADUser -Filter * -SearchBase "OU=Informatique, OU=Utilisateurs, OU=Techsecure, DC=TECHSECURE, DC=LOCAL" -SearchScope Subtree | `
>> Select-Object Name, SamAccountName, DistinguishedName
```
<img width="1427" height="215" alt="16" src="https://github.com/user-attachments/assets/77634265-4039-4657-a5d2-42b9cdcfc344" />

## 4.3 - Recherche par OU
1. Je compte le nombre d'utilisateurs dans l'OU "Informatique" uniquement (sans les sous-OU)
2. Je compte le nombre d'utilisateurs dans l'OU "Informatique" en incluant tous les sous-niveaux
```powershell
(Get-ADUser -Filter * -SearchBase "OU=Informatique, OU=Utilisateurs, OU=Techsecure, DC=TECHSECURE, DC=LOCAL" -SearchScope OneLevel).Count
@(Get-ADUser -Filter * -SearchBase "OU=Informatique, OU=Utilisateurs, OU=Techsecure, DC=TECHSECURE, DC=LOCAL" -SearchScope Subtree).Count
```
<img width="1579" height="147" alt="17" src="https://github.com/user-attachments/assets/6fc5f8c5-da94-49b4-a75c-1a75fd270e04" />






































