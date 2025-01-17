### **Documentação Técnica: Pavimentação da Plataforma Humanitec**

---

### **Área**

**Intelligence Management - Automation & Orchestration**

**Infraestrutura Digital | STDP**

---

### **Decisão de Arquitetura**

Adoção de um cluster Kubernetes para atender as demandas da plataforma **Humanitec**, garantindo a implementação do modelo de orquestração e infraestrutura como código (IaC).

---

### **Descrição do Problema**

A **Humanitec** é uma plataforma que opera no modelo SaaS (Software as a Service) e funciona como um **Internal Developer Platform (IDP)**, permitindo a abstração de operações e a automação do ciclo de vida de aplicações. No entanto, para o pleno funcionamento, é necessário implementar uma infraestrutura Kubernetes dedicada para executar o **Operator (CRD)** da Humanitec.

Este **Operator** é responsável por:

1. Gerenciar a orquestração de infraestrutura e aplicações de maneira declarativa.
2. Garantir o aprovisionamento dinâmico de recursos na nuvem.
3. Integrar pipelines de CI/CD e realizar deploys consistentes.
4. Fornecer configurações e deploys seguros por meio de namespaces isolados.

Sem essa infraestrutura, a operação da Humanitec pode ser comprometida, uma vez que o Operator é o componente essencial que conecta o portal de orquestração às infraestruturas subjacentes.

---

### **Premissas**

1. O cluster Kubernetes será configurado com um namespace dedicado para o **Operator da Humanitec**, com permissões e restrições alinhadas às melhores práticas de segurança.
2. Configurações de firewall serão liberadas para os seguintes endpoints:
    - `drivers.humanitec.io`
    - `agent-01.humanitec.io`
3. As conexões deverão ser realizadas exclusivamente na porta **TCP/443**, utilizando certificados TLS para comunicação segura.

---

### **Alternativas Consideradas**

### **1. Cluster ARO Existente na Azure**

- Utilização de um cluster **Azure Red Hat OpenShift (ARO)** já existente na infraestrutura corporativa.
- **Vantagens**: Redução de custo inicial e reaproveitamento de recursos.
- **Desvantagens**: Necessidade de compatibilidade com o Operator da Humanitec.

### **2. Cluster ARO On-Premises Integrado com Azure Arc**

- Configuração de um cluster **ARO on-premises**, gerenciado por **Azure Arc**.
- **Vantagens**: Maior controle de dados e integração híbrida.
- **Desvantagens**: Exige configuração inicial complexa e custos adicionais.

### **3. Novo Cluster AKS na Azure**

- Implementação de um cluster Kubernetes no serviço gerenciado **Azure Kubernetes Service (AKS)**.
- **Vantagens**: Escalabilidade nativa, baixo custo operacional e suporte otimizado para o Operator da Humanitec.
- **Desvantagens**: Custo inicial de setup e provisionamento de recursos.

### **4. Cluster AKS Existente na Azure**

- Reaproveitamento de um cluster **AKS** já existente na infraestrutura corporativa.
- **Vantagens**: Menor tempo de implementação e economia de recursos.
- **Desvantagens**: Risco de impacto em workloads existentes.

---

### **Decisão Recomendada**

Com base nas análises realizadas, a recomendação é:

- **Efetuar o deploy em um novo cluster AKS na Azure**.
- **Justificativa**:
    - Suporte nativo para Kubernetes e CRDs.
    - Escalabilidade ajustável de acordo com as demandas do Operator.
    - Infraestrutura gerenciada, reduzindo o overhead de administração.

---

### **Implicações Técnicas**

1. **Segurança**: Necessidade de configuração avançada para proteger namespaces, roles e policies.
2. **Escalabilidade**: A solução deve ser capaz de suportar o crescimento da Humanitec em termos de usuários e requisições.
3. **Custo**: Custos adicionais com criação e gerenciamento de um novo cluster Kubernetes.

---

### **Próximos Passos**

1. Provisionar o cluster Kubernetes no **Azure Kubernetes Service (AKS)**.
2. Configurar o **Operator da Humanitec** com os namespaces e policies exigidos.
3. Liberar as regras de firewall para os endpoints da Humanitec.
4. Realizar testes de integração e validação do ambiente.

---

---

### **Documentação Técnica: Modelo Operacional Humanitec**

---

### **Área**

**Intelligence Management - Automation & Orchestration**

**Engenharia de Plataformas Técnicas e Funcionais (DS)**

---

### **Decisão de Arquitetura**

Definição do modelo operacional do **Humanitec** para integração e funcionamento no ambiente **Bradesco**.

---

### **Descrição do Problema**

A plataforma **Humanitec** opera em dois modos distintos de integração com o ambiente Kubernetes, cada um com seus **trade-offs**:

1. **Direct Mode**:
    - O **Operator da Humanitec** é responsável diretamente por aplicar os manifestos (via `kubectl apply`) no cluster Kubernetes relacionado ao **POD**.
    - **Vantagens**: A simplicidade operacional elimina intermediários no ciclo de deploy.
    - **Desvantagens**: Menor aderência aos padrões internos de pipelines já consolidados.
2. **GitOps Mode**:
    - Neste modelo, o **Operator da Humanitec** realiza um **push** no repositório de configuração de estado (state repository) gerenciado pelo **ArgoCD**.
    - O **ArgoCD**, por sua vez, realiza o deploy no cluster Kubernetes.
    - **Vantagens**: Compatibilidade total com as práticas atuais de **GitOps** e esteiras existentes.
    - **Desvantagens**: Introdução de uma dependência adicional (ArgoCD) no pipeline.

