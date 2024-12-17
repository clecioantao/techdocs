# Humanitec - Resumo

# **Humanitec**

---

### **Objetivo**

Este documento tem como objetivo apresentar a implantação da solução **Humanitec Platform Orchestrator** no Banco Bradesco, detalhando os benefícios, processos, arquitetura, segurança, e demais componentes necessários para o sucesso da POC.

---

### **Público-alvo**

Arquitetos, Engenheiros de Plataforma, Analistas de Sistemas e Desenvolvedores de Software.

---

## **Índice**

1. **Contexto Humanitec**
    
    1.1 Visão Geral
    
    1.2 Componentes Principais
    
    1.3 Papéis e Responsabilidades
    
    1.4 Processos Operacionais
    
    1.5 Benefícios Esperados
    
    1.6 Documentação e Suporte
    
2. **Modelo de Arquitetura**
3. **Hardening**
4. **Taxonomia**
5. **Responsáveis pelos Componentes**
6. **Segurança de Redes**
    
    6.1 Isolamento de Segredos
    
    6.2 Minimização da Exposição da API
    
    6.3 Abordagem GitOps
    
    6.4 Múltiplos Repositórios de Segredos
    
    6.5 Gerenciamento Centralizado de Segredos
    
    6.6 Integração com Ferramentas de Segurança
    
7. **Logs**
8. **Backup**
9. **Plano de Alta Disponibilidade (HA)**
10. **Monitoramento**
11. **Modelo de Suporte**
12. **Gestão de Acesso**
13. **Matriz RACI**
14. **Estimativa de Custos**

---

## **1. Contexto Humanitec**

### **1.1 Visão Geral**

A **Humanitec** é uma plataforma de orquestração de infraestrutura que simplifica a gestão de workloads e recursos em ambientes multi-cloud e híbridos. Sua principal proposta é fornecer **abstração e automação** para desenvolvedores e engenheiros de infraestrutura.

---

### **1.2 Componentes Principais**

| **Componente** | **Descrição** |
| --- | --- |
| **Workloads** | Aplicações e serviços gerenciados pela Humanitec. |
| **Score** | Ferramenta declarativa para descrever a infraestrutura desejada. |
| **Terraform** | Ferramenta de IaC para provisionamento de infraestrutura. |
| **Custom Drivers** | Drivers personalizados para integração com Ansible e Terraform Enterprise. |
| **Monitoring** | Observabilidade fornecida via **Dynatrace** no contexto do Bradesco. |
| **Humanitec Operator** | CRD que opera nativamente dentro do Kubernetes, automatizando o provisionamento de recursos. |
| **Backstage** | Portal de desenvolvimento self-service para abstrair a solicitação de infraestrutura. |

---

### **1.3 Papéis e Responsabilidades**

| **Papel** | **Responsabilidades** |
| --- | --- |
| **Desenvolvedores** | Criam manifestos *score*, definem infraestrutura e utilizam Humanitec para deploys automatizados. |
| **Engenheiros de Plataforma** | Desenvolvem os módulos Terraform, drivers customizados e definem "resource definitions". |
| **Infraestrutura Digital** | Sustentam e versionam os módulos Terraform no HCP Terraform. |
| **Time de Segurança** | Garantem conformidade com baseline AKS, monitoram exposição de APIs e segurança de segredos. |

---

### **1.4 Processos Operacionais**

**Fluxo Resumido**:

1. **Definição da Infraestrutura**:
    - Desenvolvedores criam manifestos declarativos usando *score*.
    - Módulos Terraform são usados para provisionar os recursos.
2. **Pipeline CI/CD**:
    - Imagens Docker são geradas no **Nexus**.
    - Valores são atualizados via **ArgoCD** e Helm.
3. **Orquestração Humanitec**:
    - Humanitec Operator provisiona automaticamente os recursos no Kubernetes.
4. **Monitoramento e Observabilidade**:
    - Métricas e logs são monitorados via **Dynatrace**.

---

### **1.5 Benefícios Esperados**

- **Eficiência Operacional**: Redução do tempo de deploy.
- **Consistência e Conformidade**: Automação elimina erros manuais.
- **Escalabilidade**: Infraestrutura gerenciada em *stacks*.
- **Segurança**: Uso centralizado de segredos e GitOps.

