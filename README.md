# relazione_terraform_azure_devops
**Gestione dell’infrastruttura Azure con Terraform e Azure DevOps**

**Introduzione**

Negli ultimi anni, la gestione dell’infrastruttura IT ha subito una trasformazione radicale grazie all’introduzione dell’approccio *Infrastructure as Code* (IaC). Questa metodologia consente agli amministratori di sistema e agli sviluppatori di definire e gestire l’infrastruttura attraverso il codice, abbandonando le configurazioni manuali a favore di processi automatizzati, ripetibili e tracciabili. Il risultato è una maggiore coerenza tra gli ambienti, una riduzione degli errori umani e un controllo più rigoroso sulle modifiche apportate.

Uno degli strumenti più popolari per l’IaC è **Terraform**, un software open-source sviluppato da HashiCorp, che consente di descrivere l’infrastruttura desiderata in un linguaggio dichiarativo chiamato HCL (HashiCorp Configuration Language). Attraverso l’utilizzo di *provider*, Terraform è in grado di interfacciarsi con le API di diversi servizi cloud. In ambito Microsoft Azure, Terraform comunica con le API Azure Resource Manager (ARM), consentendo la creazione, la modifica e la distruzione di risorse in maniera programmatica.

Questa relazione si propone di esplorare i principali provider Terraform disponibili per Azure, le modalità di autenticazione sicura verso il cloud, l’integrazione di Terraform all’interno di pipeline DevOps automatizzate e le best practice da seguire per mantenere il codice infrastrutturale robusto e sicuro. Infine, viene presentato un esempio pratico per la realizzazione di una Web App su Azure, interamente gestita tramite Terraform e pipeline CI/CD in Azure DevOps.

**Provider Terraform per Azure**

Nel contesto di Terraform, i *provider* sono moduli fondamentali che definiscono il modo in cui Terraform interagisce con ciascun tipo di servizio esterno. In pratica, un provider è un ponte tra il linguaggio dichiarativo di Terraform e le API del servizio di destinazione. Nel caso di Azure, esistono diversi provider ufficiali e non ufficiali, ciascuno specializzato in determinati ambiti.

**AzureRM (Azure Resource Manager)**

Il provider azurerm è il principale strumento di interazione tra Terraform e Azure. È utilizzato per gestire la quasi totalità delle risorse Azure: dai gruppi di risorse alle macchine virtuali, dalle reti virtuali ai database e ai servizi web. Questo provider è il più maturo e stabile tra quelli disponibili per Azure, ed è regolarmente aggiornato per supportare le nuove funzionalità offerte dalla piattaforma.

La configurazione base del provider azurerm richiede la definizione del provider stesso all’interno di un blocco terraform, indicando la versione desiderata, seguita da un blocco provider che definisce eventuali opzioni. Un esempio tipico include la creazione di un gruppo di risorse in una specifica regione geografica. Questo tipo di risorsa rappresenta l’unità logica base in cui vengono organizzate le risorse Azure.

**Esempio di configurazione:**
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm" # Specifica da dove scaricare il provider
      version = "~> 3.0"             # Usa una versione 3.x del provider
    }
  }
}

provider "azurerm" {
  features {} # Configurazione richiesta dal provider (qui vuota)
}

resource "azurerm_resource_group" "example" {
  name     = "example-resources"  # Nome del gruppo di risorse
  location = "West Europe"        # Regione geografica
}
```


**AzureAD (Azure Active Directory)**

Azure Active Directory è il servizio di identità e accesso cloud-based di Microsoft. Terraform consente di interagire con questo servizio tramite il provider azuread. Grazie a questo provider è possibile automatizzare la gestione di utenti, gruppi, ruoli e applicazioni, migliorando la sicurezza e riducendo la necessità di interventi manuali nei portali web.

Questo provider è utile, ad esempio, in scenari dove è necessario creare dinamicamente account utente per nuovi ambienti, assegnare ruoli o registrare nuove applicazioni all'interno dell'organizzazione.

**Esempio:**

```hcl
provider "azuread" {
  tenant_id = "<tenant_id>" # ID del tenant di Azure Active Directory
}

