# MAITO-S-lab-7-sec
# Guide Complet : Audit de Sécurité Mobile avec MobSF et DIVA

---

**Auteur** : AIT OURAJLI MOHAMED   

---

## 🎯 Introduction

Ce guide vous accompagne dans la mise en place d'une **plateforme d'analyse de sécurité mobile** capable de détecter les vulnérabilités dans les applications Android. Vous allez combiner :

- **Mobile Security Framework (MobSF)** : Framework d'analyse statique et dynamique
- **Android Virtual Device (AVD)** : Émulateur Android rooté sans Play Store
- **DIVA (Damn Insecure and Vulnerable App)** : Application intentionnellement vulnérable pour l'apprentissage

**Durée totale estimée** : 30-45 minutes (en fonction de votre débit internet)

---

## 🔧 Prérequis Techniques

Avant de commencer, assurez-vous de disposer de :

### Logiciels Obligatoires
- **Android Studio** (dernière version) → [Télécharger](https://developer.android.com/studio)
- **Docker Desktop** → [Télécharger](https://www.docker.com/products/docker-desktop)
- **Git** → [Télécharger](https://git-scm.com)

### Configuration Système Requise
- **RAM disponible** : Minimum 8 GB (recommandé 16 GB)
- **Espace disque** : Minimum 30 GB libres
- **Processeur** : Support de la virtualisation (VT-x sur Intel, AMD-V sur AMD)
- **Système d'exploitation** : Windows 10+, macOS 10.14+, Linux (Ubuntu 18.04+)

### Vérification Préalable
```bash
# Vérifier l'installation d'Android SDK
adb version

# Vérifier Docker
docker --version

# Vérifier Git
git --version
```

---

## 🏗️ Configuration de l'Environnement Virtuel

### Phase 1️⃣ : Création de l'Image Virtuelle Personnalisée (⏱️ 10 min)

#### Étape 1.1 : Lancer le gestionnaire AVD
1. Ouvrez **Android Studio**
2. Accédez à : `Tools` → `Device Manager` (ou `AVD Manager`)
3. Cliquez sur `Create Virtual Device`

#### Étape 1.2 : Sélectionner le profil matériel
- Catégorie : `Phones`
- Modèle recommandé : **Pixel 5** ou **Pixel 6** (résolution ≥ 1080p)
- Cliquez : `Next`

#### Étape 1.3 : Configuration du système d'exploitation

| Paramètre | Valeur Recommandée | Raison |
|-----------|-------------------|--------|
| **Version Android** | API 29-30 | Support optimal de Frida et MobSF |
| **Architecture** | x86_64 | Performance d'émulation maximale |
| **Variant** | Android (sans Google APIs ni Play Store) | Sécurité renforcée, moins de services externes |

**⚠️ Point critique** : Assurez-vous que l'image ne contient **PAS** de "Google Play Store" dans le nom.

#### Étape 1.4 : Nommage et finalisation
- Définissez le nom de l'AVD : `MobSF_Lab_Edition`
- Exemple : `MobSF_Lab_Edition_MAITOT`
- RAM allouée : 4-6 GB
- Espace disque interne : 2 GB minimum
- Cliquez : `Finish`

<img width="232" height="40" alt="image" src="https://github.com/user-attachments/assets/fb8d56ae-b714-49dc-8cc8-a28c36fd9263" />

#### ✅ Vérification
```bash
# Lister les AVD disponibles
emulator -list-avds

# Vérifier la présence de votre AVD
# Sortie attendue : MobSF_Lab_Edition_MAITO
```
<img width="258" height="57" alt="image" src="https://github.com/user-attachments/assets/9bcad533-19c9-4bd3-aaa5-39bf0b4fb376" />

---

### Phase 2️⃣ : Récupération des Outils MobSF (⏱️ 2 min)

Téléchargez le dépôt officiel MobSF qui contient les scripts d'automatisation :

```bash
# Naviguer vers votre dossier de travail
cd ~/Documents # par exemple

# Cloner le dépôt MobSF
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git

# Accéder au répertoire
cd Mobile-Security-Framework-MobSF
```
<img width="760" height="143" alt="image" src="https://github.com/user-attachments/assets/e5d0ebb9-bef2-49f0-b270-f693d40be49f" />

**💡 Note** : Ce répertoire contient les scripts spécialisés pour démarrer l'AVD avec les permissions rootées.

---

### Phase 3️⃣ : Démarrage de l'Émulateur Rooté (⏱️ 3 min)

#### Sur Linux/macOS :
```bash
# Donner les permissions d'exécution
chmod +x scripts/start_avd.sh

# Lancer le script
./scripts/start_avd.sh MobSF_Lab_Edition_MAITO
```

#### Sur Windows (PowerShell) :
```powershell
# Exécuter avec permissions élevées
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Lancer le script
.\scripts\start_avd.ps1 MobSF_Lab_Edition_MAITO
```

**⏳ Durée d'attente** : 30-60 secondes pour le démarrage complet.

<img width="848" height="158" alt="image" src="https://github.com/user-attachments/assets/82b4634e-adf0-43cf-a983-4f1f2ab30b91" />
<img width="835" height="98" alt="image" src="https://github.com/user-attachments/assets/45eef166-060a-4342-a65a-05ec14220e8e" />


#### Vérifier le statut de l'émulateur
```bash
# Dans un nouveau terminal
adb devices -l

# Sortie attendue :
# emulator-5554          device product:sdk_google_phone_x86_64 ...
```

**Conservez l'ID de l'émulateur** (ex : `emulator-5554`) → vous l'utiliserez dans la phase d'installation MobSF.
<img width="258" height="57" alt="image" src="https://github.com/user-attachments/assets/3fdebed3-6fbe-4448-bab6-455c5703ceff" />

---

## 🚀 Installation et Déploiement de MobSF

### Phase 4️⃣ : Extraction et Exécution du Container MobSF (⏱️ 5 min)

MobSF s'exécute dans un conteneur Docker pour l'isolation et la portabilité.

#### Étape 4.1 : Télécharger l'image MobSF
```bash
# Télécharger l'image Docker MobSF (environ 2-3 GB)
docker pull opensecurity/mobile-security-framework-mobsf:latest
```
<img width="660" height="253" alt="image" src="https://github.com/user-attachments/assets/ea033276-40c8-4a0c-b339-32f7dfda39b2" />

#### Étape 4.2 : Lancer le conteneur avec la bonne configuration

⚠️ **IMPORTANT** : Remplacez `emulator-5554` par l'ID exact obtenu à l'étape 3.

```bash
docker run -it --rm \
  --name mobsf_instance \
  -p 8000:8000 \
  -e MOBSF_ANALYZER_IDENTIFIER=emulator-5554 \
  opensecurity/mobile-security-framework-mobsf:latest
```
<img width="850" height="425" alt="image" src="https://github.com/user-attachments/assets/b1f253e0-101f-445c-9c82-35d4069d9db2" />
<img width="860" height="187" alt="image" src="https://github.com/user-attachments/assets/03e9e0dc-a6d1-48dd-9ebd-cb94ba0bbe8b" />

**Explication des paramètres** :
- `-it` : Mode interactif avec terminal
- `--rm` : Supprime le conteneur après arrêt
- `-p 8000:8000` : Mappe le port 8000 (web interface) vers localhost
- `-e MOBSF_ANALYZER_IDENTIFIER` : Spécifie l'émulateur cible pour l'analyse dynamique

#### Étape 4.3 : Accès à l'interface web
1. Ouvrez votre navigateur
2. Allez à : **http://localhost:8000**

<img width="959" height="539" alt="image" src="https://github.com/user-attachments/assets/05bf0827-db5a-4b39-86b1-223b8da3e2bb" />

3. Identifiants par défaut :
   - 👤 **Utilisateur** : `mobsf`
   - 🔑 **Mot de passe** : `mobsf`

<img width="959" height="539" alt="image" src="https://github.com/user-attachments/assets/a88e4780-030e-43cf-8fe9-43b3b83a110d" />

---

## 📊 Procédure d'Analyse

### Phase 5️⃣ : Préparation de l'APK Cible (DIVA) (⏱️ 3 min)

#### Option A : Télécharger la version précompilée (⭐ Recommandé)
```bash
# Sur le site officiel Payatu
# URL : http://www.payatu.com/damn-insecure-and-vulnerable-app/

# Ou via GitHub directement
wget https://github.com/payatu/diva-android/releases/download/[version]/DivaDebug.apk
# Ou clonez et compilez :
git clone https://github.com/payatu/diva-android.git
cd diva-android
./gradlew assembleDebug
```

**📁 Localisation** : Placez le fichier `DIVA-debug.apk` dans un dossier accessible (ex : `~/Downloads/`)

#### Option B : Construire depuis les sources (⏱️ 10 min supplémentaires)
```bash
# Cloner le dépôt DIVA
git clone https://github.com/payatu/diva-android.git

# Naviguer dans le dossier
cd diva-android

# Compiler l'APK debug
./gradlew assembleDebug

# L'APK généré se trouve dans :
# app/build/outputs/apk/debug/app-debug.apk
```

<img width="581" height="122" alt="image" src="https://github.com/user-attachments/assets/75ef845e-94d0-4693-bf88-fbea25dffb5f" />

---

### Phase 6️⃣ : Analyse Statique et Dynamique Complète (⏱️ 15 min)

#### Étape 6.1 : Téléverser et analyser l'APK
1. Dans MobSF (http://localhost:8000), cliquez sur `Upload & Analyze`
2. Sélectionnez le fichier `DIVA-debug.apk`
3. Cliquez `Upload`

**Processus automatisé** :
- Extraction du manifest Android
- Identification des permissions dangereuses
- Analyse du code source (décompilation)
- Extraction des strings et ressources
- Détection des vulnérabilités connues

#### Étape 6.2 : Examiner le rapport statique
Le rapport comprend :
- 📋 **Manifest Analysis** : Permissions, composants, intent filters
- 🔐 **Permissions** : Classification HIGH/MEDIUM/LOW
- 💾 **Hardcoded Secrets** : Clés API, tokens, mots de passe en dur
- 📦 **Dépendances** : CVE des librairies tierces
- 🔍 **Analyse de code** : Anti-patterns de sécurité

<img width="959" height="539" alt="image" src="https://github.com/user-attachments/assets/559019e9-5634-4a11-b11c-c223c0fd6962" />

#### Étape 6.3 : Activer l'analyse dynamique
1. Dans le rapport statique, recherchez le bouton `Dynamic Analyzer` (ou `Start Dynamic Analysis`)
2. Cliquez pour initier l'analyse en temps réel

**Actions automatisées par MobSF** :
```
✓ Installation de DIVA sur l'émulateur (via adb install)
✓ Démarrage du serveur Frida
✓ Configuration du proxy HTTPS global
✓ Initialisation des hooks de monitoring
✓ Lancement de l'interface d'analyse dynamique
```

<img width="959" height="539" alt="image" src="https://github.com/user-attachments/assets/76630fad-cfd3-4896-8dfe-aae0ba61e0d5" />

<img width="959" height="539" alt="image" src="https://github.com/user-attachments/assets/257549c7-2038-42df-acc1-4bc287f9d3a6" />

#### Étape 6.4 : Exploration en temps réel dans l'émulateur
```
📱 Sur l'écran de l'émulateur :

1. L'application DIVA s'ouvre automatiquement
2. Explorez les 13 challenges vulnérables :
   - Insecure Logging
   - Hardcoded Secrets
   - Insecure Data Storage
   - Access Control Issues
   - Insecure Intents
   - SQLite Injection
   - Input Validation Issues
   - et autres...

3. Pour chaque challenge :
   - Lisez la description
   - Effectuez l'action proposée
   - MobSF capture tout en arrière-plan
```

<img width="367" height="531" alt="image" src="https://github.com/user-attachments/assets/53ee25c7-0cb3-4c65-8868-927510cad17e" />

---

## 🔬 Exploitation Pratique

### Phase 7️⃣ : Monitoring en Temps Réel

<img width="958" height="533" alt="image" src="https://github.com/user-attachments/assets/94f0bb1c-34f8-44af-b302-30a1f6204f3e" />

#### A. Logs Runtime
```
MobSF Console → Runtime Logs

Affichage en live :
- Messages de debug (System.out.println)
- Exceptions levées
- Flux d'exécution de l'application
- Données sensibles écrites en log
```

**Exemple vulnérable DIVA** :
```
Insecure Logging Challenge
→ L'app log les credentials en clair
→ MobSF capture : "[SENSITIVE] Username: user123, Password: mypass123"
```

#### B. Interception du Trafic Réseau
```
MobSF → Network Traffic Tab

Capture automatique :
- Requêtes HTTP/HTTPS
- Corps de requête et réponse
- Headers (cookies, authentification)
- Certificats SSL/TLS
```

**Configuration proxy** :
- MobSF agit comme man-in-the-middle transparent
- Même les connexions SSL sont décryptées et affichées
- Aucune configuration manuelle requise

#### C. Injection Dynamique avec Frida

```
MobSF → Frida Tab

Cliquez : "Spawn & Inject"
```

<img width="306" height="65" alt="image" src="https://github.com/user-attachments/assets/ba764c25-c9a3-47d5-b55d-6c86178b803a" />

**Exemples d'exploitation** :

```javascript
// Hook 1 : Intercepter les lectures de SharedPreferences
Java.perform(function() {
    var SharedPref = Java.use('android.content.SharedPreferences');
    var Editor = Java.use('android.content.SharedPreferences$Editor');
    
    Editor.putString.implementation = function(key, value) {
        console.log("[INTERCEPTED] SharedPref SET: " + key + " = " + value);
        return this.putString(key, value);
    };
});

// Hook 2 : Bypass de vérification de signature
Java.perform(function() {
    var System = Java.use('java.lang.System');
    System.getProperty.implementation = function(prop) {
        console.log("[CHECK] Property requested: " + prop);
        return this.getProperty(prop);
    };
});

// Hook 3 : Logging des appels HTTP
Java.perform(function() {
    var HttpClient = Java.use('javax.net.ssl.HttpsURLConnection');
    HttpClient.setRequestProperty.implementation = function(key, value) {
        console.log("[NETWORK] Header: " + key + " = " + value);
        return this.setRequestProperty(key, value);
    };
});
```

#### D. Exploration du Système de Fichiers
```
MobSF → File Monitor

Affichage :
- Fichiers créés par l'application
- Localisation : /data/data/com.payatu.diva/...
- Permissions et propriétaires
- Contenu sensible en clair
```

**Exemple DIVA - Insecure Data Storage** :
```
Fichier découvert : /data/data/com.payatu.diva/shared_prefs/default.xml

Contenu (en clair !) :
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="username">hardcoded_user</string>
    <string name="password">hardcoded_password</string>
    <string name="api_key">sk-12345678</string>
</map>

⚠️ VULNÉRABILITÉ : Données critiques sans chiffrement
```

#### E. Monitoring des Intents
```
MobSF → Intent Monitor

Affichage :
- Intents implicites envoyés
- Action et données transmises
- Destinataires potentiels
- Failles IDOR (Insecure Direct Object References)
```

---

<img width="639" height="535" alt="image" src="https://github.com/user-attachments/assets/5a6f608d-57bf-470e-b772-9ce0cd2de5e5" />

---

## 📈 Workflow Complet d'Analyse

```
┌─────────────────────────────────────────────────────┐
│ 1. TÉLÉVERSEMENT APK                                │
│    └─ Analyse statique automatique                 │
├─────────────────────────────────────────────────────┤
│ 2. RAPPORT STATIQUE                                 │
│    ├─ Permissions dangereuses identifiées          │
│    ├─ Secrets en dur détectés                      │
│    └─ Dépendances vulnérables listées              │
├─────────────────────────────────────────────────────┤
│ 3. ACTIVATION DYNAMIQUE                             │
│    ├─ Installation APK sur émulateur               │
│    ├─ Démarrage Frida Server                       │
│    └─ Configuration proxy HTTPS                    │
├─────────────────────────────────────────────────────┤
│ 4. MONITORING EN TEMPS RÉEL                         │
│    ├─ Logs runtime                                 │
│    ├─ Trafic réseau                                │
│    ├─ Accès fichiers                               │
│    └─ Injection Frida                              │
├─────────────────────────────────────────────────────┤
│ 5. RAPPORT CONSOLIDÉ                                │
│    ├─ Vulnérabilités découvertes                  │
│    ├─ Risque évalué (HIGH/MEDIUM/LOW)            │
│    └─ Export PDF/JSON                              │
└─────────────────────────────────────────────────────┘
```

---

## 🔧 Dépannage et Optimisations

### ❌ Problème : L'émulateur ne démarre pas

```bash
# Solution 1 : Vérifier la virtualisation
# Linux
grep -c 'vmx\|svm' /proc/cpuinfo  # Résultat > 0 = OK

# Solution 2 : Augmenter la RAM allouée
# Éditez ~/.android/avd/MobSF_Lab_Edition_DUPONT/config.ini
hw.ramSize=6144

# Solution 3 : Démarrer en mode headless (sans interface graphique)
emulator -avd MobSF_Lab_Edition_DUPONT -no-window -gpu swiftshader_indirect
```

### ❌ Problème : MobSF ne détecte pas l'émulateur

```bash
# Vérifier les appareils connectés
adb devices

# Si absent : redémarrer le serveur ADB
adb kill-server
adb start-server
adb devices

# Vérifier l'ID exact et le réutiliser dans Docker
docker run -it --rm \
  -p 8000:8000 \
  -e MOBSF_ANALYZER_IDENTIFIER=emulator-XXXX \
  opensecurity/mobile-security-framework-mobsf:latest
```

### ❌ Problème : Port 8000 déjà utilisé

```bash
# Libérer le port
# Linux/macOS
lsof -i :8000 | grep LISTEN | awk '{print $2}' | xargs kill -9

# Windows PowerShell
Get-Process | Where-Object {$_.Handles -like "*8000"} | Stop-Process

# Ou utiliser un autre port
docker run -it --rm \
  -p 9000:8000 \
  -e MOBSF_ANALYZER_IDENTIFIER=emulator-5554 \
  opensecurity/mobile-security-framework-mobsf:latest

# Accès : http://localhost:9000
```

### ⚡ Optimisation : Améliorer les Performances

```bash
# 1. Allouer plus de cœurs à l'émulateur
# ~/.android/avd/MobSF_Lab_Edition_DUPONT/config.ini
hw.cpu.ncore=4

# 2. Activer la GPU acceleration
hw.gpu.enabled=true
hw.gpu.mode=host

# 3. Réduire les services système (plus rapide)
qemu.disable_winsys=true

# 4. Démarrer avec snapshot pré-compilé
emulator -avd MobSF_Lab_Edition_DUPONT -snapshot default
```

---

## 📊 Rapport Final et Export

### Générer un Rapport Consolidé

```
MobSF Interface → Report Button (en haut à droite)

Options disponibles :
1. PDF Report  ← Lisible, parfait pour documentation
2. JSON Export ← Parsable, idéal pour scripts
3. HTML Report ← Visualisation interactive
```
Exemples de logs :

<img width="657" height="539" alt="image" src="https://github.com/user-attachments/assets/5d3123f1-7ec5-47e6-bf18-981aadca8ac6" />

<img width="956" height="539" alt="image" src="https://github.com/user-attachments/assets/4b7948a3-2d98-40e0-8ab4-9ef71ea656c1" />

<img width="959" height="539" alt="image" src="https://github.com/user-attachments/assets/a7c876f1-711a-4d9e-946f-4d471630a041" />


### Structure du Rapport PDF

```
1. RÉSUMÉ EXÉCUTIF
   • Nombre de vulnérabilités critiques
   • Score de sécurité global
   • Recommandations prioritaires

2. ANALYSE STATIQUE
   • Manifest review
   • Permissions
   • Code source (patterns dangereux)
   • Dépendances vulnérables

3. ANALYSE DYNAMIQUE
   • Trafic réseau capturé
   • Accès fichiers suspects
   • Logs sensibles
   • Appels API critiques

4. PREUVES DE CONCEPT (PoC)
   • Screenshots des vulnérabilités
   • Walkthroughs d'exploitation
   • Données extractibles

5. RECOMMANDATIONS
   • Par ordre de criticité
   • Actions correctives détaillées
```

---

## 📚 Ressources Complémentaires

- **Documentation MobSF** : https://mobsf.github.io/mobile-security-framework-mobsf/
- **DIVA Challenges** : https://github.com/payatu/diva-android/wiki
- **OWASP Mobile Top 10** : https://owasp.org/www-project-mobile-top-10/
- **Frida Documentation** : https://frida.re/docs/home/

---

## ✅ Checklist de Complétion

- [ ] AVD créé et nommé `MobSF_Lab_Edition_MAITO` ( par exemple )
- [ ] Émulateur démarre sans erreur
- [ ] `adb devices` affiche l'émulateur en ligne
- [ ] MobSF accessible sur http://localhost:8000
- [ ] DIVA APK téléchargé et en local
- [ ] Analyse statique DIVA complétée
- [ ] Analyse dynamique activée avec succès
- [ ] Au moins 5 vulnérabilités identifiées
- [ ] Rapport exporté en PDF
- [ ] Tests Frida réussis

---

*Ce guide est le fruit d'une analyse personnelle de la sécurité mobile et représente une méthodologie unique de testing.*
