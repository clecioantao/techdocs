Aqui está o texto extraído da imagem:

Humanitec

Objetivo

O conteúdo desse documento tem como objetivo apresentar a solução da Humanitec (Platform Orchestrator) no Banco Bradesco.

Público-alvo

Arquitetos, Engenheiros, Analistas de Sistemas e Desenvolvedores.

Índice
	•	Índice
	•	Contexto Humanitec
	•	1. Visão Geral
	•	2. Componentes Principais
	•	3. Papéis e Responsabilidades
	•	4. Processos Operacionais
	•	5. Benefícios Esperados
	•	6. Documentação e Suporte
	•	Developer Control Plane
	•	Integration e Delivery Plane
	•	Monitoring & Logging Plane

Caso precise de algum ajuste ou o documento em outro formato, como Word ou PDF, é só me avisar!

Aqui está o texto extraído da segunda imagem:

Índice (continuação)
	•	Security Plane
	•	Modelo de Arquitetura
	•	Hardening
	•	Taxonomia
	•	Responsável do Componente
	•	Segurança de Redes
	•	1. Isolamento de Segredos
	•	2. Minimização da Exposição da API do Cluster
	•	3. Abordagem GitOps
	•	4. Múltiplos Repositórios de Segredos
	•	5. Gerenciamento Centralizado de Segredos
	•	6. Integração com Ferramentas de Segurança
	•	Logs
	•	Backup
	•	Plano de HA - Alta Disponibilidade
	•	Monitoramento
	•	Modelo de Suporte
	•	Gestão de Acesso
	•	Matriz RACI
	•	Estimativa de Custos

Contexto Humanitec

O que é a Humanitec?

A Humanitec é uma ferramenta que permite a orquestração unificada de workloads (aplicações) e infraestrutura de maneira totalmente agnóstica no “front-end”. Isso significa que quem especifica os recursos no manifesto pode fazê-lo sem se preocupar com os detalhes da implementação da infraestrutura subjacente.

Para que serve a Humanitec?

A Humanitec oferece uma camada de abstração para as equipes de desenvolvimento, facilitando a gestão da infraestrutura por meio da utilização de uma ferramenta CNCF chamada ‘score’.

O principal benefício proporcionado pela Humanitec é a melhoria da experiência do desenvolvedor, além de otimizar o trabalho das equipes de engenharia de cloud responsáveis pela manutenção da plataforma e dos códigos de Infraestrutura como Código (IaC).

Para mais detalhes sobre a ferramenta, consulte a documentação do desenvolvedor da Humanitec: Humanitec | Developer Docs.

Caso deseje seguir com as próximas imagens, basta Aqui está o texto extraído da terceira imagem:

Principais motivos para utilizar a Humanitec
	1.	Orquestração Unificada:
	•	A Humanitec permite a orquestração de workloads (aplicações) e infraestrutura de forma unificada, simplificando a gestão e operação de ambientes complexos.
	2.	Abstração da Infraestrutura:
	•	Oferece uma camada de abstração que permite aos desenvolvedores especificar os recursos no manifesto sem se preocupar com os detalhes da infraestrutura subjacente, utilizando a ferramenta CNCF ‘score’.
	3.	Experiência do Desenvolvedor:
	•	Melhora significativamente a experiência do desenvolvedor ao simplificar o processo de implantação e gestão de aplicações, permitindo que se concentrem mais no desenvolvimento de funcionalidades.
	•	No contexto Bradesco, a construção do score acontecerá de forma automatizada, trazendo a infraestrutura desejada, bem como os values (Argo) referentes ao POD.
	4.	Eficiência para Equipes de Engenharia de Cloud:
	•	Facilita o trabalho das equipes de engenharia de cloud, que são responsáveis pela sustentação da plataforma e dos códigos de Infraestrutura como Código (IaC), proporcionando uma gestão mais eficiente e menos propensa a erros.
	•	No contexto Bradesco, os engenheiros de plataforma constroem os módulos Terraform, criam o “mapa” ou “resource definitions” do score, variáveis e chamam a maneira correta.
	5.	Agnosticismo de Plataforma:
	•	A Humanitec é agnóstica em relação à plataforma, o que significa que pode ser integrada com diversas tecnologias e provedores de infraestrutura, oferecendo flexibilidade e adaptabilidade às necessidades específicas da organização.
	6.	Automação e Consistência:
	•	Promove a automação de processos e garante consistência na implantação e gestão de recursos, reduzindo a possibilidade de erros manuais e aumentando a confiabilidade das operações.
	7.	Manutenção Centralizada e Escalabilidade:
	•	O grande ganho de valor esperado é a manutenção centralizada, facilitada e a escala de implementações de recursos IaC em forma de stack, sem onerar um time para sustentar e evoluir a infraestrutura totalmente separado da aplicação.
	8.	Documentação e Suporte:
	•	A Humanitec oferece uma documentação abrangente e suporte robusto, facilitando a adoção e o uso contínuo da ferramenta pelas equipes de desenvolvimento e operações.