O objetivo é integrar a **Humanitec** de forma otimizada ao ambiente do Bradesco, minimizando retrabalhos em esteiras já existentes de **Infraestrutura como Código (IaC)** e **App Deployment**, garantindo alinhamento aos padrões de governança e segurança internos.

---

### **Premissas**

1. O modelo escolhido deverá respeitar as práticas existentes do Bradesco para deploys baseados em **Helm Templates** e configuração por repositórios Git.
2. O **GitOps Mode** será configurado para realizar **commits** em repositórios já existentes, evitando a criação de novos pipelines de deploy sempre que possível.
3. Será necessário provisionar e configurar um repositório de **state config** dedicado para integração com o **ArgoCD** e o **Humanitec Operator**.

---

### **Alternativas Consideradas**

### **1. GitOps Mode com ArgoCD**

- **Descrição**: Utilizar o modelo GitOps para realizar commit em um repositório de estado, integrando com o **ArgoCD** para gerenciar o deployment dos **PODs**.
- **Vantagens**:
    - Integração com pipelines GitOps existentes.
    - Alta governança e rastreabilidade por meio do repositório Git.
    - Respeita os padrões internos de templates Helm.
- **Desvantagens**:
    - Requer configuração inicial do repositório de state e integração com o ArgoCD.
    - Introduz dependências no pipeline (ArgoCD).

### **2. Direct Mode (Humanitec Direct Deploy)**

- **Descrição**: Configurar a **Humanitec** para realizar o `kubectl apply` diretamente nos clusters Kubernetes, eliminando a necessidade de intermediários.
- **Vantagens**:
    - Simplicidade na configuração e menor overhead inicial.
    - Menor complexidade operacional no início do projeto.
- **Desvantagens**:
    - Reduz a compatibilidade com esteiras baseadas em GitOps e Helm Templates.
    - Pode exigir ajustes manuais para governança e segurança no deploy.

---

### **Decisão Recomendada**

**Adotar o GitOps Mode** como modelo operacional padrão para o ambiente Bradesco.

### **Justificativas**:

1. Alinhamento com as práticas existentes de **GitOps**, permitindo uma integração fluida com o **ArgoCD**.
2. Compatibilidade com pipelines e templates **Helm** já implementados.
3. Maior rastreabilidade e governança devido ao gerenciamento centralizado em repositórios Git.
4. Escalabilidade e padronização de deploys em múltiplos clusters Kubernetes.

---

### **Implicações Técnicas**

1. **Provisionamento de Repositório**: É necessário configurar e validar o repositório de **state config** para integração com o **ArgoCD**.
2. **Segurança**: Garantir permissões adequadas de acesso ao repositório para o **Humanitec Operator**.
3. **Complexidade Inicial**: Configurar o pipeline GitOps pode introduzir complexidade no início, mas a longo prazo reduz retrabalho e facilita a manutenção.

---

### **Sobre a Integração Humanitec**

O **Humanitec** é uma plataforma de **Internal Developer Platform (IDP)** que simplifica e automatiza o gerenciamento de deploys e infraestrutura por meio de práticas modernas como **GitOps** e **infraestrutura declarativa**.

**Principais Benefícios**:

- **Direct Mode**: Ideal para configurações simples e ambientes menores, onde governança avançada não é prioridade.
- **GitOps Mode**: Perfeito para ambientes corporativos com esteiras complexas e requisitos rigorosos de rastreabilidade e governança.

No ambiente do Bradesco, a integração com **ArgoCD** fortalece as práticas de **IaC**, alinhando a **Humanitec** às estratégias corporativas de automação e orquestração.

---

### **Próximos Passos**

1. Configurar o **repositório de state config** para integração GitOps.
2. Validar a integração do **Humanitec Operator** com o **ArgoCD**.
3. Realizar testes de deploy para validar a compatibilidade com os templates Helm.
4. Monitorar e documentar o desempenho da solução para ajustes futuros.

---

---

### **Documentação Técnica: Deployment das Aplicações**

---

### **Área**

**Intelligence Management - Automation & Orchestration**

**Engenharia de Plataformas Técnicas e Funcionais (DS)**

**DevSecOps**

---

### **Decisão de Arquitetura**

Definir como as aplicações serão provisionadas nos clusters corporativos (**ARO**, **ROSA**, **AKS**) integrando as funcionalidades do **Humanitec** ao pipeline de CI/CD existente, sem alterar ou comprometer a arquitetura atual.

---

### **Descrição do Problema**

A organização utiliza uma arquitetura robusta de **CI/CD** baseada em **ArgoCD** para o deployment de microsserviços. Essa solução suporta práticas de **GitOps** que garantem consistência, rastreabilidade e governança dos deployments.

O **Humanitec** propõe agregar valor ao fluxo atual ao orquestrar workloads utilizando uma ferramenta chamada **SCORE**, que introduz uma nova camada de abstração. O **SCORE Manifest** adiciona uma chave de **resources**, permitindo que os desenvolvedores abstraiam a complexidade da infraestrutura.

O desafio reside em integrar o **Humanitec** e o **SCORE** ao modelo atual sem exigir modificações disruptivas na arquitetura existente.

---

### **Premissas**

1. O **Humanitec** deve agregar ao modelo **GitOps** existente baseado em **ArgoCD**, sem comprometer os fluxos atuais.
2. O uso de **SCORE** é permitido, mas deve se integrar aos **values** e ao **Helm Template** atualmente utilizados pela equipe **DevSecOps**.
3. A solução deve respeitar os padrões de governança, segurança e automação já adotados pela organização.

---

### **Alternativas Consideradas**

### **1. Adotar o SCORE como Nova Forma de Orquestração**

