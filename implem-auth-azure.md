# Manual Implementação Auth Azure

### **Guia Técnico Detalhado: Implementação de Autenticação com Terraform e HCP Terraform na Azure**

Este documento descreve **passo a passo** como configurar e aplicar cada uma das soluções de autenticação apresentadas para provisionar infraestrutura com Terraform na Azure Cloud. Ele inclui explicações detalhadas, comandos e exemplos.

---

## **1. Service Principal com Credenciais de Cliente**

### **Objetivo**

Criar um Service Principal no Azure Active Directory (AAD) e configurá-lo no Terraform para autenticação.

---

### **Passo 1: Criar um Service Principal**

1. Acesse a Azure CLI.
2. Execute o comando para criar o Service Principal:
    
    ```bash
    az ad sp create-for-rbac --name "TerraformSP" --role Contributor --scopes /subscriptions/<subscription-id>
    
    ```
    
    - Substitua `<subscription-id>` pelo ID da assinatura.
    - O parâmetro `-role` define as permissões. Utilize `Reader`, `Contributor` ou uma função personalizada.
3. Exemplo de saída:
    
    ```json
    {
      "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "displayName": "TerraformSP",
      "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    
    ```
    
    - `appId` → `ARM_CLIENT_ID`.
    - `password` → `ARM_CLIENT_SECRET`.
    - `tenant` → `ARM_TENANT_ID`.

---

### **Passo 2: Configurar Variáveis no Terraform**

1. Crie um arquivo `variables.tf`:
    
    ```hcl
    variable "client_id" {}
    variable "client_secret" {}
    variable "tenant_id" {}
    variable "subscription_id" {}
    
    ```
    
2. Crie um arquivo `terraform.tfvars`:
    
    ```hcl
    client_id       = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    client_secret   = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    tenant_id       = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    subscription_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    
    ```
    

---

### **Passo 3: Configurar o Provider no Terraform**

1. No arquivo principal (`main.tf`), configure o provider:
    
    ```hcl
    provider "azurerm" {
      features {}
    
      client_id       = var.client_id
      client_secret   = var.client_secret
      tenant_id       = var.tenant_id
      subscription_id = var.subscription_id
    }
    
    ```
    
2. Teste a configuração:
    
    ```bash
    terraform init
    terraform plan
    
    ```
    

---

## **2. Managed Identity**

### **Objetivo**

Usar uma Identidade Gerenciada para autenticar Terraform em recursos do Azure.

---

### **Passo 1: Habilitar Managed Identity em uma VM**

1. Crie ou edite a VM no Azure para habilitar Managed Identity:
    
    ```bash
    az vm identity assign --name <vm-name> --resource-group <resource-group>
    
    ```
    
    - Substitua `<vm-name>` pelo nome da VM.
    - Substitua `<resource-group>` pelo grupo de recursos.

---

### **Passo 2: Atribuir Permissões à Identidade**

1. Atribua uma função à identidade:
    
    ```bash
    az role assignment create --assignee <principal-id> --role "Contributor" --scope /subscriptions/<subscription-id>
    
    ```
    
    - Use o `principalId` retornado no passo anterior.

---

### **Passo 3: Configurar o Provider no Terraform**

1. Configure o provider para usar a Managed Identity:
    
    ```hcl
    provider "azurerm" {
      features {}
      use_msi = true
    }
    
    ```
    
2. Teste a configuração:
    
    ```bash
    terraform init
    terraform plan
    
    ```
    

---

## **3. Azure CLI Authentication**

### **Objetivo**

Usar a autenticação da CLI do Azure para desenvolvimento local.

---

### **Passo 1: Fazer Login na Azure CLI**

1. Execute:
    
    ```bash
    az login
    
    ```
    
    - Isso abrirá o navegador para autenticação.

---

### **Passo 2: Configurar o Provider no Terraform**

1. Configure o provider para usar o contexto da CLI:
    
    ```hcl
    provider "azurerm" {
      features {}
    }
    
    ```
    
2. Teste a configuração:
    
    ```bash
    terraform init
    terraform plan
    
    ```
    

---

## **4. Azure AD Token Authentication**

### **Objetivo**

Usar tokens OAuth2 para autenticação.

---

### **Passo 1: Obter um Token OAuth2**

1. Execute:
    
    ```bash
    az account get-access-token --resource=https://management.azure.com/
    
    ```
    
    - O token será exibido na saída.

---

### **Passo 2: Configurar o Provider no Terraform**

1. Configure o provider com o token:
    
    ```hcl
    provider "azurerm" {
      features {}
    
      client_id       = var.client_id
      subscription_id = var.subscription_id
      tenant_id       = var.tenant_id
      access_token    = var.access_token
    }
    
    ```
    

---

## **5. Azure Key Vault Integration**

### **Objetivo**

Armazenar credenciais no Azure Key Vault e recuperá-las dinamicamente.

---

### **Passo 1: Criar um Key Vault**

1. Crie o Key Vault:
    
    ```bash
    az keyvault create --name <keyvault-name> --resource-group <resource-group> --location <location>
    
    ```
    

---

### **Passo 2: Armazenar as Credenciais**

1. Adicione segredos ao Key Vault:
    
    ```bash
    az keyvault secret set --vault-name <keyvault-name> --name "client-id" --value "<ARM_CLIENT_ID>"
    az keyvault secret set --vault-name <keyvault-name> --name "client-secret" --value "<ARM_CLIENT_SECRET>"
    
    ```
    

---

### **Passo 3: Recuperar Credenciais no Terraform**

1. Use um script ou módulo para buscar os segredos:
    
    ```hcl
    data "azurerm_key_vault_secret" "client_id" {
      name         = "client-id"
      key_vault_id = "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.KeyVault/vaults/<keyvault-name>"
    }
    
    data "azurerm_key_vault_secret" "client_secret" {
      name         = "client-secret"
      key_vault_id = "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.KeyVault/vaults/<keyvault-name>"
    }
    
    provider "azurerm" {
      features {}
    
      client_id     = data.azurerm_key_vault_secret.client_id.value
      client_secret = data.azurerm_key_vault_secret.client_secret.value
    }
    
    ```
    

---

## **6. HashiCorp Vault**

### **Objetivo**

Gerar credenciais dinâmicas para o Terraform.

---

### **Passo 1: Configurar o Secrets Engine no Vault**

1. Ative o Secrets Engine para Azure:
    
    ```bash
    vault secrets enable azure
    
    ```
    
2. Configure o Azure no Vault:
    
    ```bash
    vault write azure/config \
      subscription_id=<subscription-id> \
      tenant_id=<tenant-id> \
      client_id=<client-id> \
      client_secret=<client-secret>
    
    ```
    
3. Crie uma função para gerar credenciais:
    
    ```bash
    vault write azure/roles/terraform-role \
      ttl=1h \
      application_object_id=<app-id> \
      permissions=Contributor
    
    ```
    

---

### **Passo 2: Configurar o Provider no Terraform**

1. Configure o Terraform para buscar credenciais do Vault:
    
    ```hcl
    provider "azurerm" {
      features {}
    
      client_id       = vault_generic_secret.client_id
      client_secret   = vault_generic_secret.client_secret
    }
    
    data "vault_generic_secret" "azurerm" {
      path = "azure/creds/terraform-role"
    }
    
    ```
    

---

Cada solução descrita aqui pode ser implementada com base no contexto e requisitos de segurança da sua organização. Caso precise de mais detalhes sobre um caso específico, avise!