Como ilustrado, a Humanitec cria uma abstração sobre como, onde e o que será provisionado através dos recursos definidos.

Envie mais imagens caso deseje Aqui está o texto extraído da terceira imagem:

Principais motivos para utilizar a Humanitec
	1.	Orquestração Unificada:
	•	A Humanitec permite a orquestração de workloads (aplicações) e infraestrutura de forma unificada, simplificando a gestão e operação de ambientes complexos.
	2.	Abstração da Infraestrutura:
	•	Oferece uma camada de abstração que permite aos desenvolvedores especificar os recursos no manifesto sem se preocupar com os detalhes da infraestrutura subjacente, utilizando a ferramenta CNCF ‘score’.
	3.	Experiência do Desenvolvedor:
	•	Melhora significativamente a experiência do desenvolvedor ao simplificar o processo de implantação e gestão de aplicações, permitindo que se concentrem mais no desenvolvimento de funcionalidades.
	•	No contexto Bradesco, a construção do score acontecerá de forma automatizada, trazendo a infraestrutura desejada, bem como os values (Argo) referentes ao POD.
	4.	Eficiência para Equipes de Engenharia de Cloud:
	•	Facilita o trabalho das equipes de engenharia de cloud, que são responsáveis pela sustentação da plataforma e dos códigos de Infraestrutura como Código (IaC), proporcionando uma gestão mais eficiente e menos propensa a erros.
	•	No contexto Bradesco, os engenheiros de plataforma constroem os módulos Terraform, criam o “mapa” ou “resource definitions” do score, variáveis e chamam a maneira correta.
	5.	Agnosticismo de Plataforma:
	•	A Humanitec é agnóstica em relação à plataforma, o que significa que pode ser integrada com diversas tecnologias e provedores de infraestrutura, oferecendo flexibilidade e adaptabilidade às necessidades específicas da organização.
	6.	Automação e Consistência:
	•	Promove a automação de processos e garante consistência na implantação e gestão de recursos, reduzindo a possibilidade de erros manuais e aumentando a confiabilidade das operações.
	7.	Manutenção Centralizada e Escalabilidade:
	•	O grande ganho de valor esperado é a manutenção centralizada, facilitada e a escala de implementações de recursos IaC em forma de stack, sem onerar um time para sustentar e evoluir a infraestrutura totalmente separado da aplicação.
	8.	Documentação e Suporte:
	•	A Humanitec oferece uma documentação abrangente e suporte robusto, facilitando a adoção e o uso contínuo da ferramenta pelas equipes de desenvolvimento e operações.

Como ilustrado, a Humanitec cria uma abstração sobre como, onde e o que será provisionado através dos recursos Aqui está o texto extraído da quarta imagem:

Estrutura de Trabalho Operacional (Operational Framework)

1. Visão Geral

A Humanitec é uma plataforma que facilita a orquestração de workloads e infraestrutura de forma unificada, proporcionando uma camada de abstração para desenvolvedores e engenheiros de plataforma. Este framework operacional descreve as práticas e processos recomendados para utilizar a Humanitec de maneira eficiente e eficaz.

2. Componentes Principais
	•	Workloads: Aplicações e serviços que são implantados e gerenciados pela Humanitec.
	•	Infraestrutura: Recursos de infraestrutura, como clusters Kubernetes, bancos de dados e serviços de rede, que são provisionados e gerenciados pela Humanitec.
	•	Score: Ferramenta CNCF utilizada para definir a infraestrutura desejada de forma declarativa.
	•	Terraform: Ferramenta de IaC utilizada para provisionar e gerenciar a infraestrutura.
	•	Custom Drivers: Drivers desenvolvidos internamente para interagir com a Ansible Automation Platform e com o Terraform Enterprise no-code provisioning.
	•	Módulos Customizados do Terraform: Módulos customizados utilizados nas definições de recursos (resource definitions) para atender às necessidades específicas da organização.