- **Descrição**: Aceitar o **SCORE Manifest** como o padrão para orquestração de workloads no **ArgoCD**, substituindo o modelo atual.
- **Vantagens**:
    - Abstração da complexidade para os desenvolvedores.
    - Automação simplificada e foco em recursos.
- **Desvantagens**:
    - Implica em um redesenho completo do pipeline de **CI/CD**.
    - Alta curva de aprendizado para equipes que precisam migrar para o novo modelo.
    - Riscos de interrupção em deploys existentes.

### **2. Integração Customizada do Humanitec no ArgoCD**

- **Descrição**: Desenvolver uma integração personalizada onde o **Humanitec** realize commits no repositório de estado atual do **ArgoCD** sem modificar os templates de deployment existentes.
- **Vantagens**:
    - Preserva o modelo de **CI/CD** existente.
    - Adiciona valor ao pipeline atual sem criar rupturas ou retrabalhos.
    - Integração direta com **Helm** e **GitOps**, alinhando-se aos padrões internos.
- **Desvantagens**:
    - Requer desenvolvimento adicional para criar a integração.
    - Pode demandar monitoramento contínuo para evitar conflitos entre os fluxos existentes e a integração.

---

### **Decisão Recomendada**

**Adotar a integração customizada do Humanitec ao ArgoCD**, permitindo que o **Humanitec** se integre diretamente ao repositório de estado atual do **ArgoCD** sem alterar os templates de deployment.

### **Justificativas**

1. **Preservação do Modelo Atual**: Essa abordagem evita redesenhos disruptivos no pipeline de CI/CD, garantindo estabilidade operacional.
2. **Valor Adicional**: O **Humanitec** adiciona valor ao fluxo existente ao invés de substituí-lo, permitindo abstração de infraestrutura sem comprometer a eficiência.
3. **Governança e Padrões**: A solução mantém a compatibilidade com os padrões atuais de **Helm Templates** e **GitOps**, já validados pelas equipes **DevSecOps**.

---

### **Implicações Técnicas**

1. **Integração com Repositórios de Estado**:
    - Será necessário criar uma lógica que permita ao **Humanitec** realizar commits diretamente no repositório de estado do **ArgoCD**, alinhando-se ao fluxo atual de deploys.
2. **Compatibilidade com SCORE**:
    - A integração deve permitir o uso do **SCORE Manifest**, garantindo que suas chaves de **resources** sejam mapeadas corretamente para os valores esperados nos **Helm Templates**.
3. **Monitoramento e Logs**:
    - Devem ser configurados mecanismos para monitorar conflitos entre a integração do **Humanitec** e os fluxos existentes, garantindo rastreabilidade em tempo real.
4. **Segurança**:
    - Permissões adequadas devem ser configuradas para que o **Humanitec** acesse e modifique o repositório de estado do **ArgoCD** sem introduzir vulnerabilidades.

---

### **Próximos Passos**

1. Desenvolver a integração personalizada entre o **Humanitec** e o **ArgoCD**, com foco em commits nos repositórios de estado.
2. Validar a compatibilidade do **SCORE Manifest** com os **Helm Templates** atuais, garantindo que a abstração seja funcional.
3. Realizar testes de integração e deploy em um ambiente de staging para garantir estabilidade.
4. Documentar o processo e treinar as equipes envolvidas na utilização do novo fluxo.

---

---

### **Documentação Técnica: Caderno de Testes**

---

### **Objetivo**

Validar se a solução proposta atende os problemas atuais, promove melhorias operacionais e apoia o crescimento em escala do banco.

---

### **Engenharia de Plataforma**

### **Área**:

**Aut. Cloud**

### **Pontos de Validação**

