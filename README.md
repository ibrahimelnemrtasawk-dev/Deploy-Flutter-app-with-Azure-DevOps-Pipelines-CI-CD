# 🚀 Deploying a Flutter App with Azure DevOps Pipelines

Flutter makes it easy to build beautiful apps for Android and iOS — and **Azure DevOps** gives you powerful automation for CI/CD.  
This guide walks you through **deploying a Flutter app using Azure DevOps Pipelines**, step by step.

---

## 🧰 Prerequisites

Before starting, make sure you have:

- ✅ An [Azure DevOps](https://dev.azure.com/) account  
- ✅ A Flutter project (or create one below)  
- ✅ Access to your Google Play Console and/or Apple Developer Account  

---

## 🏗 Step 1: Create a Repository in Azure DevOps

1. Log in to your **Azure DevOps dashboard**
2. From the sidebar, click **Repos → Create Repository**
3. Choose a project name and visibility (Private/Public)
4. Click **Create**


## 💻 Step 2: Clone the Repo and Create Your Flutter Project

On your local machine:

```bash
git clone https://dev.azure.com/<your-organization>/<project>/_git/<repo-name>
cd <repo-name>
flutter create .
```


## 🚀 Step 3: Commit and Push Your Code

```bash
git add .
git commit -m "Initial Flutter project setup"
git push origin main
```


## ⚙️ Step 4: Set Up Your Azure Pipeline YAML File
In your project root (or under an azure folder), create:

```bash
azure-production.yaml
```
This YAML file defines the CI/CD pipeline.


## 🔧 Step 5: Create Library Variables (Flutter & Java Versions)

In Azure DevOps:
1. Go to Pipelines → Library → + Variable Group
2. Add variables like:
   
| Variable Name       | Example Value | Description                    |
| ------------------- | ------------- | ------------------------------ |
| `FLUTTER_VERSION`   | 3.24.3        | Flutter SDK version            |
| `JAVA_VERSION`      | 17            | Java version for Android build |
| `KEY_ALIAS`         | my-key        | Android keystore alias         |
| `KEY_PASSWORD`      | *****         | Key password                   |
| `KEYSTORE_PASSWORD` | *****         | Keystore password              |
| `KEYSTORE_FILE`     | key.jks       | Uploaded secure file           |



## 🧩 Step 6: Define Your Azure Pipeline
Example azure-production.yaml:

```yaml
trigger:
  branches:
    include:
      - main  # or your preferred branch

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: 'FlutterAppVariables'  # link to your variable group

steps:
  - task: UseFlutterVersion@0
    inputs:
      version: '$(FLUTTER_VERSION)'

  - script: flutter pub get
    displayName: 'Install Dependencies'

  - task: JavaToolInstaller@0
    inputs:
      versionSpec: '$(JAVA_VERSION)'
      jdkArchitecture: 'x64'
    displayName: 'Install Java'

  - script: |
      flutter build appbundle --release
    displayName: 'Build Android Release'

  # Sign your Android app
  - task: AndroidSigning@3
    inputs:
      apkFiles: '**/*.aab'
      apksignerKeystoreFile: '$(KEYSTORE_FILE)'
      apksignerKeystorePassword: '$(KEYSTORE_PASSWORD)'
      apksignerKeyAlias: '$(KEY_ALIAS)'
      apksignerKeyPassword: '$(KEY_PASSWORD)'
    displayName: 'Sign Android App'

  # Upload to Google Play
  - task: GooglePlayRelease@4
    inputs:
      serviceConnection: 'GooglePlayServiceConnection'
      action: 'SingleBundle'
      bundleFile: '**/*.aab'
      track: 'production'
    displayName: 'Upload to Google Play'

```


## 🍎 Step 7: iOS Build (Optional)
For iOS, use a macOS agent:

```yaml
pool:
  vmImage: 'macos-latest'

steps:
  - script: flutter build ipa --release
    displayName: 'Build iOS Release'
```
Then configure an App Store Release task to upload to TestFlight or the App Store.


## ✅ Step 8: Run Your Pipeline
Go to:

Pipelines → New Pipeline → Azure Repos Git → Select Repo → YAML file

Select your azure-production.yaml and click Run.

You’ll see logs for each stage (install, build, sign, release).


## 📘 Summary

| Stage      | Description                                     |
| ---------- | ----------------------------------------------- |
| Repo Setup | Created Azure repo and pushed Flutter code      |
| Variables  | Defined versions and secure credentials         |
| Pipeline   | Wrote YAML to automate build, sign, and deploy  |
| CI/CD      | Ran pipeline to build and release automatically |



## ✨ Conclusion

You’ve now built a CI/CD pipeline for Flutter using Azure DevOps.
Every push to main automatically builds, signs, and releases your app.
💡 For production apps, use Azure Key Vault to securely manage credentials.