3. Papéis e Responsabilidades
	•	Desenvolvedores:
	•	Criam e mantêm os manifestos de score para definir a infraestrutura necessária para suas aplicações.
	•	Utilizam a Humanitec para implantar e gerenciar suas aplicações de forma automatizada.
	•	No contexto Bradesco: A construção do score acontece de forma automatizada, trazendo a infraestrutura desejada, bem como os values (Argo) referentes ao POD.
	•	Engenheiros de Plataforma:
(O texto da continuação ainda está incompleto)

Caso tenha mais imagens ou queira finalizar o documento, pode enviar! Estou à disposição.

Envie mais imagens caso deseje Aqui está o texto extraído da quinta imagem:

3. Papéis e Responsabilidades (continuação)
	•	Engenheiros de Plataforma:
	•	Criam os “mapas” ou “resource definitions” que definem como o score irá chamar os recursos e como a Humanitec absorverá essas informações.
	•	Desenvolvem drivers customizados para interagir com a Ansible Automation Platform e com o Terraform Enterprise no-code provisioning.
	•	Utilizam módulos customizados do Terraform nas definições de recursos para atender às necessidades específicas da organização.
	•	No contexto Bradesco: Constroem os módulos Terraform, criam o “mapa” ou “resource definitions” de como o score vai chamar um determinado recurso, e como a Humanitec absorverá essa informação da maneira correta.
	•	Infraestrutura Digital | STDP:
	•	Desenvolvem e mantêm os módulos Terraform para provisionar a infraestrutura.
	•	Sustentam o módulo e o versionam no HCP Terraform para o consumo da Humanitec.

4. Processos Operacionais

Definição de Infraestrutura:
	•	Desenvolvedores criam manifestos de score que definem a infraestrutura necessária para suas aplicações.
	•	Engenheiros de plataforma criam e mantêm módulos Terraform que provisionam a infraestrutura definida nos manifestos de score.
	•	Utilização de drivers customizados para interagir com a Ansible Automation Platform e com o Terraform Enterprise no-code provisioning.

Implantação de Aplicações:
	•	Desenvolvedores utilizam a Humanitec para implantar suas aplicações, que automaticamente provisiona a infraestrutura necessária e aplica os valores de configuração (Argo) aos PODs.

Gestão de Infraestrutura:
	•	Engenheiros de plataforma monitoram e mantêm a infraestrutura provisionada, garantindo que ela esteja em conformidade com as definições de score e os requisitos operacionais.
	•	Utilização de módulos customizados do Terraform nas definições de recursos para atender às necessidades específicas da organização.

5. Benefícios Esperados
	•	Manutenção Centralizada: Facilita a manutenção centralizada da infraestrutura, reduzindo a complexidade e o esforço necessário para gerenciar recursos de IaC.
	•	Escalabilidade: Permite a escala de implementações de recursos IaC em forma de stack, sem onerar um time para sustentar e evoluir a infraestrutura separadamente da aplicação.
	•	Automação e Consistência: Promove a automação de processos e garante consistência na implantação e gestão de recursos, aumentando a confiabilidade das operações.

6. Documentação e Suporte

(Texto ainda incompleto)

Se houver mais imagens, é só enviar! Estou organizando todo o conteúdo para um documento final.

Aqui está o texto extraído da sexta imagem:

	•	A Humanitec oferece uma documentação abrangente e suporte robusto, facilitando a adoção e o uso contínuo da ferramenta pelas equipes de desenvolvimento e operações.
	•	Para mais informações, consulte a documentação do desenvolvedor da Humanitec.

Este framework operacional visa proporcionar uma abordagem estruturada e eficiente para a utilização da Humanitec, maximizando os benefícios para desenvolvedores e engenheiros de plataforma.

Arquitetura de Referência Humanitec

Developer Control Plane