1. **Facilitação do Conceito DRY (Don't Repeat Yourself):**
    - A solução promove a reutilização de componentes e padrões para a construção de stacks de infraestrutura?
    - Reduz redundâncias e aumenta a eficiência no uso de recursos?
2. **Redução do Time-to-Market:**
    - A solução acelera a criação e deploy de novos módulos **Terraform**?
    - Oferece automação que agilize o provisionamento?
3. **Simplificação do Gerenciamento de Componentes:**
    - A solução facilita a criação, gerenciamento e modificação de novos componentes e arquétipos?
    - Oferece suporte a alterações parametrizadas ou de fácil integração?
4. **Integração com Github Actions (GitOps IaC):**
    - A solução complementa ou substitui o uso do **Github Actions** nas esteiras de **GitOps IaC**?
    - Oferece vantagens técnicas ou de custo ao substituir?

---

### **DevSecOps**

### **Área**:

**DevSecOps**

### **Pontos de Validação**

1. **Complementação ou Substituição do ArgoCD:**
    - A solução oferece recursos que complementem ou substituam as funcionalidades do **ArgoCD**?
    - Suporta práticas de GitOps sem criar retrabalhos ou complexidades adicionais?
2. **Integração com Esteira CI/CD Atual:**
    - A solução se integra de forma nativa à esteira de **CI/CD** existente?
    - A implementação pode ser realizada sem grandes impactos ou alterações estruturais?

---

### **Desenvolvedor**

### **Área**:

**Desenvolvimento**

### **Pontos de Validação**

1. **Mudanças no Repositório de Código:**
    - Quais são as alterações necessárias no repositório de código para que os desenvolvedores utilizem a solução?
    - A solução impacta a organização ou a estrutura dos repositórios?
2. **Interação com Recursos de Infraestrutura:**
    - A responsabilidade pela inclusão ou alteração de recursos de infraestrutura será necessariamente do desenvolvedor?
    - A solução oferece abstrações que simplifiquem esse processo para o desenvolvedor?

---

### **Próximos Passos**

1. **Definição do Escopo de Testes:**
    - Identificar cenários específicos que cobrem as validações de Engenharia de Plataforma, DevSecOps e Desenvolvedores.
    - Criar uma matriz de requisitos e expectativas para cada ponto de validação.
2. **Execução de Testes:**
    - Simular cenários reais para verificar a integração com as esteiras atuais de CI/CD e IaC.
    - Avaliar o impacto no fluxo de trabalho dos desenvolvedores e a facilidade de uso.
3. **Análise de Resultados:**
    - Documentar os resultados de cada validação, destacando os pontos fortes e os gaps da solução.
4. **Ajustes e Feedback:**
    - Realizar ajustes na solução com base no feedback das equipes de Engenharia de Plataforma, DevSecOps e Desenvolvimento.

---

---

### **Documentação Técnica: Premissas & Requisitos**

---

### **Objetivo**

Definir os problemas que a orquestração deve resolver e esclarecer dúvidas e requisitos para integração de infraestrutura como código (**IaC**) e workflows atuais.

---

### **Problemas que Esperamos Resolver com a Orquestração**

1. **Facilitação na Migração/Integração para IaC**:
    - A orquestração deve auxiliar na transição de infraestrutura manual para modelos declarativos baseados em IaC.
2. **Redução de Alterações Manuais**:
    - Evitar mudanças manuais em variáveis ou componentes do **Internal Developer Platform (IDP)**.
3. **Eliminação de Repetição em Múltiplos Repositórios**:
    - Centralizar e simplificar alterações que afetam vários repositórios.
4. **Gerenciamento de Complexidade com Novos Arquétipos**:
    - Abstrair e automatizar a criação de arquétipos, reduzindo a dependência de configurações manuais.
5. **Redução da Dependência do Time de Plataforma**:
    - Automatizar a integração de novos arquétipos para minimizar dependências em equipes específicas.
6. **Automatização de Repositórios de Template**:
    - Criar scaffolds automaticamente para suportar novos arquétipos no IDP.
7. **Evitar Quebra de Integração com o IDP**:
    - Manter consistência e integração mesmo após mudanças em variáveis ou código.
8. **Eliminação de Acionamento Manual**:
    - Automatizar acionamentos e alterações necessárias para manter a integração entre infraestrutura e aplicação.
9. **Gerenciamento Unificado do Ciclo de Vida**:
    - Implementar uma abordagem única e integrada para o ciclo de vida de aplicações e infraestrutura.
10. **Abstração Multi-Cloud**:
    - Prover suporte para provisionamento em múltiplas clouds (AWS, Azure, GCP) de forma abstraída e uniforme.

---

### **Dúvidas Técnicas (Em Construção)**

1. **Customização do GitOps Mode**:
    - É possível customizar a forma como o **GitOps Mode** funciona, permitindo interação balanceada ou redundante com a infraestrutura?
2. **Infraestrutura Legada**:
    - Como a solução integrará ou coexistirá com a infraestrutura legada?
3. **Fonte Única de Verdade**:
    - Como manter a consistência se provisionarmos pela interface do Backstage e realizarmos edições diretas no **SCORE**?
4. **Compatibilidade do Terraform Driver**:
    - O **Terraform Driver** suporta tanto repositórios Git quanto registros privados no **Terraform Cloud**?
5. **Foco da Orquestração**:
    - A solução será voltada para workloads, infraestrutura, ou ambos?
6. **Integração com Ferramentas Existentes**:
    - A solução se integra totalmente ao ambiente e ferramentas atuais ou será necessário revisar o fluxo de ponta a ponta?
7. **Impacto na Esteira Atual de Infraestrutura e Aplicação**:
    - A esteira atual será descontinuada ou complementada pela nova solução?
8. **Substituição do Helm Values pelo SCORE**:
    - O **SCORE** substituirá o **Helm Values** no **ArgoCD** ou será complementar?

---

### **Notas Adicionais**

1. **Integração com Ferramentas CI/CD Existentes**:
    - A ferramenta deve operar de forma conjunta com as ferramentas de **CI/CD** existentes, como o **GitHub Actions** e o **ArgoCD**.
2. **Compatibilidade com Registry Privado do Terraform Cloud**:
    - A solução deve permitir o uso de drivers do Terraform tanto em repositórios Git quanto em registros privados no **Terraform Cloud**.
3. **Provisionamento via API/CLI**:
    - O provisionamento da infraestrutura deve ser realizado via API ou CLI do **Terraform Cloud**, sem dependência de execução local nos clusters Kubernetes onde os operadores estão instalados.

---

### **Próximos Passos**

1. **Refinamento de Requisitos**:
    - Revisar as premissas e esclarecer as dúvidas levantadas junto às equipes responsáveis pelas ferramentas de orquestração e integração.
2. **Validação Técnica**:
    - Testar a compatibilidade da solução com registros privados no **Terraform Cloud** e verificar as possibilidades de customização no **GitOps Mode**.
3. **Prototipação**:
    - Desenvolver um protótipo que integre o **SCORE**, **Backstage**, **GitHub Actions**, e o **ArgoCD**, avaliando o impacto em esteiras existentes.
4. **Documentação e Treinamento**:
    - Elaborar documentação detalhada e treinar as equipes de desenvolvimento, DevOps e plataforma para adoção da solução.

---

---

### **Documentação Técnica: Requisitos para o MVP**

---

### **Objetivo**

Garantir que os pré-requisitos necessários para iniciar o **MVP (Minimum Viable Product)** estejam atendidos, proporcionando um início rápido e bem-sucedido do piloto. Este checklist foi disponibilizado pela **Humanitec** para apoiar a preparação das diversas áreas envolvidas no projeto.

---

### **Checklist de Requisitos**

### **1. Planejamento Inicial**

1.1 **Realização do MVP Discovery Homework**

- Preencher o PDF denominado **"Homework"** (fornecido pela equipe da Humanitec).
- Enviar o documento preenchido à equipe da **Humanitec**.
- Agendar discussões com a **Humanitec** sobre o MVP e seus requisitos.

1.2 **Revisões Internas de Segurança e Financeiro**

- Discutir o MVP e os requisitos com as equipes de **Segurança** e **Financeiro**.
- Garantir todas as aprovações necessárias dessas áreas e de outros stakeholders relevantes para iniciar o MVP.
- Iniciar qualquer processo de revisão ou aquisição necessário para a implementação em produção.

---

### **2. Formação da Equipe Técnica**

2.1 **Designação da Equipe Técnica para Integração do MVP**

- Definir uma equipe técnica com conhecimento em:
    - **IaC/Terraform**,
    - **Kubernetes**,
    - Infraestrutura em geral.
- Garantir que a equipe esteja pronta para iniciar os trabalhos na data de início do MVP.
- Verificar períodos de indisponibilidade (férias ou feriados) da equipe.
- Garantir a alocação de ao menos **1-2 horas por dia** para trabalhar no MVP.

---

### **3. Identificação e Preparação da Aplicação de Teste**

3.1 **Seleção de uma Aplicação para Teste no MVP**

- Escolher uma aplicação simples ou um **mock** para o piloto.
- Convencer a equipe responsável pela aplicação a dedicar **4 horas** para testar a plataforma Humanitec.
- Ler o documento **"Identifying and onboarding app 1 during MVPs"** antes de abordar a equipe responsável.

3.2 **Coleta de Artefatos de Deploy**

- Reunir os seguintes artefatos para a aplicação de teste:
    - **Helm Charts**,
    - Arquivos **Terraform (TF)**,
    - Pipelines de CI/CD,
    - Diagramas de Rede.
- Identificar a documentação da pipeline DevSecOps utilizada para a aplicação.
- Obter o diagrama de arquitetura da aplicação.
- Identificar o módulo ou módulos Terraform utilizados na infraestrutura da aplicação.
- Coletar o repositório de configuração (config-repo) da aplicação.

---

### **4. Comunicação e Colaboração**

4.1 **Integração com Slack/Teams**

- Determinar se é possível acessar um **Slack/Teams** externo ou convidar membros externos para uma instância existente.
- Configurar um espaço para mensagens e colaboração com a equipe **Humanitec**.

---

### **5. Configuração da Organização Humanitec**

5.1 **Criação da Organização na Humanitec**

- Garantir que a organização **Humanitec Org** esteja criada.
- Se ainda não foi criada, fazê-lo [aqui](https://humanitec.com/).
- Convidar o arquiteto da **Humanitec** para a organização com **acesso de administrador**.
- **Exemplo de nome:** Banco Bradesco (**banco-bradesco**).

5.2 **Configuração de SSO (Single Sign-On)**

- Determinar a necessidade de SSO para autenticação.
    - Caso não seja necessário, utilizar o **GitHub como IDP (Identity Provider)**.
- Reunir os dados necessários e compartilhá-los com o arquiteto da **Humanitec**.

---

### **6. Infraestrutura e Clusters**

6.1 **Criação e Configuração de Contas Cloud**

- Criar contas na nuvem necessárias (sandbox ou produção) para o MVP.

6.2 **Criação dos Clusters Necessários**

- Verificar em qual provedor de nuvem os clusters devem ser criados (e.g., AWS, Azure).
- Criar os clusters necessários.
- Garantir acesso ao cluster para a equipe de **ENG.PLAT**.

6.3 **Configuração de Acessos**

- Configurar acesso para os seguintes endpoints da **Humanitec**:
    - `drivers.humanitec.io`
    - `agent-01.humanitec.io`
- Abrir regras de firewall para as portas:
    - **TCP/80**
    - **TCP/443**
- Confirmar se é necessário abrir outras portas além dessas.

---

### **Próximos Passos**

1. **Revisar e Completar o Checklist:**
    - Garantir que cada item do checklist seja atendido antes da data de início do MVP.
2. **Validação com Stakeholders:**
    - Validar a preparação com as equipes de Segurança, Financeiro e stakeholders técnicos.
3. **Testes de Integração:**
    - Realizar testes iniciais de integração com a aplicação selecionada para o MVP.
4. **Documentação e Monitoramento:**
    - Documentar o progresso e os ajustes realizados durante o MVP.
    - Monitorar a execução para garantir que os objetivos do piloto sejam atingidos.

---

---

### **Documentação Técnica: Integração do Backstage com o Humanitec**

---

### **Objetivo**

Definir o processo de integração do **Backstage** com a plataforma **Humanitec**, possibilitando uma experiência centralizada para desenvolvedores e equipes de plataforma ao gerenciar aplicações, infraestrutura e fluxos de orquestração.

---

### **Visão Geral da Integração**

O **Backstage** é uma plataforma de developer portal que centraliza ferramentas, serviços e documentação para equipes. A integração com a **Humanitec** possibilita:

1. **Gerenciamento de Aplicações**: Sincronizar aplicações gerenciadas pelo **Humanitec** com o catálogo do Backstage.
2. **Provisionamento de Infraestrutura**: Executar workflows do **Humanitec** diretamente no Backstage para provisionamento e gerenciamento de infraestrutura.
3. **Orquestração Declarativa**: Automatizar deploys de infraestrutura e aplicações utilizando drivers e manifests suportados pelo **Humanitec**.
4. **Observabilidade Centralizada**: Monitorar estados de deploy, erros e logs de workflows no Backstage.

---

### **Componentes da Integração**

1. **Backstage Plugins**
    - O plugin **Humanitec** será configurado no **Backstage** para interação com a API da plataforma.
    - O plugin permite:
        - Gerenciamento de drivers e manifests do Humanitec.
        - Exibição de logs e status de deploys.
        - Execução de workflows do Humanitec.
2. **Humanitec API**
    - O **Humanitec API** fornece endpoints REST para integração com o Backstage.
    - Autenticação via **API Keys** ou **OAuth** será configurada para comunicação segura.
3. **Configuração GitOps**
    - Sincronização com repositórios de configuração para atualizar o estado do **GitOps** no Backstage.
4. **Drivers e Manifests**
    - Uso de drivers Terraform e Kubernetes do Humanitec para provisionamento de infraestrutura e deploy de workloads.
    - Os manifests suportam templates personalizáveis para integrar com Helm, ArgoCD e pipelines existentes.

---

### **Pré-Requisitos para Integração**

1. **Backstage**
    - Instância do Backstage operacional na infraestrutura.
    - Catálogo configurado com serviços e aplicações existentes.
2. **Humanitec**
    - Organização criada na plataforma Humanitec.
    - API Key gerada para autenticação.
    - Configuração de drivers necessários (Terraform, Kubernetes, etc.).
3. **Infraestrutura**
    - Clusters Kubernetes configurados e integrados ao Humanitec.
    - Regras de firewall abertas para os endpoints da Humanitec:
        - `drivers.humanitec.io`
        - `agent-01.humanitec.io`
4. **Credenciais e Permissões**
    - Configurar permissões no Backstage para acesso aos recursos do Humanitec.
    - Configurar o Backstage para acesso ao repositório de configuração do GitOps.

---

### **Passos para Integração**

### **1. Configuração do Backstage Plugin**

1. Instalar o plugin oficial do **Humanitec** no Backstage.
    
    ```bash
    npm install @backstage/plugin-humanitec
    
    ```
    
2. Configurar o arquivo `app-config.yaml` do Backstage para adicionar o plugin:
    
    ```yaml
    humanitec:
      baseUrl: https://api.humanitec.io
      apiKey: YOUR_HUMANITEC_API_KEY
    
    ```
    
3. Adicionar o plugin à interface do Backstage:
    
    ```tsx
    import { HumanitecPage } from '@backstage/plugin-humanitec';
    const routes = (
      <Route path="/humanitec" element={<HumanitecPage />} />
    );
    
    ```
    

---

### **2. Configuração da API Humanitec**

1. **Gerar API Key**:
    - Acesse o painel do Humanitec e crie uma API Key com permissões para leitura/escrita.
2. **Drivers Configurados**:
    - Configure os drivers necessários no painel da Humanitec:
        - **Terraform**: Integração com repositórios Git ou registros privados no Terraform Cloud.
        - **Kubernetes**: Drivers configurados para clusters AKS, EKS ou GKE.

---

### **3. Sincronização de Repositórios GitOps**

1. Configurar repositórios de configuração no Backstage:
    - Sincronizar repositórios utilizados pelo **GitOps (ArgoCD)** para gerenciar aplicações e infraestrutura.
2. Configurar integração entre **SCORE** e **Helm Values** no repositório Git:
    - Garantir que as mudanças realizadas pelo Humanitec respeitem os templates e values existentes.

---

### **4. Execução de Workflows no Backstage**

1. Configurar ações personalizadas no Backstage para acionar workflows no Humanitec:
    - **Exemplo**: Deploy de um módulo Terraform ou um workload Kubernetes.
2. Adicionar botões de execução no painel do Backstage para provisionamento e deploy.

---

### **5. Observabilidade e Monitoramento**

1. Configurar o plugin Humanitec para exibir logs e status de workflows:
    - Logs de deploys, erros e recursos provisionados aparecerão diretamente no painel do Backstage.
2. Garantir a integração com ferramentas de observabilidade (e.g., Prometheus, Grafana) para monitorar clusters e workloads.

---

### **Pontos de Validação**

1. **Sincronização com o Catálogo**:
    - Aplicações gerenciadas pelo Humanitec devem aparecer no catálogo do Backstage.
2. **Execução de Workflows**:
    - Garantir que workflows do Humanitec podem ser acionados diretamente do Backstage.
3. **Compatibilidade com GitOps**:
    - As alterações realizadas pelo Humanitec devem respeitar templates Helm e pipelines ArgoCD existentes.
4. **Logs e Monitoramento**:
    - Logs e status de deploys devem ser exibidos em tempo real no Backstage.

---

### **Próximos Passos**

1. **Testes de Integração**:
    - Validar todos os fluxos configurados em ambiente de teste.
2. **Documentação Interna**:
    - Criar guias para desenvolvedores e equipes de plataforma utilizarem a integração.
3. **Treinamento de Equipes**:
    - Realizar workshops para explicar o uso do Backstage com o Humanitec.
4. **Acompanhamento e Ajustes**:
    - Monitorar o uso e realizar ajustes conforme feedback das equipes.
    - 
---
### **Deployment da Infraestrutura**

---

### **Área:**

**Intelligence Management - Automation & Orchestration**

---

### **Decisão de Arquitetura:**

Definir a forma como o **Humanitec** realizará ou acionará o deploy de uma infraestrutura.

---

### **Descrição do Problema:**

O **Terraform Cloud** foi identificado como a ferramenta responsável por executar os comandos `terraform plan` e `terraform apply`, tornando-se a origem principal dos deployments. Para integrar esse fluxo, o **Humanitec** precisa interagir ou disparar o provisionamento remoto no Terraform Cloud.

---

### **Premissas:**

1. **Integração com Terraform Cloud**:
    - O manifesto do driver Terraform no Humanitec deve permitir integração com o Terraform Cloud.
    - A integração pode ser feita por meio de **entrypoints** ou **variáveis** configuradas via runner customizado.
2. **Proximidade Operacional**:
    - Os **runners** (agent pools) do Terraform Cloud e o **Humanitec Operator** **DEVEM** estar no mesmo cluster **AKS** para garantir a comunicação eficiente.

---

### **Alternativas:**

1. **Integração via Entrypoints**:
    - Configurar o **Humanitec** para acionar o Terraform Cloud diretamente no manifesto, utilizando **entrypoints** que executem `terraform login`.
    - Dessa forma, o Terraform Cloud gerencia automaticamente as credenciais e identifica o contexto de deploy remoto.
2. **(Alternativa em Aberto)**:
    - **[Preencher com proposta adicional ou variante de integração]**
3. **(Alternativa em Aberto)**:
    - **[Preencher com proposta adicional ou outra abordagem técnica viável]**

---

### **Decisão:**

A integração entre o **Humanitec** e o **Terraform Cloud** será realizada utilizando **entrypoints** no manifesto do driver Terraform do Humanitec, configurando comandos como `terraform login`. Isso permitirá que o Terraform Cloud gerencie as credenciais automaticamente e execute o provisionamento remoto de forma segura e eficiente.

---

### **Justificativas:**

1. **Simplicidade e Eficiência**:
    - O uso de **entrypoints** permite uma configuração mais direta e padronizada no manifesto do Humanitec.
    - A abordagem reduz a complexidade, uma vez que o **Terraform Cloud** resolve automaticamente as credenciais e gerencia os deployments remotamente.
2. **Flexibilidade na Integração**:
    - A integração via entrypoints oferece flexibilidade para customizar a comunicação com o Terraform Cloud, garantindo compatibilidade com diferentes cenários de infraestrutura.
3. **Manutenção Simplificada**:
    - Evitar a necessidade de criar runners customizados ou pipelines complexos reduz custos de manutenção e pontos de falha na automação.
4. **Conformidade e Escalabilidade**:
    - A obrigatoriedade de manter os runners (Terraform Cloud) e o Humanitec Operator no mesmo cluster AKS assegura latência reduzida e conformidade operacional, além de facilitar a escalabilidade do ambiente.

---

### **Implicações:**

1. **Dependência do Cluster AKS**:
    - Como os runners e o Humanitec Operator devem estar no mesmo cluster AKS, a disponibilidade do cluster se torna crítica para a execução do pipeline.
2. **Configuração Inicial**:
    - Será necessário configurar corretamente os entrypoints no manifesto do Humanitec, além de garantir que as credenciais estejam sincronizadas no Terraform Cloud.
3. **Monitoramento e Logs**:
    - O pipeline exigirá uma camada robusta de monitoramento para rastrear falhas na execução do `terraform plan` e `terraform apply`, bem como a interação entre o Humanitec e o Terraform Cloud.
4. **Limitação em Casos Não Suportados**:
    - Qualquer limitação futura nos entrypoints ou mudanças na API do Terraform Cloud podem impactar o fluxo de trabalho e exigir adaptações.

---

### **Notas Adicionais:**

Com essa abordagem, a organização consegue um modelo simples, escalável e seguro para gerenciar deploys de infraestrutura. A validação do ambiente no cluster AKS será essencial para mitigar os riscos relacionados à dependência operacional. Caso necessário, uma alternativa deverá ser revisada para cenários de falha no cluster AKS ou no Terraform Cloud.

---

A decisão de adotar o Score em uma grande corporação financeira requer uma análise cuidadosa devido às particularidades do setor e às características do produto emergente. Vamos considerar os aspectos principais para ajudar na sua decisão.

1. Benefícios Potenciais do Score
Abstração e Simplificação:

O Score elimina a necessidade de gerenciar manifestos Kubernetes específicos, oferecendo uma interface agnóstica para diferentes ambientes.
Isso reduz a complexidade operacional e facilita a padronização em múltiplos times.
Velocidade de Adaptação:

Como o Score abstrai detalhes de infraestrutura, equipes podem se concentrar no desenvolvimento e no deploy, acelerando o time-to-market.
Agnosticismo de Plataforma:

Permite criar configurações reutilizáveis entre diferentes provedores de cloud (AWS, GCP, Azure) ou ambientes on-premises, alinhando-se a estratégias multicloud e híbridas.
Cultura DevOps:

Promove boas práticas de colaboração entre desenvolvedores e operações, ajudando a implementar uma mentalidade de infraestrutura como código (IaC).
2. Riscos Envolvidos
Produto Emergente:

Risco: O Score ainda está em seus estágios iniciais e pode carecer de suporte robusto, comunidade ativa e compatibilidade com ferramentas de mercado.
Impacto: Se houver bugs críticos ou funcionalidades limitadas, sua organização pode enfrentar gargalos.
Integração com o Ecossistema Atual:

Risco: Ferramentas existentes como Terraform, ArgoCD e Helm são amplamente adotadas e maduras. Substituí-las ou integrá-las ao Score pode ser desafiador.
Impacto: A curva de aprendizado e customizações pode gerar atrasos e custos adicionais.
Conformidade Regulatória:

Risco: O setor financeiro exige conformidade rigorosa (LGPD, GDPR, PCI DSS). Um produto emergente pode não oferecer suporte nativo ou auditabilidade adequada.
Impacto: Violações podem levar a multas e danos à reputação.
Suporte e Comunidade:

Risco: A ausência de uma comunidade madura ou suporte comercial robusto pode deixar sua equipe sem ajuda em momentos críticos.
Impacto: Soluções alternativas ou personalizações podem aumentar os custos.
3. Avaliação do Custo-Benefício
Quando Pode Valer a Pena?
Projetos Isolados ou Experimentais:
Adotar o Score em projetos-piloto ou para aplicações não críticas reduz o impacto de falhas e oferece uma base para avaliação do produto.
Equipes Pequenas com Foco em Velocidade:
Se sua organização prioriza agilidade em detrimento de maturidade no curto prazo, o Score pode simplificar o processo.
Multicloud ou Híbrido:
Se o ambiente já é distribuído entre clouds ou inclui on-premises, o Score pode facilitar a padronização.
Quando Evitar?
Sistemas Críticos:
Para aplicações de missão crítica (como gateways de pagamento ou plataformas de trading), o Score pode introduzir riscos desnecessários.
Grandes Ambientes Consolidados:
Se sua infraestrutura já está bem estabelecida com ferramentas maduras (Terraform, Helm, ArgoCD), a adoção pode ser mais disruptiva do que benéfica.
4. Estratégia para Mitigar Riscos
Começar com um Piloto:

Escolha um projeto pequeno e não crítico para validar o Score em um ambiente controlado.
Documente os aprendizados e ajustes necessários.
Engajamento com a Comunidade/Time do Produto:

Participe de discussões no GitHub ou canais oficiais para garantir que o produto atende às necessidades específicas.
Contribua com feedback e relatórios de bugs.
Plano de Saída:

Desenvolva uma estratégia para reverter para ferramentas tradicionais caso o Score não atenda às expectativas.
Conformidade e Auditoria:

Certifique-se de que o Score pode ser configurado para atender aos requisitos regulatórios e de segurança antes de expandir seu uso.
5. Conclusão
Adotar o Score em uma grande corporação financeira pode ser uma boa aposta para projetos inovadores ou para simplificar ambientes multicloud/híbridos. No entanto, é crucial considerar o impacto potencial nos sistemas críticos e o custo de integração com ferramentas existentes.

A recomendação é:

Comece com um projeto piloto bem delimitado.
Avalie métricas como eficiência operacional, compatibilidade e maturidade do suporte antes de expandir o uso para o ambiente produtivo.

---

### **005 - Integração com o IDP:**

Área
Intelligence Management - Automation & Orchestration
DevEx

Decisão de Arquitetura
Integração com o IDP

Descrição do Problema
Atualmente o IDP atua realizando um commit no repositório IaC e a esteira é responsável por processá-lo.
Precisamos definir a arquitetura como o IDP atuará com a Humanitec ou isso ficará em outra camada.
A Humanitec espera um apply no score e/ou no repo de state target para triggerar o Operator que aplica a infraestrutura. Sendo assim, o IDP precisará interagir diretamente com o score.

Premissas

IDP irá interagir com o score do Humanitec para incluir e/ou remover os recursos de acordo com o front-end.
IDP irá commitar o repositório para a esteira.
O IDP irá consumir um repo de scaffold para fazer o placeholder das informações.
Alternativas

GitHub Actions faz a interação com a Humanitec após o IDP commitar no repositório fazendo o score/manifesto para incluir ou editar recursos.
IDP faz a chamada direta via API na Humanitec para trigger uma inclusão ou alteração de recursos.

Decisão
A decisão tomada foi utilizar a Alternativa 2, onde o IDP fará a chamada direta via API na Humanitec para incluir, alterar ou remover recursos, sem depender de intermediários como o GitHub Actions para processar commits no repositório. Esta abordagem simplifica a arquitetura e elimina possíveis pontos de falha, garantindo maior controle direto sobre a interação com o score.

Justificativas
Simplicidade Operacional: A chamada direta à API da Humanitec elimina a necessidade de configurar e manter esteiras intermediárias, reduzindo a complexidade da solução.
Agilidade: A integração direta com a Humanitec permite que as mudanças sejam aplicadas em tempo real, reduzindo o tempo para propagação de alterações nos recursos.
Menor Dependência: Não depender do GitHub Actions para mediar a interação evita problemas com integrações adicionais e aumenta a autonomia do IDP.
Flexibilidade: A solução oferece maior flexibilidade para personalizar a interação entre o IDP e a Humanitec, sem limitações impostas por ferramentas de terceiros.
Escalabilidade: A abordagem direta via API se adapta melhor a um crescimento futuro, suportando mais usuários e operações simultâneas.

Implicações
Responsabilidade no IDP: O IDP assumirá maior responsabilidade em gerenciar diretamente a interação com a API da Humanitec, exigindo maior robustez no código e monitoramento adequado.
Segurança: Será necessário implementar autenticação segura e permissões adequadas ao usar a API da Humanitec para evitar acessos não autorizados.
Monitoramento e Logs: Deve ser configurado um sistema de monitoramento e logging para rastrear todas as chamadas à API e identificar possíveis falhas ou problemas de desempenho.
Treinamento da Equipe: A equipe de desenvolvimento e operação precisará se familiarizar com a API da Humanitec para lidar com a nova solução e suas particularidades.
Customização do Score: O IDP deverá garantir que as alterações enviadas à API respeitem o formato esperado pelo score da Humanitec, evitando inconsistências ou erros.


