# DevOps_03_IaC
Repozitoř k 3. lekci

# DevOps Úkol 03: AWS IAM, Bezpečnost a CloudFormation (IaC)

Tento repozitář obsahuje řešení domácího úkolu zaměřeného na bezpečné nastavení AWS účtu, konfiguraci AWS CLI v prostředí RHEL10 a nasazení automatického hlídání rozpočtu pomocí AWS CloudFormation šablony.

### 🎯 Cíle projektu

* **Zabezpečení Root účtu:** Aktivace vícefaktorového ověření (MFA) pro hlavní AWS účet a jeho bezpečné uzamčení.
* **Správa identit (IAM):** Vytvoření samostatného IAM uživatele s právy `AdministratorAccess` pro běžnou administrátorskou práci (dodržení AWS Best Practices).
* **Konfigurace AWS CLI:** Propojení lokálního prostředí (RHEL10) s AWS pomocí Access Keys pod dedikovaným pojmenovaným profilem.
* **Infrastruktura jako kód (IaC):** Automatizované nasazení AWS Budget (rozpočtu) pomocí CloudFormation šablony.
* **Finanční kontrola:** Nastavení automatických e-mailových notifikací při dosažení 80 % a 99 % definovaného rozpočtu.

### 📂 Struktura projektu

Projekt obsahuje následující klíčové soubory:
* **aws-cf-create-budget.yaml** - CloudFormation šablona definující AWS rozpočet, typy započítávaných nákladů (CostTypes) a parametry pro e-mailové notifikace.
* **README.md** - Tato dokumentace popisující cíle, strukturu a postup nasazení.

🚀 Návod k použití

### 1. Prerekvizity

* Nainstalované AWS CLI na RHEL10.
* Přístup k virtuálnímu stroji přes SSH (MobaXterm / VS Code).
* Vygenerované Access Keys pro IAM uživatele (nikoliv pro Root účet).

### 2. Konfigurace AWS CLI (Pojmenovaný profil)

Pro izolaci školních úkolů byl vytvořen dedikovaný profil `DevOpsLectures`. Konfigurace se spustí příkazem:

```bash
aws configure --profile DevOpsLectures
```

Během konfigurace se zadávají následující hodnoty:
* **AWS Access Key ID** a **AWS Secret Access Key** (získané z IAM sekce uživatele)
* **Default region name:** eu-central-1 (Frankfurt) nebo us-east-1 (N. Virginia)
* **Default output format:** json

Ověření správnosti nastavení a identity:

```Bash
aws sts get-caller-identity --profile DevOpsLectures
```

### 3. Nasazení CloudFormation Stacku
Šablonu rozpočtu lze nasadit buď přímo přes AWS Web Console, nebo efektivněji pomocí AWS CLI z terminálu:

```Bash
aws cloudformation create-stack \
  --stack-name DevOps-lekce3-budget \
  --template-body file://aws-cf-create-budget.yaml \
  --parameters \
      ParameterKey=AccountName,ParameterValue=MujHomeLab \
      ParameterKey=BudgetLimit,ParameterValue=10 \
  --profile DevOpsLectures
```

### 4. Aktualizace rozpočtu za chodu (Update Stack)
V případě změny parametrů (např. úprava e-mailu pro notifikace nebo změna finančního limitu) se šablona aktualizuje bez nutnosti jejího mazání:

```Bash
aws cloudformation update-stack \
  --stack-name DevOps-lekce3-budget \
  --template-body file://aws-cf-create-budget.yaml \
  --parameters \
      ParameterKey=AccountName,ParameterValue=MujHomeLab-NovyEmail \
      ParameterKey=BudgetLimit,ParameterValue=20 \
  --profile DevOpsLectures
```

### 🔒 Bezpečnost

* **Zákaz Root klíčů:** Pro AWS CLI a automatizační nástroje se zásadně nepoužívají přístupové klíče Root účtu. Všechny klíče k Rootu byly deaktivovány a smazány.

* **Utajení klíčů:** Soubory s přístupovými klíči (~/.aws/credentials) jsou uloženy bezpečně na lokálním disku a nikdy nejsou součástí gitu.

* **MFA:** Jak Root účet, tak nově vytvořený IAM administrátorský uživatel mají striktně vynucené přihlašování pomocí hardwarového/virtuálního MFA tokenu.