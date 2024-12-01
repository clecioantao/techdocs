# **Autenticação Azure**

### **Autenticação para Provisionamento de Infraestrutura com Terraform e HCP Terraform na Azure Cloud**

---

### **Visão Geral**

Este documento descreve as opções de autenticação disponíveis para uso com Terraform e HCP Terraform ao provisionar infraestrutura em uma organização que prioriza segurança da informação, utilizando a Azure Cloud como plataforma.

---

### **Opções de Autenticação**

### **1. Service Principal com Credenciais de Cliente**

### **Descrição**

O Service Principal (SP) é uma identidade criada no Azure Active Directory (AAD) com permissões atribuídas para gerenciar recursos na Azure.

### **Credenciais Necessárias**

- `ARM_CLIENT_ID`: ID do Service Principal.
- `ARM_CLIENT_SECRET`: Segredo associado ao Service Principal.
- `ARM_TENANT_ID`: ID do Tenant no AAD.
- `ARM_SUBSCRIPTION_ID`: ID da assinatura Azure.

### **Vantagens**

- Segregação clara de permissões.
- Suporte nativo no Terraform.
- Pode ser gerenciado por gerenciadores de segredos (Azure Key Vault, HashiCorp Vault).

### **Desvantagens**

- Necessidade de proteger o segredo (`ARM_CLIENT_SECRET`).

### **Exemplo de Configuração no Terraform**

```hcl
hcl
Copiar código
provider "azurerm" {
  features {}

  client_id       = var.client_id
  client_secret   = var.client_secret
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}

```

---

### **2. Managed Identity**

### **Descrição**

As Managed Identities são identidades gerenciadas pela Azure atribuídas a recursos, como VMs ou containers, eliminando a necessidade de armazenar credenciais.

### **Tipos**

- **System-assigned:** Identidade gerada automaticamente e vinculada a um recurso específico.
- **User-assigned:** Identidade reutilizável que pode ser associada a vários recursos.

### **Vantagens**

- Elimina segredos ou credenciais armazenadas.
- Seguro e integrado nativamente na Azure.

### **Desvantagens**

- Requer execução do Terraform em recursos suportados pela Azure.

### **Exemplo de Configuração no Terraform**

```hcl
hcl
Copiar código
provider "azurerm" {
  features {}
  use_msi = true
}

```

---

### **3. Azure CLI Authentication**

### **Descrição**

O Terraform utiliza o contexto autenticado da Azure CLI no dispositivo onde está sendo executado.

### **Vantagens**

- Simples para testes e desenvolvimento local.
- Não requer configuração adicional.

### **Desvantagens**

- Não recomendado para automação em pipelines CI/CD.
- Menos seguro para ambientes compartilhados.

### **Exemplo de Configuração no Terraform**

```hcl
hcl
Copiar código
provider "azurerm" {
  features {}
}

```

---

### **4. Azure AD Token Authentication**

### **Descrição**

Usa tokens OAuth2 gerados pelo Azure AD para autenticar no Terraform.

### **Vantagens**

- Ideal para sistemas que já gerenciam tokens.
- Dispensa o uso de segredos permanentes.

### **Desvantagens**

- Requer renovação periódica dos tokens.
- Complexo de configurar.

### **Exemplo de Configuração no Terraform**

```hcl
hcl
Copiar código
provider "azurerm" {
  features {}
  client_id       = var.client_id
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
  access_token    = var.access_token
}

```

---

### **5. Azure Key Vault Integration**

### **Descrição**

Armazena as credenciais no Azure Key Vault e recupera dinamicamente durante a execução do Terraform.

### **Vantagens**

- Maior segurança no armazenamento de credenciais.
- Suporte à rotação de segredos.

### **Desvantagens**

- Requer configuração adicional para integração.

### **Exemplo**

- Use um módulo ou script para buscar os segredos no Key Vault e passá-los ao Terraform.

---

### **6. HashiCorp Vault com Secrets Engine para Azure**

### **Descrição**

O HashiCorp Vault pode ser configurado para gerar credenciais dinâmicas para o Azure.

### **Vantagens**

- Credenciais com tempo de vida limitado.
- Controle centralizado sobre permissões e acesso.

### **Desvantagens**

- Necessidade de configurar e gerenciar o Vault.

### **Exemplo**

O Terraform pode buscar credenciais diretamente do Vault utilizando o `vault` provider.

---

### **7. HCP Vault Integration**

### **Descrição**

Se estiver usando o HCP Vault, ele pode ser integrado com o Terraform para gerenciamento dinâmico de credenciais e segredos.

### **Vantagens**

- Semelhante ao HashiCorp Vault, mas com gerenciamento simplificado.

### **Desvantagens**

- Necessário possuir o HCP Vault configurado.

---

### **Recomendações**

- **Produção:** Prefira **Managed Identity** ou **Service Principal** com credenciais armazenadas no Azure Key Vault.
- **Desenvolvimento:** Utilize **Azure CLI Authentication** para testes locais.
- **Automação em Pipelines:** Utilize **Managed Identity** para maior segurança ou **Service Principal** gerenciado pelo Vault/Key Vault.

---

### **Tabela Comparativa**

| Opção | Segurança | Facilidade de Configuração | Automação | Recomendado para |
| --- | --- | --- | --- | --- |
| Service Principal | Alta | Moderada | Sim | Produção |
| Managed Identity | Alta | Alta | Sim | Produção |
| Azure CLI Authentication | Baixa | Alta | Não | Desenvolvimento |
| Azure AD Token | Alta | Baixa | Sim | Integrações |
| Azure Key Vault | Alta | Moderada | Sim | Produção |
| HashiCorp Vault | Alta | Moderada | Sim | Produção |
| HCP Vault | Alta | Alta | Sim | Produção |

---

### **Conclusão**

Escolha a opção de autenticação com base nos requisitos de segurança, facilidade de uso e automação. Implementar soluções como Managed Identity ou Service Principal em conjunto com Key Vault garante segurança elevada para ambientes corporativos.