No Developer Control Plane, os desenvolvedores criam aplicações e definem os recursos de infraestrutura necessários (como Redis, bancos de dados, etc.) através do score. Esta camada também inclui a Plataforma de Desenvolvimento Interno (IDP), como o Developer Hub da RedHat, mas no contexto do Bradesco, utilizamos o Backstage.

O Backstage fornece uma interface amigável de self-service para que as equipes de desenvolvimento possam solicitar infraestrutura de forma ágil e automatizada.

Integration e Delivery Plane

Na camada de delivery, temos pipelines de CI que constroem os containers com o código-fonte e os publicam em um registry, que no contexto do Bradesco é o Nexus.

Além disso, a pipeline atualiza a tag da imagem nos arquivos values.yaml do Argo, que utiliza o Helm para fazer o deploy da aplicação no cluster pré-definido no repositório.

A orquestração da infraestrutura também ocorre conforme necessário, como por exemplo, a criação de um novo Redis ou banco de dados através das definições de recursos e módulos Terraform desenvolvidos.

Se houver mais imagens ou algo a ser ajustado, estou pronto para finalizar o Aqui está o texto extraído da sétima imagem:

Monitoring & Logging Plane

Para a observabilidade de toda a solução/aplicação, no contexto do Bradesco, utilizamos o Dynatrace. O Dynatrace é responsável por monitorar métricas, uso e alarmes das plataformas, bem como das aplicações, garantindo uma visão abrangente e detalhada do desempenho e da saúde dos sistemas.

Security Plane

A camada de segurança abrange a autenticação e o controle de acesso baseado em funções (RBAC) das plataformas, bem como auditoria e gerenciamento de permissões por áreas. O foco está em garantir a segurança da plataforma, proporcionando um ambiente seguro e controlado para o desenvolvimento e operação das aplicações.

Este framework operacional visa proporcionar uma abordagem estruturada e eficiente para a utilização da Humanitec, maximizando os benefícios para desenvolvedores e engenheiros de plataforma.

Modelo de Arquitetura

(Imagem de diagrama apresentada aqui)

Hardening

Em desenvolvimento

Taxonomia

(Sem conteúdo adicional disponível nesta seção)

Caso queira revisar ou adicionar algo mais ao documento, basta me informar! Estou finalizando o texto Aqui está o texto extraído da oitava imagem:

Responsável do Componente

Conforme definição acordada entre as partes, ficou estabelecido que o time responsável pela plataforma HCP Terraform será:

Departamento	Equipe Responsável	Contato
4250 - INFRAESTRUTURA DE TI	ARNALDO FAGNANI (4250 - INFRAESTRUTURA DE TI)	arnaldo.fagnani@bradesco.com.br

Qualquer dúvida, necessidade de ajustes, definições, consulta, entrar em contato com a equipe responsável para melhor direcionamento.

Segurança de Redes

Na parte de segurança de redes para o Humanitec, a solução se dá seguindo as melhores práticas do AKS dentro do baseline do Bradesco, o que já garante todos os aspectos de segurança, uma vez que o Operator funciona nativamente dentro de um Kubernetes.

Utilizar o Humanitec Operator (CRD Kubernetes)

O Humanitec Operator oferece vários benefícios em termos de segurança de redes, incluindo:
	•	O armazenamento seguro de segredos,
	•	Minimização da exposição da API do cluster,
	•	Abordagem GitOps para segurança baseada em Git,
	•	Flexibilidade com múltiplos repositórios de segredos,
	•	Gerenciamento centralizado de segredos,
	•	Integração com ferramentas de segurança.

Esses benefícios ajudam a garantir que sua infraestrutura e aplicações sejam gerenciadas de forma segura e eficiente.

Isolamento de Segredos

(O conteúdo desta seção ainda não foi apresentado)

Caso tenha mais imagens ou algo adicional a ser incluído, estarei à Aqui está o texto extraído da nona imagem:

1. Armazenamento Seguro

O Humanitec Operator permite que você armazene segredos em seus próprios repositórios de segredos, como AWS Secrets Manager, Azure Key Vault, Google Cloud Secret Manager e HashiCorp Vault. Isso garante que os segredos sejam armazenados de forma segura e conforme as políticas de segurança da sua organização.

Acesso Controlado

Apenas o Humanitec Operator e os componentes autorizados têm acesso aos segredos, reduzindo o risco de exposição acidental ou maliciosa.