resource "azuread_user" "example" {
  user_principal_name = "utente@example.com" # Email di accesso dell'utente
  display_name        = "Utente Esempio"     # Nome visualizzato
  password            = "Password123!"       # Password iniziale (meglio usare una variabile sicura)
}
```

**Azure DevOps**

Il provider azuredevops permette di gestire componenti relativi alla piattaforma Azure DevOps, come progetti, repository Git, pipeline di build e deployment, e permessi. Azure DevOps è una suite di strumenti Microsoft che supporta metodologie Agile e DevOps, facilitando la collaborazione tra sviluppatori e operatori.

Utilizzando questo provider, è possibile automatizzare la creazione e la configurazione di un progetto DevOps, creando ad esempio un repository Git o definendo una pipeline CI/CD. Questo approccio consente di trattare anche gli strumenti DevOps come codice, mantenendo la coerenza e il controllo completo dell’intero stack.

**Esempio:**

```hcl
provider "azuredevops" {
  org_service_url       = "https://dev.azure.com/<organizzazione>" # URL dell'organizzazione Azure DevOps
  personal_access_token = var.pat                                   # Token di accesso personale (PAT) definito come variabile
}

resource "azuredevops_project" "example" {
  name        = "TerraformProject"                     # Nome del progetto DevOps da creare
  description = "Progetto gestito con Terraform"       # Descrizione del progetto
}
```


**AzAPI**

Non tutte le risorse Azure sono immediatamente disponibili nel provider azurerm, specialmente quando si tratta di funzionalità nuove, in anteprima o particolarmente avanzate. Per questi casi, Microsoft mette a disposizione il provider azapi, che consente di interfacciarsi direttamente con le REST API di Azure. Questo provider è molto potente e flessibile, ma richiede una maggiore comprensione delle strutture JSON delle API REST e delle specifiche di ogni risorsa.

Un aspetto interessante di azapi è che, a differenza di azurerm, non ha bisogno di aspettare l'implementazione di nuove risorse da parte del team di Terraform, poiché consente di specificare direttamente il tipo di risorsa e la versione API da usare.

**API** significa **Application Programming Interface**: è un **insieme di regole** che permettono a due programmi/software di **comunicare tra loro**.

**REST** significa **REpresentational State Transfer**: è **uno stile** di progettazione delle API, basato su **regole precise**, molto usato sul web.

Se un'API è **REST**, significa che:

- Usa **HTTP** (proprio come il web: GET, POST, PUT, DELETE).
- Si basa su **risorse** (es.: utenti, prodotti, ordini).
- Ogni risorsa ha un **URL** unico.
- Le operazioni sono standardizzate

**Esempio:**

```hcl
provider "azapi" {
  # Inizializza il provider AzAPI, usato per accedere direttamente alle REST API di Azure
  # Utile per risorse non ancora supportate dal provider "azurerm"
}

resource "azapi_resource" "custom" {
  type     = "Microsoft.Resources/resourceGroups@2021-04-01" # Tipo di risorsa e versione API
  name     = "azapi-rg"                                      # Nome della risorsa (resource group)
  location = "West Europe"                                   # Località geografica
  parent_id = "/subscriptions/<subscription-id>"             # ID della sottoscrizione Azure
}
```


**Azure Stack**

Per le organizzazioni che lavorano in ambienti ibridi, ovvero che utilizzano una combinazione di cloud pubblico e infrastrutture locali, Microsoft offre Azure Stack. Il provider dedicato consente di gestire risorse su queste piattaforme locali mantenendo la compatibilità con i modelli Azure. È molto utilizzato in contesti con requisiti normativi o di sicurezza stringenti, come enti governativi o sanitari.

**Metodi di Autenticazione con Azure**

Per poter interagire con Azure in modo sicuro, Terraform deve autenticarsi. I metodi principali di autenticazione sono:

- **Azure CLI**: metodo pratico per sviluppo locale. Richiede l’esecuzione del comando az login, che apre una sessione autenticata valida per tutte le operazioni successive.
- **Service Principal**: il metodo più usato in ambienti automatizzati (CI/CD). Prevede la creazione di una "identità applicativa" a cui vengono assegnati permessi specifici su risorse Azure. L’autenticazione può avvenire tramite *client secret* o certificato.
- **Managed Identity**: disponibile per le risorse Azure come le VM o le pipeline DevOps. Questa identità viene gestita da Azure e permette l’autenticazione senza necessità di gestire manualmente le credenziali.
- **OIDC (OpenID Connect)**: usato principalmente con GitHub Actions e Terraform Cloud. È un meccanismo moderno e sicuro che consente a provider esterni di autenticarsi verso Azure senza usare segreti statici.

**Pipeline CI/CD in Azure DevOps con Terraform**

Uno dei punti di forza di Terraform è la sua capacità di integrarsi perfettamente in processi DevOps. Utilizzando Azure DevOps è possibile creare pipeline CI/CD (Continuous Integration / Continuous Deployment) per automatizzare l’intero ciclo di vita dell’infrastruttura.

**Funzionamento della pipeline**

Una pipeline definita in Azure DevOps per Terraform esegue una serie di passaggi, tra cui:

1. **Clonazione del codice** da un repository Git, ad esempio Azure Repos o GitHub.
1. **Installazione di Terraform** nell’agente che esegue la pipeline.
1. **Inizializzazione** dell’ambiente con terraform init, che prepara la directory di lavoro e scarica i provider.
1. **Validazione** della configurazione con terraform validate, per rilevare errori sintattici o logici.
1. **Creazione del piano di esecuzione** con terraform plan, che mostra cosa accadrà nell’infrastruttura senza apportare modifiche.
1. **Applicazione delle modifiche** con terraform apply, che realizza i cambiamenti previsti. In ambienti di produzione, questa fase può essere soggetta ad approvazione manuale.

**Definizione YAML della pipeline**

L’approccio più moderno e flessibile consiste nella definizione della pipeline tramite un file azure-pipelines.yml, versionato insieme al codice. Di seguito, un esempio di base:

**Esempio di pipeline YAML**

```yaml
trigger:
  branches:
    include:
      - main  # La pipeline si attiva sui commit al branch "main"