---

### **1.6 Documentação e Suporte**

- Documentação completa via **Humanitec Docs**.
- Suporte Humanitec + equipe Bradesco.

---

## **2. Modelo de Arquitetura**

*(Diagrama de referência Humanitec no Bradesco aqui)*

---

## **3. Hardening**

1. Restrições RBAC no Kubernetes.
2. Isolamento de segredos com **HashiCorp Vault**.
3. Controle de exposição do cluster via políticas de rede.

---

## **4. Taxonomia**

| **Recurso** | **Descrição** |
| --- | --- |
| Workloads | Aplicações gerenciadas. |
| Score | Declaração de infraestrutura. |
| AKS | Cluster Kubernetes base. |

---

## **5. Responsáveis pelos Componentes**

| **Departamento** | **Equipe Responsável** | **Contato** |
| --- | --- | --- |
| Infraestrutura Digital (4250) | Arnaldo Fagnani | arnaldo.fagnani@bradesco.com.br |

---

## **6. Segurança de Redes**

### 6.1 Isolamento de Segredos

Segurança garantida via **Vault**.

### 6.2 Minimização da Exposição

APIs privadas e baselines AKS garantem exposição mínima.

### 6.3 Abordagem GitOps

Todas as alterações versionadas no **Git**.

---

## **7. Logs**

Logs gerenciados no **Dynatrace** e integrados com baseline AKS.

---

## **8. Backup**

Seguindo políticas de backup do AKS no Bradesco.

---

## **9. Plano de Alta Disponibilidade (HA)**

1. **Infraestrutura distribuída em múltiplos nós**.
2. **AKS com autoescalabilidade habilitada**.

---

## **10. Monitoramento**

Monitoramento ativo via **Dynatrace** com alertas e métricas.

---

## **11. Modelo de Suporte**

- **Interno**: Sustentação pela equipe de Infraestrutura Digital.
- **Humanitec**: Suporte de terceiro nível.

---

## **12. Gestão de Acesso**

RBAC no Kubernetes + políticas de segurança Zero Trust.

---

## **13. Matriz RACI**

| **Atividade** | **R** | **A** | **C** | **I** |
| --- | --- | --- | --- | --- |
| Desenvolvimento Score | Dev | Arq | Eng | Ops |
| Provisionamento Terraform | Eng | Ops | Sec | Dev |

---

## **14. Estimativa de Custos**

| **Componente** | **Custo Estimado** |
| --- | --- |
| Humanitec SaaS | $XXX |
| AKS | $XXX |
| Dynatrace Monitoring | $XXX |

---

## **Estrutura dos Slides**

---

### **Slide 1: Capa**

- **Título:** Implantação da POC Humanitec no Banco Bradesco
- **Subtítulo:** Plataforma de Orquestração de Infraestrutura
- **Responsável:** Equipe de Infraestrutura Digital | 4250
- **Data:** [Inserir data]
- **Imagem:** Logotipo Bradesco + Humanitec

---

### **Slide 2: Objetivo**

- **Título:** Objetivo
- **Conteúdo:**
    - Apresentar a solução **Humanitec Platform Orchestrator**.
    - Demonstrar benefícios e processos operacionais.
    - Detalhar arquitetura, segurança e estrutura de suporte.
- **Imagem:** Ícone representativo de orquestração.

---

### **Slide 3: Visão Geral Humanitec**

- **Título:** O que é a Humanitec?
- **Conteúdo:**
    - Plataforma de **orquestração de workloads e infraestrutura**.
    - Fornece uma **camada de abstração** para desenvolvedores e infraestrutura.
    - Melhora a **experiência do desenvolvedor** com **automação** e simplificação.
- **Imagem:** Fluxo ilustrativo da orquestração Humanitec.

---

### **Slide 4: Principais Motivos para Utilizar**

- **Título:** Principais Motivos para Utilizar a Humanitec
- **Conteúdo (bullets):**
    - Orquestração Unificada
    - Abstração de Infraestrutura com *Score*
    - Melhor Experiência do Desenvolvedor
    - Eficiência para Equipes de Cloud
    - Agnosticismo de Plataforma
    - Automação e Consistência
    - Manutenção Centralizada e Escalabilidade