2. Minimização da Exposição da API do Cluster
	•	Proteção da API: Ao utilizar o Humanitec Operator, você pode evitar expor o endpoint do servidor API de seu cluster ao Humanitec Platform Orchestrator. Isso minimiza a superfície de ataque e protege a API do cluster contra acessos não autorizados.
	•	Operação Interna: Em cenários de auto-hospedagem, todas as operações são realizadas dentro da infraestrutura controlada pela sua organização, aumentando a segurança e o controle sobre o ambiente.
	•	Cluster Privado: Seguindo o contexto do baseline do AKS no Bradesco, o cluster será totalmente privado.

3. Abordagem GitOps
	•	Segurança Baseada em Git: No modo GitOps, os manifestos de recursos são armazenados em repositórios Git para gerenciar mudanças de configuração, garantindo que apenas usuários autorizados possam modificar os recursos.
	•	Auditoria e Rastreamento: Todas as mudanças nos manifestos de recursos são versionadas e rastreadas no repositório Git, proporcionando uma trilha de auditoria clara e facilitando a detecção de alterações não autorizadas.

4. Múltiplos Repositórios de Segredos
	•	Flexibilidade e Segurança: A capacidade de configurar múltiplos repositórios de segredos permite que você segmente e isole segredos com base em diferentes critérios, como ambiente (desenvolvimento, teste, produção) ou aplicação. Isso aumenta a segurança ao limitar o escopo de acesso a segredos específicos.

5. Gerenciamento Centralizado de Segredos
	•	Consistência e Conformidade: O Humanitec Operator centraliza o gerenciamento de segredos, garantindo que todos os segredos sejam gerenciados de forma consistente e conforme as políticas de segurança da sua organização.
	•	Automação Segura: A automação do provisionamento de segredos reduz a possibilidade de erros manuais e garante que os segredos sejam provisionados de forma segura e eficiente.

6. Integração com Ferramentas de Segurança

(O conteúdo desta seção ainda não foi apresentado)

Se houver mais imagens ou partes do documento a incluir, estou pronto para consolidar Aqui está o texto extraído da décima imagem:

Compatibilidade com Ferramentas de Segurança

O Humanitec Operator pode ser integrado com ferramentas de segurança existentes, como sistemas de monitoramento e auditoria, para fornecer uma camada adicional de segurança e conformidade.

Modes of Operation: GitOps Mode

(Diagrama apresentado mostrando fluxo de operação GitOps mode)

Referências
	•	Baseline - Azure Kubernetes Service (AKS) - CORPORATIVO | Programa Leap - Confluence (bradesco.com.br)

Boas práticas AKS Contexto Bradesco

Além do uso do cluster privado, seguiremos as melhores práticas citadas no baseline de AKS no que tange rede e serviços gerenciados:
	•	Baseline - Azure Kubernetes Service (AKS) - CORPORATIVO | Programa Leap - Confluence (bradesco.com.br)

Logs

Como o Operador da Humanitec é executado em um cluster AKS, seguiremos as melhores práticas definidas no Baseline do componente, como ilustrado no item 2.1 do baseline:
	•	Baseline - Azure Kubernetes Service (AKS)

Caso queira consolidar o documento completo ou incluir mais imagens, me avise! Estou pronto para finalizar o Aqui está o texto extraído da décima primeira imagem:

A equipe responsável pelo suporte interno é a DITI - AUTOMAÇÃO CLOUD (4250 - INFRAESTRUTURA DE TI).

Para o acionamento de solicitações e dúvidas, deve-se encaminhar um e-mail ou entrar em contato com a equipe de Automação Cloud liderada pelo Arnaldo Fagnani
	•	arnaldo.fagnani@bradesco.com.br

Suporte do produto via Humanitec
Em desenvolvimento

Gestão de Acesso

Em desenvolvimento

Matriz RACI

Em desenvolvimento

Construção da Matriz RACI com as áreas responsáveis pelo componente e interdependências com respectivos papéis.

Recurso	R - Responsável	A - Aprovador	C - Consultado	I - Informado
Solução				
Monitoramento				
Gestão de Acessos				
Redes				

Caso precise consolidar tudo em um documento ou queira ajustes específicos, é só avisar!