pool:
  vmImage: 'ubuntu-latest'  # Macchina virtuale dell'agente

steps:
  - checkout: self  # Clona il repository

  - task: UseTerraform@0
    inputs:
      terraformVersion: '1.5.0'  # Versione di Terraform

  - script: terraform init
    displayName: 'Terraform Init'  # Inizializza la directory di lavoro

  - script: terraform validate
    displayName: 'Terraform Validate'  # Controlla la validità dei file

  - script: terraform plan -out=tfplan
    displayName: 'Terraform Plan'  # Genera il piano di esecuzione

  - script: terraform apply tfplan
    displayName: 'Terraform Apply'  # Applica le modifiche pianificate
```

**Scenario: Provisioning di un'infrastruttura base per un sito web aziendale**

Supponiamo di dover creare un'infrastruttura Azure per ospitare un'applicazione web. Utilizzeremo Terraform per creare:

- Un gruppo di risorse
- Un piano di servizio App Service
- Una Web App

Codice Terraform di esempio:

```hcl
provider "azurerm" {
  features {} # Obbligatorio per inizializzare il provider azurerm
}

resource "azurerm_resource_group" "rg" {
  name     = "webapp-rg"         # Nome del gruppo di risorse
  location = "West Europe"       # Regione Azure dove sarà creato
}

resource "azurerm_app_service_plan" "asp" {
  name                = "webapp-plan"                    # Nome del piano di servizio
  location            = azurerm_resource_group.rg.location # Stessa località del resource group
  resource_group_name = azurerm_resource_group.rg.name   # Collegamento al gruppo di risorse

  sku {
    tier = "Standard"  # Tier del piano (produzione)
    size = "S1"        # Dimensione dell'istanza
  }
}

resource "azurerm_app_service" "webapp" {
  name                = "mywebappdemo123"                # Nome univoco della Web App
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  app_service_plan_id = azurerm_app_service_plan.asp.id
}
```

Questo esempio mostra come creare un'applicazione web base su Azure con pochi blocchi Terraform. Una pipeline DevOps potrebbe poi automatizzare il deploy e il monitoraggio continuo.

**Best Practices**

Per garantire qualità e sicurezza, è consigliabile seguire alcune buone pratiche:

- **Formattare il codice** con terraform fmt
- **Validare la configurazione** con terraform validate
- **Separare gli ambienti** (dev, test, prod) usando directory o workspaces
- **Gestire le credenziali** come variabili sicure, mai in chiaro
- **Riutilizzare il codice** organizzandolo in moduli
- **Documentare le configurazioni** per facilitare manutenzione e collaborazione

**Conclusione**

Terraform rappresenta una soluzione potente, flessibile e ampiamente adottata per la gestione dell’infrastruttura su Azure. L’integrazione con provider specifici come AzureRM, AzureAD, AzAPI e Azure DevOps consente un controllo granulare e centralizzato di ogni aspetto dell’ambiente cloud. Unita a pipeline automatizzate in Azure DevOps, questa combinazione permette alle organizzazioni di modernizzare la propria gestione IT, riducendo i rischi, aumentando la produttività e assicurando un alto livello di sicurezza e coerenza.