- **Imagem:** Gráfico ou diagrama simples com os itens destacados.

---

### **Slide 5: Modelo de Arquitetura**

- **Título:** Modelo de Arquitetura Humanitec
- **Conteúdo:**
    - **Developer Control Plane**
    - **Integration & Delivery Plane**
    - **Monitoring & Logging Plane**
    - **Security Plane**
- **Imagem:** Diagrama da arquitetura (compondo os planos).

---

### **Slide 6: Papéis e Responsabilidades**

- **Título:** Papéis e Responsabilidades
- **Tabela:**
    
    
    | **Papel** | **Responsabilidade** |
    | --- | --- |
    | Desenvolvedores | Criam manifestos e utilizam Humanitec para deploys |
    | Engenheiros de Plataforma | Criam módulos Terraform e drivers customizados |
    | Infraestrutura Digital | Sustentam os módulos Terraform no HCP Terraform |

---

### **Slide 7: Processos Operacionais**

- **Título:** Processos Operacionais
- **Fluxo Visual:**
    1. **Definição da Infraestrutura**
    2. **Pipeline CI/CD**
    3. **Orquestração com Humanitec**
    4. **Monitoramento e Logs**
- **Imagem:** Linha do tempo ou fluxograma.

---

### **Slide 8: Segurança de Redes**

- **Título:** Segurança de Redes
- **Conteúdo (bullets):**
    - Isolamento de Segredos com **HashiCorp Vault**
    - Minimização da Exposição da API do Cluster
    - Abordagem **GitOps** com auditoria no Git
    - Gestão Centralizada de Segredos
    - Cluster AKS **Privado** no baseline do Bradesco
- **Imagem:** Ícones de segurança e um diagrama da rede segura.

---

### **Slide 9: Benefícios Esperados**

- **Título:** Benefícios Esperados
- **Conteúdo:**
    - Redução de erros manuais
    - Automação e Consistência
    - Experiência aprimorada do desenvolvedor
    - Gestão centralizada e escalável
    - Segurança robusta integrada
- **Imagem:** Gráfico ou ícones para cada benefício.

---

### **Slide 10: Monitoramento e Backup**

- **Título:** Monitoramento e Backup
- **Conteúdo:**
    - Monitoramento: **Dynatrace** (métricas, logs e alarmes)
    - Backup: Segue o **baseline AKS** no Bradesco.
- **Imagem:** Ícones de monitoramento e proteção de dados.

---

### **Slide 11: Alta Disponibilidade (HA)**

- **Título:** Plano de Alta Disponibilidade
- **Conteúdo:**
    - **Cluster distribuído em múltiplos nós**
    - **Autoescalabilidade** habilitada no AKS
- **Imagem:** Gráfico ilustrando HA no cluster Kubernetes.

---

### **Slide 12: Suporte e Gestão**

- **Título:** Modelo de Suporte
- **Conteúdo:**
    - **Time Interno Bradesco:** Infraestrutura Digital (4250)
        - Contato: **arnaldo.fagnani@bradesco.com.br**
    - **Suporte Humanitec:** Escalonamento direto com fabricante.
- **Imagem:** Ícones de suporte e colaboração.

---

### **Slide 13: Matriz RACI**

- **Título:** Matriz RACI
- **Tabela:**
    
    
    | **Atividade** | **R** | **A** | **C** | **I** |
    | --- | --- | --- | --- | --- |
    | Desenvolvimento de Score | Dev | Arq | Eng | Ops |
    | Provisionamento Terraform | Eng | Ops | Sec | Dev |

---

### **Slide 14: Estimativa de Custos**

- **Título:** Estimativa de Custos
- **Conteúdo:**
    - **Humanitec SaaS:** $XXX
    - **AKS:** $XXX
    - **Dynatrace:** $XXX
- **Imagem:** Gráfico de barras com os custos estimados.

---

### **Slide 15: Encerramento**

- **Título:** Próximos Passos
- **Conteúdo:**
    - Validação da POC com equipes de desenvolvimento.
    - Refinamento do modelo de suporte e automação.
    - Avaliação da escalabilidade para outros projetos.
- **Imagem:** Ícones de progresso ou uma linha do tempo.
