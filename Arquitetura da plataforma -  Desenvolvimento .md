# Desenvolvimento

[Development](https://developer.humanitec.com/training/fundamentals/platform-architecture/development/)

A **fase de Desenvolvimento** corresponde a todos os elementos destacados em **verde** no diagrama. Essa etapa está principalmente relacionada ao:

- **Plano de Controle do Desenvolvedor (Developer Control Plane):**
    - Foco nas ferramentas e processos que os desenvolvedores utilizam no dia a dia para criar e gerenciar código, como IDEs e repositórios de controle de versão.
    - Inclui a interação com o código-fonte das aplicações e da própria plataforma.
- **Plano de Integração e Entrega (Integration and Delivery Plane):**
    - Engloba a automação inicial de processos como a **Integração Contínua (CI)** e o uso de **registros de artefatos** para armazenar componentes gerados durante o desenvolvimento.

Essa fase é essencial para garantir que os desenvolvedores tenham um ambiente integrado e funcional para iniciar e iterar rapidamente no desenvolvimento de aplicações e serviços.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image.png)

### Caminho Dourado: Criando um DNS para o Catálogo de Nancy em Staging

O diagrama apresenta um **"caminho dourado"** para entender como uma tarefa é executada nas diferentes fases de uma IDP, ilustrado pelo caso de Nancy, que precisa de um DNS para sua carga de trabalho chamada **"catalogue"** no ambiente de **staging**.

### **Fase 1: Desenvolvimento (verde)**

1. **Etapa:** Nancy adiciona duas linhas ao arquivo `Score`.
2. **Resultado:** A solicitação é criada e uma **Delta de Deployment** (mudança no estado desejado) é gerada.
3. **Ação:** O pipeline de CI executa o comando `humctl Score`, analisando as alterações e validando o código.

### **Fase 2: Orquestração (azul)**

1. **Etapa:** O Orquestrador da Plataforma cria um **Deployment Set** (conjunto de definições de deployment).
2. **Resultado:** Configurações da carga de trabalho são criadas, incluindo:
    - Definição de `dns-staging`
    - Criação do gráfico de recursos (**Resource Graph**)
    - O gráfico é enviado para o agente responsável.
3. **Etapa:** O Orquestrador gera o **Resource Graph** para a aplicação.
4. **Resultado:** As configurações e dependências são organizadas de forma lógica, prontas para execução.

### **Fase 3: Execução (laranja)**

1. **Etapa:** O Operador executa o gráfico de recursos (**Resource Graph**).
    - Chama o driver correspondente e envia a solicitação para criar o DNS.
    - Atualiza os recursos no Kubernetes e injeta os segredos necessários.
2. **Resultado:** O DNS é criado com sucesso, e a carga de trabalho fica acessível no endereço:
    - **`catalogue-staging.newapp.io`**

Esse processo detalha como as fases da plataforma trabalham em conjunto para entregar rapidamente recursos prontos para produção, com foco em eficiência e automação. Se desejar, posso expandir ou explicar mais alguma etapa!

### Expansão Detalhada das Etapas

Explicação mais detalhada de cada etapa apresentada no "Caminho Dourado" do diagrama:

### **Fase 1: Desenvolvimento (verde)**

**1. Nancy adiciona duas linhas ao arquivo `Score`**

- Essa ação é realizada no contexto do **Plano de Controle do Desenvolvedor (Developer Control Plane)**.
- Nancy adiciona configurações específicas no arquivo `Score` (um arquivo de configuração ou manifesto usado para definir os requisitos de sua aplicação, como DNS, recursos de computação, etc.).
- Essas linhas podem incluir informações como:
    - O nome da aplicação ou serviço.
    - O tipo de recurso necessário (neste caso, DNS).
    - O ambiente em que o recurso será provisionado (staging).

**2. CI executa `humctl Score`**

- Após Nancy realizar as alterações no código, o pipeline de **Integração Contínua (CI)** é acionado automaticamente.
- O comando `humctl Score` é executado:
    - Ele valida as alterações feitas no arquivo `Score`.
    - Gera uma "Delta de Deployment", que é um conjunto de mudanças necessárias para atender às novas configurações.
    - Esse processo garante que a alteração de Nancy seja válida e esteja pronta para avançar para a fase de Orquestração.

### **Fase 2: Orquestração (azul)**

- *3. O Orquestrador cria o **Deployment Set***
- O **Orquestrador da Plataforma** recebe as configurações validadas do CI.
- Ele cria um **Deployment Set**, que é um conjunto de definições necessárias para executar as mudanças.
    - Nesse caso, inclui a criação do DNS para o ambiente de staging.
    - Configurações adicionais, como políticas de segurança e dependências, também são adicionadas.

**4. O Orquestrador gera o Resource Graph**

- Após a criação do Deployment Set, o Orquestrador organiza essas informações em um **Resource Graph**.
    - O **Resource Graph** é um gráfico que representa todos os recursos necessários para atender à solicitação de Nancy.
    - Ele detalha as dependências entre os recursos e a ordem em que devem ser criados.
    - Por exemplo:
        - Primeiro, cria-se o DNS.
        - Depois, conecta-se o DNS à carga de trabalho no Kubernetes.

**5. Resource Graph é enviado ao agente**

- O gráfico gerado é enviado para um **agente** ou operador responsável por executar as mudanças no ambiente.
- Este agente geralmente atua dentro do cluster Kubernetes, garantindo que os recursos sejam criados conforme as especificações.

### **Fase 3: Execução (laranja)**

**6. O Operador executa o Resource Graph**

- O **Operador** é um componente que processa o gráfico de recursos e executa cada etapa de forma sequencial e lógica.
- Neste caso, ele:
    - Chama o driver responsável por gerenciar o DNS.
    - Faz uma solicitação PUT para criar o DNS, fornecendo:
        - O nome do DNS (por exemplo, `catalogue-staging`).
        - O tipo de ambiente (staging).
    - Atualiza os recursos do Kubernetes para que o DNS esteja vinculado à carga de trabalho.
    - Injeta segredos e configurações de segurança necessárias para a operação.

**7. O DNS é criado e funcional**

- Após o Operador concluir a execução, o DNS é criado.
- A carga de trabalho de Nancy fica disponível no ambiente de staging, com o seguinte endereço:
    - **`catalogue-staging.newapp.io`**
- O serviço está pronto para ser usado, permitindo que Nancy ou outros membros da equipe testem e validem a funcionalidade.

### Conexão Entre as Fases

Este fluxo detalhado mostra como as três fases da IDP — Desenvolvimento, Orquestração e Execução — trabalham juntas para transformar uma pequena mudança de configuração em um recurso operacional:

1. **Desenvolvimento:** Nancy define o que precisa (DNS).
2. **Orquestração:** A plataforma organiza os recursos necessários para atender à solicitação.
3. **Execução:** A solicitação é implementada, e o recurso está pronto para uso.

### Benefícios do Processo

- **Automação:** Todas as etapas são automatizadas, reduzindo o trabalho manual e minimizando erros.
- **Eficiência:** A integração entre as fases garante que as alterações fluam rapidamente, desde a ideia inicial até a execução.
- **Escalabilidade:** A estrutura modular permite adicionar novas funcionalidades (como outros serviços ou ambientes) de forma simples e eficiente.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%201.png)

### Visão Geral da Fase de Desenvolvimento

O diagrama destaca o fluxo da **fase de Desenvolvimento**, ilustrando o ciclo interno (**inner loop**) e a interação entre os componentes principais. Aqui está uma explicação detalhada:

### **1. Desenvolvimento Local**

- O processo começa com o desenvolvimento local, onde os desenvolvedores criam e ajustam o código em suas máquinas.
- Este é o ponto inicial para implementar e testar novas funcionalidades.
- Ferramentas comuns nesta etapa incluem:
    - IDEs (como VS Code, IntelliJ, etc.)
    - Simuladores ou ambientes locais de teste (como Docker Compose).

### **2. Composição no Score**

- Após ajustes no código local, o próximo passo é definir a composição do aplicativo ou serviço no **Score**.
- O **Score Compose** é utilizado para configurar como os componentes devem se comportar e interagir.
    - Exemplos de configurações incluem:
        - Recursos necessários (CPU, memória, DNS, etc.).
        - Dependências e serviços conectados.

### **3. Ciclo Interno (Inner Loop)**

- Durante o ciclo interno, os desenvolvedores realizam iterações rápidas entre o desenvolvimento local e o **Score Compose**:
    - Ajustam o código e as configurações com base em feedback.
    - Executam testes repetidos para garantir que tudo está funcionando corretamente antes de avançar para a próxima etapa.

### **4. Pipeline CI/CD**

- Após as validações locais, o código e as configurações são enviados para o **Pipeline CI/CD**.
- O pipeline executa várias etapas automatizadas, como:
    - Validação e integração contínua.
    - Construção de imagens de contêiner.
    - Testes automatizados.

### **5. Registro (Registry)**

- As imagens de contêiner e os artefatos gerados pelo pipeline CI/CD são armazenados em um **Registry**.
- Este registro é essencial para:
    - Gerenciar versões de aplicativos.
    - Fornecer artefatos prontos para implantação nos ambientes seguintes.

### **6. Orquestrador da Plataforma**

- O registro conecta-se ao **Orquestrador da Plataforma**, que organiza e gerencia as implantações nos ambientes apropriados.
- Embora o Orquestrador seja um componente da fase de Orquestração, ele interage com os resultados desta fase para garantir a continuidade do fluxo.

### Benefícios do Processo

- **Iteração Rápida:** O ciclo interno permite que os desenvolvedores experimentem e ajustem rapidamente antes de enviar alterações para o pipeline CI/CD.
- **Automação:** O pipeline CI/CD reduz o trabalho manual e garante consistência nos artefatos entregues.
- **Escalabilidade:** O registro e o Orquestrador tornam o processo escalável para múltiplos ambientes e equipes.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%202.png)

### Opções de Interface para Nancy

O diagrama apresenta as diferentes opções de interface que a usuária Nancy pode utilizar para interagir com a plataforma e implantar seus serviços. Essas opções oferecem flexibilidade dependendo do nível de automação, integração e facilidade de uso desejados.

### **Opções Disponíveis**

### **1. Portal**

- Nancy pode usar um portal como ponto de entrada para configurar e gerenciar sua aplicação.
- A partir do portal, ela pode:
    - **Criar ou estender configurações**: Gerar automaticamente um arquivo **Score** básico (scaffold) ou estender uma configuração existente.
    - Submeter o Score para o pipeline, utilizando uma interface gráfica amigável.

### **2. Arquivo Score**

- Após gerar o arquivo Score por meio do portal ou manualmente, Nancy pode definir configurações específicas, como:
    - Recursos necessários (CPU, memória, armazenamento).
    - Dependências da aplicação.
    - Configurações de ambiente, como variáveis e portas.
- A submissão do Score pode ocorrer de duas formas:
    - **Via Git-push**: Nancy realiza um push das alterações para um repositório Git, o que aciona o pipeline CI/CD automaticamente.
    - **Via `humctl`**: Usando o comando `humctl score deploy`, que valida e envia diretamente o arquivo para o **Orquestrador da Plataforma**.

### **3. CLI (humctl)**

- Nancy também pode interagir com a plataforma via linha de comando, utilizando o utilitário **humctl**.
- Funções principais:
    - Submeter o arquivo Score diretamente.
    - Validar configurações antes do envio.
    - Converter o arquivo Score para JSON, quando necessário, para integração direta com APIs.
- Essa abordagem é ideal para desenvolvedores que preferem automação e maior controle sobre as configurações.

### **4. Chamadas de API Diretas**

- Outra opção é Nancy realizar chamadas de API diretamente para o **Orquestrador da Plataforma**.
- Essa abordagem permite:
    - Integrações personalizadas com outras ferramentas ou pipelines.
    - O envio de configurações em formato JSON, convertidas a partir do Score ou criadas manualmente.

### **Fluxo Final**

Independentemente da interface escolhida, todas as interações culminam no **Orquestrador da Plataforma**, que gerencia a execução das configurações e o provisionamento dos recursos necessários.

### **Benefícios das Diferentes Opções**

- **Portal**: Simplicidade e acessibilidade para usuários menos técnicos.
- **Score**: Flexibilidade para definir configurações detalhadas e padronizadas.
- **CLI (humctl)**: Automação e controle avançado para desenvolvedores experientes.
- **API Direta**: Integração customizada e maior controle em casos específicos.

Essa flexibilidade permite que Nancy e sua equipe escolham a abordagem que melhor se adapta às suas necessidades, sem comprometer a eficiência do fluxo de trabalho. Se desejar, posso detalhar algum ponto ou interface em particular!

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%203.png)

### Exemplo Prático: "Eu preciso de um DNS"

Este exemplo ilustra como Nancy pode solicitar um **DNS** para sua aplicação utilizando o portal e o arquivo **Score**.

### **1. Uso do Portal**

- Nancy utiliza a interface gráfica do portal para adicionar uma dependência de recurso DNS.
- **Processo:**
    - No portal, ela acessa a seção de dependências de recursos e seleciona **DNS**.
    - Define os atributos necessários, como:
        - **ID**: Nome ou identificador do DNS.
        - **Class**: A classe ou tipo de DNS (por exemplo, `default` ou personalizado).
    - Após preencher os campos, ela clica em **Done** para concluir a configuração.
- **Vantagens do Portal:**
    - Interface amigável e intuitiva, ideal para usuários que preferem interações gráficas.
    - Reduz a necessidade de escrita manual de configurações no arquivo Score.

### **2. Configuração no Arquivo Score**

- Após a interação no portal, as configurações do DNS são automaticamente refletidas no arquivo **Score** ou podem ser ajustadas manualmente.
- **Exemplo de Configuração no Score:**

```yaml
app:
  name: my-service
  type: web

resources:
  dns:
    name: my-service-dns
    environment: staging
    class: default
```

- **Explicação do YAML:**
    - `app`:
        - Define o nome do serviço (`my-service`) e o tipo (`web`).
    - `resources`:
        - Inclui a definição do recurso DNS:
            - Nome do DNS: `my-service-dns`.
            - Ambiente: `staging`.
            - Classe do DNS: `default`.

### **Fluxo Completo**

1. **Solicitação no Portal:**
    - Nancy configura o DNS pelo portal com uma interface gráfica amigável.
2. **Atualização do Score:**
    - As configurações realizadas no portal são traduzidas para o arquivo **Score**.
    - Alternativamente, Nancy pode editar manualmente o arquivo Score para ajustes adicionais.
3. **Envio para o Pipeline:**
    - Após configurar o Score, Nancy pode:
        - Fazer um **push** para o repositório Git, acionando o pipeline CI/CD.
        - Usar a CLI (`humctl score deploy`) para enviar as configurações diretamente ao Orquestrador.
4. **Execução pelo Orquestrador:**
    - O Orquestrador processa as configurações e provisiona o DNS conforme solicitado.

### **Benefícios do Processo**

- **Flexibilidade:**
    - Nancy pode optar por usar o portal ou configurar diretamente o arquivo Score.
- **Padronização:**
    - O arquivo Score garante que as configurações sejam consistentes entre diferentes serviços e equipes.
- **Automação:**
    - A integração com o pipeline CI/CD e o Orquestrador permite que o DNS seja provisionado automaticamente.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%204.png)

### Como o Humctl e a Extensão VSC Podem Ajudar Nancy

O **humctl** e a extensão para Visual Studio Code (VSC) são ferramentas essenciais para manter Nancy produtiva durante o desenvolvimento local, dentro do ciclo interno (**inner loop**). Ambas oferecem integração com a plataforma e automatizam ações, tornando os fluxos de trabalho mais rápidos e eficientes.

### **Funcionalidades e Benefícios**

### **1. Humctl**

O **humctl** é uma interface de linha de comando que permite interagir diretamente com a API do Orquestrador da Plataforma. Ele oferece suporte para executar as ações mais comuns relacionadas à implantação e gerenciamento de recursos.

- **Principais Funcionalidades:**
    - Chamar a API do Orquestrador para gerenciar recursos.
    - Visualizar logs de implantação e erros diretamente no terminal.
    - Simular o impacto de ações (como adicionar um DNS).
    - Obter o gráfico de dependências para visualizar como os recursos estão interconectados.
    - Verificar os tipos de recursos disponíveis na plataforma (por exemplo, confirmar se a plataforma pode atender a solicitação de DNS).
- **Exemplo de Comandos com Humctl:**
    
    ```bash
    humctl score deploy --file my-service.score.yaml
    humctl resources list
    humctl logs deployment my-service
    ```
    

### **2. Extensão VSC**

A extensão VSC é integrada diretamente ao Visual Studio Code, um dos IDEs mais usados. Ela oferece uma interface gráfica dentro do IDE, permitindo que Nancy interaja com a plataforma sem sair do ambiente de desenvolvimento.

- **Principais Funcionalidades:**
    - **Visualização do Gráfico de Dependências:**
        - Permite que Nancy entenda as conexões entre os recursos e serviços configurados.
    - **Simulação de Impacto:**
        - Mostra como alterações no Score ou em configurações de recursos impactarão a aplicação.
    - **Logs e Erros:**
        - Exibe logs de implantações e mensagens de erro diretamente no IDE.
    - **Ações Self-Service:**
        - Permite que Nancy acione ações, como implantações, criação de ambientes ou adição de recursos, diretamente da interface gráfica.
- **Vantagens da Integração com IDEs:**
    - Aumenta a produtividade, eliminando a necessidade de alternar entre ferramentas.
    - Torna o processo mais visual e acessível, especialmente para desenvolvedores que preferem uma interface gráfica.

### **Cenários Práticos**

1. **Visualização de Gráficos de Dependência:**
    - Nancy pode usar o **humctl** ou a extensão VSC para gerar e visualizar o gráfico de dependências do serviço. Isso ajuda a entender quais recursos (como DNS, banco de dados) estão conectados e como eles afetam o funcionamento da aplicação.
2. **Simulação do Impacto de Adicionar um DNS:**
    - Antes de enviar o arquivo Score para o pipeline, Nancy pode simular o impacto da adição de um DNS e verificar se a plataforma suporta o recurso solicitado.
3. **Resolução de Erros:**
    - Usando o **humctl** ou a extensão, Nancy pode acessar logs detalhados de implantações para identificar e corrigir erros sem precisar acessar o console do orquestrador.
4. **Ações Self-Service:**
    - Implantar imagens, criar novos ambientes, ou adicionar recursos (como volumes de armazenamento) diretamente do terminal ou IDE, sem depender de outras ferramentas ou equipes.

### **Vantagens Gerais**

- **Manutenção no Inner Loop:**
    - Nancy permanece no ciclo interno de desenvolvimento, com ciclos rápidos de teste e feedback.
- **Flexibilidade:**
    - Pode optar por usar a linha de comando (humctl) ou uma interface gráfica (extensão VSC), dependendo da sua preferência.
- **Produtividade:**
    - Acesso direto às ferramentas e informações necessárias reduz o tempo gasto com tarefas repetitivas ou interrupções.
- **Automação:**
    - Simplifica e acelera ações que, de outra forma, exigiriam várias etapas manuais.

### **Conclusão**

O **humctl** e a extensão para VSC são ferramentas complementares que ajudam Nancy a interagir de forma eficiente com a plataforma, fornecendo visibilidade, controle e automação durante o desenvolvimento local.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%205.png)

### **Score Compose: Implantação Local Facilitada**

O **Score Compose** é uma ferramenta que converte arquivos de configuração **Score** em configurações de **Docker Compose**, permitindo que você inicie cargas de trabalho e suas dependências localmente de maneira rápida e eficiente.

### **Principais Funcionalidades do Score Compose**

1. **Conversão Automática:**
    - Converte um arquivo **Score** (YAML) em um arquivo **docker-compose.yaml**.
    - Isso permite que a aplicação e suas dependências sejam executadas localmente em contêineres Docker.
2. **Execução Local:**
    - Ideal para testes e desenvolvimento local, permitindo que os desenvolvedores validem suas configurações antes de enviá-las para o pipeline CI/CD.
3. **Integração com CI/CD:**
    - O Score Compose pode ser integrado ao pipeline CI/CD para executar testes de integração rapidamente, usando ambientes auto-contidos.

### **Opções de Uso**

### **Opção 1: Implantação Local**

- Execute o comando `score deploy` para iniciar a aplicação e suas dependências em sua máquina local.
- **Benefícios:**
    - Permite que Nancy teste localmente sua aplicação e verifique como ela interage com suas dependências (como banco de dados, DNS, etc.).
    - Facilita o processo de desenvolvimento iterativo no **ciclo interno (inner loop)**.

### **Opção 2: Integração com o CI/CD**

- O pipeline CI/CD pode usar o **Score Compose** para criar ambientes auto-contidos para testes de integração.
- **Benefícios:**
    - Acelera o tempo de execução dos testes de integração.
    - Garante que os ambientes de teste sejam consistentes, replicando a configuração definida no Score.

### **Exemplo Prático**

No exemplo de Nancy, ela deseja testar localmente o DNS para sua aplicação. Para isso, ela simplesmente executa o comando:

```bash
score-compose generate score.yaml
```

Esse comando:

1. Gera um arquivo `docker-compose.yaml` com base nas configurações do `score.yaml`.
2. Inicia a aplicação localmente, juntamente com todos os recursos necessários (DNS, banco de dados, etc.).

### **Vantagens do Score Compose**

1. **Facilidade de Uso:**
    - Reduz a complexidade para configurar ambientes de desenvolvimento local.
    - Automatiza a criação de dependências usando Docker.
2. **Consistência:**
    - Garante que as configurações locais e do pipeline CI/CD sejam as mesmas, evitando erros de configuração em produção.
3. **Produtividade:**
    - Nancy pode testar rapidamente mudanças locais sem a necessidade de acessar ambientes remotos ou configurar recursos manualmente.
4. **Integração Simplificada:**
    - Funciona perfeitamente com o Docker e pipelines CI/CD.

### **Quando Usar o Score Compose?**

- **Durante o Desenvolvimento:**
    - Para validar mudanças localmente no código e nas configurações.
    - Para garantir que as dependências da aplicação (como DNS e banco de dados) estejam funcionando corretamente.
- **No CI/CD:**
    - Para criar ambientes de teste auto-contidos e acelerar testes de integração.

### **Resumo**

O **Score Compose** é uma ferramenta poderosa para desenvolvedores que desejam simplificar o desenvolvimento local e integrar fluxos de trabalho automatizados com CI/CD. Ele oferece flexibilidade, consistência e produtividade, tornando-o indispensável em plataformas modernas de desenvolvimento.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%206.png)

### Pipeline CI/CD: Construção, Testes e Implantação com Score Deploy

O pipeline CI/CD demonstrado no diagrama executa uma sequência de etapas para construir, testar e implantar workloads (cargas de trabalho) em diferentes ambientes: desenvolvimento, staging e produção. Este fluxo automatiza desde o momento em que o desenvolvedor realiza um **git push** até a implantação final pela API do Orquestrador da Plataforma.

### **Etapas do Pipeline**

### **1. Git Push**

- O fluxo começa quando o desenvolvedor faz um **push** de alterações para o repositório Git.
- Esse evento aciona o pipeline CI/CD.

### **2. Construção da Workload**

- O pipeline executa o processo de **build**, que pode incluir:
    - Construção de imagens de contêiner.
    - Compilação de código.
    - Geração de artefatos.

### **3. Testes Automatizados**

- Após a construção, o pipeline executa:
    - **Unit Tests:** Testes unitários para garantir que cada componente funcione isoladamente.
    - **Smoke Tests:** Testes rápidos para verificar se o sistema como um todo está funcionando corretamente.

### **4. Push da Imagem para o Registry**

- A imagem construída é enviada para um **registro de imagens** (como Docker Hub, Amazon ECR ou Google Container Registry).
- Isso garante que a imagem esteja disponível para o ambiente de destino.

### **5. Instalação do Humctl CLI**

- O pipeline instala o CLI **humctl** para interagir diretamente com o Orquestrador da Plataforma.

### **6. Score Deploy**

- O comando **humctl score deploy** é executado, utilizando as configurações definidas no arquivo **Score**.
    - Esse comando envia as configurações para o Orquestrador da Plataforma.

### **7. Orquestrador da Plataforma**

- O **Orquestrador da Plataforma** recebe o payload contendo:
    - **Deployment**: Informações da workload a ser implantada.
    - **Image Registry Path**: O caminho para a imagem no registro.
    - **Metadata/Tag**: Informações de versão ou tags da workload.
- O Orquestrador processa as configurações e gerencia a implantação nos recursos subjacentes.

### **8. Implantação na Infraestrutura**

- O Orquestrador provisiona os **recursos necessários** e implanta a workload no ambiente especificado.
- Os recursos podem incluir:
    - DNS
    - Armazenamento
    - Redes
    - Computação (como contêineres Kubernetes).

### **Benefícios do Pipeline**

1. **Automação Completa**
    - Reduz o trabalho manual, desde a construção até a implantação.
    - Garante que os recursos sejam provisionados de maneira consistente e eficiente.
2. **Testes de Qualidade**
    - Os testes automatizados no pipeline ajudam a identificar e corrigir problemas antes da implantação.
3. **Flexibilidade**
    - Pode ser configurado para diferentes ambientes (desenvolvimento, staging, produção) com base nos requisitos.
4. **Visibilidade**
    - Logs de cada etapa do pipeline fornecem rastreabilidade e permitem monitoramento em tempo real.

### **Resumo do Payload Enviado ao Orquestrador**

O payload que o Orquestrador recebe contém três componentes principais:

1. **Deployment**
    - Contém informações específicas sobre a workload (configurações e recursos necessários).
2. **Image Registry Path**
    - O caminho da imagem no registro, garantindo que a workload utilize a versão correta do contêiner.
3. **Metadata/Tag**
    - Dados adicionais para identificar a versão ou tag da workload, ajudando a rastrear alterações.

### **Fluxo Final**

1. O desenvolvedor faz alterações no código e realiza um **git push**.
2. O pipeline CI/CD executa todas as etapas automaticamente.
3. O **humctl score deploy** envia as configurações para o Orquestrador, que provisiona os recursos e implanta a workload.

Essa abordagem garante automação, consistência e rapidez na entrega de workloads para qualquer ambiente de destino.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%207.png)

### Fase de Orquestração: O Papel do Orquestrador Após o CI

Após o pipeline de CI atingir o endpoint de implantação, o **Orquestrador da Plataforma** assume o controle do fluxo. Ele é responsável por processar os dados enviados pelo pipeline e gerenciar a geração, provisionamento e integração dos recursos necessários para a aplicação.

### **Função do Orquestrador da Humanitec**

1. **Pipelines da Humanitec**:
    - O **Orquestrador da Humanitec**, através dos **Humanitec Pipelines**, gera um **Resource Graph** (gráfico de recursos).
    - Esse gráfico organiza os recursos necessários para atender à solicitação de implantação.
2. **Objetos Criados no Resource Graph**:
    - **Workloads**:
        - Representam as cargas de trabalho que precisam ser implantadas.
    - **Resources**:
        - Incluem os recursos físicos ou virtuais, como DNS, banco de dados, armazenamento e CPU.
    - **SecretMappings**:
        - Gerenciam segredos e configurações sensíveis (como credenciais ou chaves de API) de forma segura.

### **Comportamento Padrão dos Pipelines Humanitec**

O comportamento padrão do Orquestrador pode ser utilizado para várias operações, como:

- **Promoção entre Ambientes**:
    - Automatiza a promoção de workloads de um ambiente para outro (por exemplo, de staging para produção).
- **Operações Avançadas via API**:
    - Permite acionar operações complexas por meio de chamadas de API personalizadas.

### **Componentes do Fluxo de Orquestração**

1. **Plano de Controle do Desenvolvedor (Developer Control Plane)**:
    - Nancy e sua equipe interagem com ferramentas como:
        - Controle de versão.
        - Código-fonte da aplicação e da plataforma.
        - Arquivos **Score** e módulos Terraform (**TF Modules**) para configurar a infraestrutura.
2. **Plano de Integração e Entrega (Integration and Delivery Plane)**:
    - Após o pipeline CI/CD executar as etapas de build e testes, ele envia as informações para o Orquestrador.
3. **Plano de Recursos (Resource Plane)**:
    - O Orquestrador utiliza **Humanitec CRs** (Custom Resources) para gerenciar os seguintes elementos:
        - **Operator**:
            - Gerencia os recursos Kubernetes e integra-os ao fluxo.
        - **Agent**:
            - Executa as operações diretamente nos recursos provisionados.
        - **TF Runner**:
            - Executa módulos Terraform para gerenciar recursos de infraestrutura externa, como serviços em nuvem.
4. **Plano de Monitoramento e Segurança (Monitoring and Logging Plane)**:
    - Ferramentas de observabilidade são utilizadas para monitorar os recursos e workloads.
    - Segredos e identidades são gerenciados de forma centralizada para garantir a segurança.

### **Resumo do Fluxo**

1. O pipeline CI/CD conclui as etapas de build e testes.
2. O **humctl score deploy** é chamado para enviar as configurações do arquivo Score ao Orquestrador.
3. O Orquestrador:
    - Gera o Resource Graph, organizando workloads, recursos e segredos.
    - Provisiona os recursos e implanta a aplicação nos ambientes necessários.
4. A infraestrutura provisionada é gerenciada por operadores, agentes e módulos Terraform, conforme necessário.

### **Benefícios da Orquestração pela Humanitec**

1. **Automação Total**:
    - Reduz o esforço manual ao gerenciar workloads e recursos.
2. **Padronização**:
    - Garante consistência em todos os ambientes, desde desenvolvimento até produção.
3. **Flexibilidade**:
    - Suporta integração com diferentes tipos de infraestrutura (Kubernetes, Terraform, etc.).
4. **Segurança**:
    - Gerencia segredos de forma segura e evita exposição de dados sensíveis.

### Detalhes sobre Como a Humanitec Organiza Recursos e Promove Workloads entre Ambientes

A **Humanitec** oferece um orquestrador avançado que organiza recursos e facilita a promoção de workloads entre ambientes (como desenvolvimento, staging e produção). Aqui estão os detalhes sobre o funcionamento interno do sistema e como ele gerencia os processos:

### **Organização de Recursos na Humanitec**

1. **Resource Graph (Gráfico de Recursos):**
    - O **Resource Graph** é um mapa gerado automaticamente pelo Orquestrador da Humanitec, que organiza e conecta os recursos necessários para uma workload.
    - Ele contém três componentes principais:
        - **Workloads:** Representam as aplicações ou serviços que serão implantados.
        - **Resources:** Incluem recursos como DNS, banco de dados, armazenamento, CPU e memória.
        - **SecretMappings:** Gerenciam segredos e variáveis sensíveis, como chaves de API ou credenciais.
2. **Custom Resources (CRs):**
    - A Humanitec utiliza **Custom Resources (CRs)** no Kubernetes para descrever e gerenciar os recursos.
    - Exemplos de CRs:
        - **DNSResource:** Configura e gerencia entradas DNS para a aplicação.
        - **DatabaseResource:** Cria e conecta bancos de dados.
        - **VolumeResource:** Gerencia volumes de armazenamento.
3. **Provisionamento Dinâmico de Recursos:**
    - Baseado nas configurações enviadas (como o arquivo Score), o Orquestrador determina automaticamente quais recursos precisam ser provisionados.
    - Ele faz isso interagindo com:
        - Operadores Kubernetes para gerenciar recursos no cluster.
        - Runners Terraform para criar recursos fora do Kubernetes (como VMs, buckets de armazenamento ou load balancers em nuvens públicas).
4. **Gestão de Segredos:**
    - Os **SecretMappings** garantem que segredos e dados sensíveis sejam armazenados com segurança.
    - Eles são injetados nos serviços ou recursos somente quando necessário, evitando exposição desnecessária.

### **Promoção de Workloads entre Ambientes**

A **promoção de workloads entre ambientes** é o processo de mover uma aplicação ou serviço de um ambiente (como desenvolvimento) para outro (como staging ou produção). A Humanitec automatiza e padroniza este processo, garantindo consistência e segurança.

### **Etapas do Processo de Promoção**

1. **Definição de Ambientes:**
    - Ambientes (como `dev`, `staging` e `prod`) são configurados na Humanitec com seus próprios conjuntos de recursos e configurações.
    - Cada ambiente pode ter diferentes especificações, como:
        - Maior capacidade de CPU/memória em produção.
        - Menores permissões em desenvolvimento para evitar impactos acidentais.
2. **Configuração do Pipeline:**
    - Um pipeline de promoção é configurado para mover workloads entre os ambientes.
    - Baseia-se nas definições do arquivo Score e no Resource Graph.
3. **Ajuste de Configurações no Ambiente de Destino:**
    - O Orquestrador ajusta dinamicamente os recursos para o ambiente de destino.
    - Por exemplo:
        - No ambiente de produção, ele pode alterar o banco de dados para uma instância de maior desempenho.
        - Configurações de segurança, como chaves e certificados, são injetadas de forma específica para cada ambiente.
4. **Validação Automática:**
    - Antes de promover uma workload, o pipeline executa validações, como:
        - Verificar se todos os recursos do ambiente de destino estão disponíveis.
        - Executar testes básicos (como smoke tests) para garantir que a aplicação funcione corretamente.
5. **Promoção:**
    - Após as validações, o Orquestrador move a workload para o ambiente de destino.
    - Isso inclui:
        - Provisionamento de novos recursos (se necessário).
        - Atualização de recursos existentes.
        - Desativação de recursos que não são mais necessários.

### **Exemplo Prático de Promoção**

Nancy quer promover sua aplicação de **staging** para **produção**. Aqui está como a Humanitec organiza o processo:

1. **Configurações no Staging:**
    - Nancy testa sua aplicação no ambiente **staging**, onde o banco de dados é configurado com menor capacidade e logs são coletados para validação.
2. **Definição do Score:**
    - O arquivo `score.yaml` contém configurações gerais que são adaptadas automaticamente para cada ambiente pelo Orquestrador.
    
    ```yaml
    app:
      name: my-service
      type: web
    
    resources:
      dns:
        environment: production
        name: my-service-prod
    
      database:
        type: managed
        size: large
    ```
    
3. **Promoção pelo Orquestrador:**
    - O Orquestrador verifica o Resource Graph para determinar quais mudanças são necessárias no ambiente de produção:
        - Substitui o banco de dados `small` usado no staging por um banco de dados `large`.
        - Atualiza a entrada DNS para apontar para `my-service-prod`.
4. **Validação e Implantação:**
    - O Orquestrador valida o ambiente de destino, garantindo que todos os recursos estão disponíveis e configurados corretamente.
    - A aplicação é implantada e os logs são disponibilizados para revisão.

### **Benefícios do Sistema de Promoção**

1. **Consistência entre Ambientes:**
    - As mesmas definições no arquivo Score são adaptadas automaticamente para cada ambiente.
    - Isso reduz o risco de discrepâncias ou erros.
2. **Automação Completa:**
    - Nancy não precisa gerenciar manualmente os recursos em cada ambiente; o Orquestrador cuida de tudo.
3. **Flexibilidade:**
    - Diferentes ambientes podem ter configurações personalizadas, como maior capacidade em produção.
4. **Rastreamento e Logs:**
    - Todas as promoções são registradas, permitindo rastreamento e auditoria detalhada.

### **Resumo**

A **Humanitec** organiza recursos de forma dinâmica, segura e eficiente, utilizando o **Resource Graph** para mapear todas as dependências e requisitos de workloads. Além disso, sua abordagem automatizada para promoção de workloads entre ambientes garante consistência e reduz o esforço manual, permitindo que Nancy e sua equipe se concentrem no desenvolvimento de aplicações, em vez de gerenciar infraestrutura.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%208.png)

### Fase de Execução: Papel do Agent e Operator na Execução Final

Na fase de execução, o **Agent** e o **Operator** desempenham um papel crucial para transformar as definições do Orquestrador em recursos operacionais na infraestrutura. Esses componentes garantem que as workloads sejam implantadas e configuradas corretamente no ambiente final.

### **Como Funciona a Execução Final**

### **1. Monitoramento pelo Humanitec Operator**

- O **Humanitec Operator** é responsável por monitorar os objetos personalizados (**Htc objects**) no cluster Kubernetes.
- Ele detecta atualizações ou mudanças e resolve as dependências associadas, acionando os **Drivers** adequados.

### **2. Chamadas aos Drivers da Humanitec**

- O Operator invoca os diferentes **Drivers** para gerenciar recursos específicos:
    - **Driver Echo:** Usado para operações simples ou mensagens de teste.
    - **Driver Template:** Permite transformar configurações baseadas em templates.
    - **Driver Terrafor:** Aciona módulos Terraform para gerenciar recursos fora do Kubernetes (por exemplo, VMs ou storage na nuvem).

### **3. Gerenciamento de Segredos (SecretMappings)**

- Os **SecretMappings** leem referências de segredos externos definidos no **SecretStore**.
- O Operator se comunica com o **Secret Manager** para:
    - Obter credenciais seguras, chaves de API, ou outros dados sensíveis.
    - Injetar esses segredos nas workloads e recursos Kubernetes.

### **4. Implantação dos Recursos Kubernetes**

- O Operator traduz as definições do Resource Graph em recursos Kubernetes, como:
    - **Deployments:** Configuração de workloads baseadas em contêiner.
    - **Services:** Exposição de aplicações para comunicação interna ou externa.
    - **Ingress:** Regras para gerenciar tráfego HTTP e HTTPS para os serviços.
    - **ConfigMaps e Secrets:** Armazenamento de variáveis de configuração e segredos.

### **Componentes Envolvidos na Execução**

1. **Humanitec Operator:**
    - Resolve objetos Htc e gerencia a comunicação com outros drivers e serviços.
2. **Agent:**
    - Executa comandos diretamente na infraestrutura provisionada.
    - Trabalha em conjunto com o Operator para provisionar e atualizar recursos.
3. **Terraform Runner:**
    - Gerencia módulos Terraform para criar ou ajustar recursos fora do Kubernetes.
    - Exemplos:
        - Criar buckets de armazenamento em uma nuvem pública.
        - Configurar firewalls ou balanceadores de carga externos.
4. **Kubernetes Resources:**
    - Os recursos finais implantados no cluster Kubernetes, conforme as definições enviadas pelo Orquestrador.

### **Exemplo de Execução Prática**

Nancy está implantando sua aplicação, que precisa de um banco de dados, DNS e uma entrada Ingress para tráfego HTTP. Aqui está como os componentes trabalham juntos:

1. **Orquestrador da Plataforma:**
    - Envia o Resource Graph para o Humanitec Operator, definindo:
        - Workload da aplicação.
        - Dependências (banco de dados, DNS, etc.).
        - Mapeamentos de segredos.
2. **Humanitec Operator:**
    - Detecta as definições e invoca:
        - **Driver Terrafor**: Cria o banco de dados em uma nuvem pública.
        - **Driver Template**: Configura o DNS com base nos templates fornecidos.
        - **SecretMappings**: Obtém credenciais seguras do banco de dados.
3. **Recursos Kubernetes:**
    - O Operator provisiona:
        - Um **Deployment** para executar a aplicação.
        - Um **Service** para expor a aplicação dentro do cluster.
        - Um **Ingress** para gerenciar o tráfego externo.
4. **Agent e TF Runner:**
    - Trabalham em conjunto para aplicar os recursos fora do Kubernetes (DNS, banco de dados).

### **Benefícios do Processo de Execução**

1. **Automação Completa:**
    - Todo o processo é automatizado, desde a detecção de mudanças até o provisionamento dos recursos.
2. **Flexibilidade:**
    - Suporte a recursos Kubernetes e infraestrutura externa (por meio do Terraform Runner).
3. **Segurança:**
    - Segredos são gerenciados centralmente, reduzindo o risco de exposição.
4. **Eficiência:**
    - Reduz o esforço manual e acelera a entrega de workloads.

## O que sao objetos HTC?

Os **objetos Htc (Humanitec Custom Objects)** são **Custom Resource Definitions (CRDs)** definidos pela **Humanitec** no Kubernetes. Esses objetos são usados para descrever e gerenciar as workloads e recursos no ambiente Kubernetes, conforme configurados no Resource Graph gerado pelo **Orquestrador da Humanitec**.

Em essência, os objetos Htc são representações personalizadas de recursos e configurações que o **Humanitec Operator** monitora e gerencia no cluster Kubernetes.

### **Funções dos Objetos Htc**

1. **Definição de Workloads:**
    - Os objetos Htc descrevem as aplicações (workloads) que precisam ser implantadas, incluindo seus requisitos e dependências.
    - Eles contêm informações como:
        - Imagem do contêiner.
        - Recursos solicitados (CPU, memória, armazenamento).
        - Configurações de rede (DNS, ingressos).
2. **Gerenciamento de Recursos:**
    - Incluem especificações para criar ou modificar recursos necessários para a aplicação, como:
        - Bancos de dados.
        - ConfigMaps e Secrets.
        - Volumes persistentes.
3. **Mapeamento de Segredos:**
    - Integram as configurações de segurança com o Secret Manager para garantir que segredos (como credenciais ou chaves de API) sejam injetados nas workloads de forma segura.
4. **Base para Drivers e Terraform Runner:**
    - Os objetos Htc acionam os **Drivers Humanitec** (como o Terrafor ou Template Driver) para criar e configurar recursos fora do Kubernetes, como:
        - DNS.
        - Balanceadores de carga.
        - Bancos de dados gerenciados.

### **Como Funcionam os Objetos Htc no Kubernetes?**

1. **Definição no Resource Graph:**
    - O Orquestrador da Humanitec gera os objetos Htc como parte do **Resource Graph**, descrevendo tudo o que a aplicação precisa para funcionar.
2. **Monitoramento pelo Humanitec Operator:**
    - O **Humanitec Operator** monitora continuamente os objetos Htc no cluster Kubernetes.
    - Ele detecta mudanças ou atualizações nos objetos e aciona os drivers necessários para resolver as dependências.
3. **Tradução para Recursos Kubernetes:**
    - Com base nas definições dos objetos Htc, o Operator cria os recursos Kubernetes finais, como:
        - **Deployments** (para implantar contêineres).
        - **Services** (para expor aplicações).
        - **Ingresses** (para gerenciar tráfego externo).
        - **ConfigMaps e Secrets** (para configurações e segredos).
4. **Comunicação com Terraform Runner:**
    - Caso os objetos Htc incluam recursos fora do Kubernetes, o Operator delega essas tarefas ao **Terraform Runner**, que aplica as definições usando módulos Terraform.

### **Estrutura de um Objeto Htc**

A estrutura de um objeto Htc pode variar dependendo do recurso ou workload descrito. Um exemplo básico para descrever uma aplicação pode incluir:

```yaml
apiVersion: humanitec.io/v1
kind: HtcWorkload
metadata:
  name: my-application
  namespace: default
spec:
  image: my-docker-registry/my-application:latest
  resources:
    cpu: "500m"
    memory: "256Mi"
  dependencies:
    - name: database
      type: Postgres
      params:
        size: "small"
        version: "13"
  secrets:
    - name: DATABASE_PASSWORD
      valueFrom:
        secretManager: true
```

### **Explicação do YAML:**

- **apiVersion e kind:**
    - Especificam que este é um recurso personalizado (CRD) da Humanitec.
- **metadata:**
    - Nome e namespace do recurso.
- **spec:**
    - **image:** Especifica a imagem do contêiner.
    - **resources:** Define os recursos solicitados (CPU e memória).
    - **dependencies:** Descreve as dependências externas, como um banco de dados PostgreSQL.
    - **secrets:** Configura o mapeamento de segredos, integrando-se ao Secret Manager.

### **Benefícios dos Objetos Htc**

1. **Centralização:**
    - Permitem descrever workloads e recursos de forma padronizada em um único objeto.
2. **Automação:**
    - Acionam automaticamente os drivers e runners necessários para provisionar recursos.
3. **Flexibilidade:**
    - Suportam tanto recursos Kubernetes quanto recursos fora do cluster (via Terraform).
4. **Segurança:**
    - Integram segredos de forma segura e transparente para as workloads.

### **Resumo**

Os **objetos Htc** são o mecanismo da Humanitec para descrever workloads e recursos no Kubernetes. Eles atuam como o ponto de integração entre o Resource Graph gerado pelo Orquestrador e a execução final pelo Humanitec Operator. Com esses objetos, a Humanitec garante que tudo, desde infraestrutura até segredos, seja provisionado de forma consistente e segura.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%209.png)

### O Papel do Orquestrador: Implantação e Promoção de Workloads e Ambientes

O **Orquestrador da Humanitec** permite realizar tanto implantações contínuas quanto a promoção de workloads e ambientes inteiros. Abaixo, detalhamos os três principais padrões operacionais ilustrados no diagrama.

### **1. Implantação Contínua de uma Workload em um Ambiente**

Este padrão é utilizado para realizar implantações contínuas (Continuous Deployment) de uma workload em um ambiente específico.

- **Fluxo:**
    1. O desenvolvedor realiza um **git push** para o repositório de código.
    2. O **pipeline CI/CD** é acionado e executa as etapas de build e testes.
    3. O pipeline invoca o **Orquestrador**, enviando as configurações da workload.
    4. O Orquestrador processa o Resource Graph e provisiona os recursos necessários no ambiente de desenvolvimento.
    5. A workload é implantada no ambiente de destino.
- **Casos de Uso:**
    - Ideal para desenvolvimento e testes rápidos.
    - Permite que as alterações sejam testadas continuamente no ambiente de desenvolvimento.

### **2. Promoção de uma Única Workload para o Próximo Ambiente**

Este padrão promove uma workload específica de um ambiente (por exemplo, **desenvolvimento**) para o próximo ambiente (como **staging**).

- **Fluxo:**
    1. Um gatilho de promoção é acionado no **pipeline CI/CD** (manualmente ou automaticamente).
    2. O pipeline notifica o **Orquestrador**, que inicia o processo de promoção.
    3. O Orquestrador ajusta as configurações da workload para o novo ambiente:
        - Altera os recursos (por exemplo, maior capacidade de CPU/memória em staging).
        - Atualiza segredos ou credenciais para o novo ambiente.
    4. A workload é implantada no próximo ambiente, mantendo consistência e rastreamento.
- **Casos de Uso:**
    - Mover uma aplicação entre ambientes em fases de validação (como staging).
    - Garantir que cada ambiente esteja corretamente configurado antes de chegar à produção.

### **3. Promoção de Todo o Ambiente**

Este padrão promove não apenas uma workload específica, mas todo o ambiente para o próximo estágio (por exemplo, mover **staging** para **produção**).

- **Fluxo:**
    1. Um gatilho de promoção é acionado no **pipeline CI/CD**.
    2. O pipeline instrui o **Orquestrador** a promover o ambiente completo.
    3. O Orquestrador verifica todas as workloads, recursos e segredos do ambiente atual (como staging).
    4. Todos os recursos são replicados ou ajustados para o ambiente de destino (como produção):
        - Ajuste de limites de recursos.
        - Atualização de configurações específicas para produção.
    5. O novo ambiente é configurado e as workloads são implantadas.
- **Casos de Uso:**
    - Promoção de um sistema inteiro (multi-serviços) para produção.
    - Criação de novos ambientes baseados em configurações existentes.

### **Padrões Avançados com Humanitec Pipelines**

Os **Humanitec Pipelines** suportam padrões operacionais mais avançados, como:

1. **Promoção Condicional:**
    - Baseada em critérios específicos, como resultados de testes, métricas de desempenho ou aprovações manuais.
2. **Rollbacks Automatizados:**
    - Se uma promoção falhar, o Orquestrador pode reverter automaticamente para o estado anterior.
3. **Integração com Observabilidade:**
    - Antes de promover workloads ou ambientes, o Orquestrador pode verificar métricas de desempenho e logs para garantir a estabilidade.
4. **Promoção Paralela:**
    - Permite a promoção simultânea de várias workloads ou ambientes para acelerar o fluxo.

### **Resumo dos Benefícios**

- **Automação Completa:**
    - Garante consistência e rastreabilidade em implantações e promoções.
- **Flexibilidade:**
    - Suporta tanto promoções individuais quanto de ambientes inteiros.
- **Segurança e Controle:**
    - Integra-se com sistemas de gerenciamento de segredos e observabilidade para operações seguras e confiáveis.
- **Padronização:**
    - Todas as implantações e promoções seguem padrões definidos, reduzindo erros manuais.

![image.png](Desenvolvimento%201737abd671b4809b838afeb5b9d584ef/image%2010.png)

### Criando Novos Ambientes: Ambientes Efêmeros e Persistentes

A Humanitec permite a criação automatizada de **ambientes efêmeros** e **persistentes**, atendendo a diferentes cenários de desenvolvimento e qualidade. Abaixo, explico os fluxos descritos no diagrama.

### **1. Uso de Ambientes Efêmeros para Pull Requests (PRs)**

Os ambientes efêmeros são criados dinamicamente para validar alterações em Pull Requests, sendo automaticamente descartados após o merge. Este padrão é ideal para times que precisam isolar e testar mudanças de forma eficiente.

### **1.1 Criação Automática do Ambiente Efêmero**

- **Fluxo:**
    1. O desenvolvedor cria uma nova branch e abre um PR (Pull Request).
    2. O pipeline PR é acionado e notifica o **Orquestrador da Plataforma** para criar um ambiente efêmero.
    3. O Orquestrador clona o ambiente de desenvolvimento e configura o ambiente efêmero.
- **Resultado:**
    - Um ambiente temporário é criado, replicando as configurações e recursos necessários para o PR.

### **1.2 Testes no Ambiente Efêmero**

- **Fluxo:**
    1. O desenvolvedor realiza um push para a branch do PR.
    2. O pipeline CI/CD executa o build e implanta as alterações no ambiente efêmero.
    3. Os testes são realizados no ambiente isolado.
- **Resultado:**
    - As mudanças do PR são validadas sem impactar outros ambientes.

### **1.3 Exclusão Automática do Ambiente Efêmero**

- **Fluxo:**
    1. Quando o PR é aprovado e a branch é mergeada, o pipeline notifica o Orquestrador.
    2. O Orquestrador exclui o ambiente efêmero.
    3. As alterações são implantadas no ambiente de desenvolvimento ou destino.
- **Resultado:**
    - O ambiente efêmero é descartado automaticamente, liberando recursos.

### **2. Criação de Ambientes Persistentes**

Além de ambientes efêmeros, a Humanitec permite criar ambientes persistentes para fases como QA (Quality Assurance) ou Staging.

### **Fluxo:**

1. O desenvolvedor solicita a criação de um novo ambiente ou a clonagem de um existente.
2. O **Orquestrador da Plataforma** processa a solicitação:
    - Clona as configurações e recursos do ambiente de origem.
    - Configura os recursos adicionais necessários para o novo ambiente.
3. O novo ambiente é disponibilizado (por exemplo, um ambiente QA).
- **Resultado:**
    - Um ambiente persistente é criado e permanece disponível para testes ou outras finalidades.

### **Comparação: Ambientes Efêmeros vs Persistentes**

| **Aspecto** | **Ambientes Efêmeros** | **Ambientes Persistentes** |
| --- | --- | --- |
| **Objetivo** | Testar PRs isoladamente | Suporte a QA, Staging ou Produção |
| **Duração** | Temporário (descartado após merge) | Longo prazo |
| **Automação** | Criados e excluídos automaticamente | Requer solicitação manual ou automatizada |
| **Uso de Recursos** | Otimizado (liberado após uso) | Requer gerenciamento contínuo |

### **Benefícios dos Ambientes Automatizados**

1. **Isolamento Completo:**
    - Alterações de PR são validadas sem impactar os ambientes principais.
2. **Eficiência de Recursos:**
    - Ambientes efêmeros são automaticamente criados e descartados, evitando desperdício de recursos.
3. **Consistência:**
    - Ambientes efêmeros e persistentes são baseados no mesmo conjunto de configurações, garantindo consistência entre eles.
4. **Automação Total:**
    - Reduz o esforço manual necessário para gerenciar ambientes, permitindo maior foco em desenvolvimento.

### **Casos de Uso**

1. **Ambientes Efêmeros:**
    - Validação rápida de Pull Requests.
    - Testes isolados para evitar conflitos com outros ambientes.
2. **Ambientes Persistentes:**
    - Ambientes QA para validação manual.
    - Ambientes Staging para testes finais antes de produção.

### **Resumo**

A Humanitec facilita tanto a criação de **ambientes efêmeros** quanto a de **persistentes**, automatizando grande parte do processo e otimizando o uso de recursos. Essa abordagem melhora o fluxo de desenvolvimento, garantindo isolamento, consistência e eficiência.

### Detalhes sobre Configuração e Exemplos Específicos de Ambientes na Humanitec

Abaixo está um guia mais detalhado sobre como configurar e trabalhar com **ambientes efêmeros** e **persistentes** na Humanitec, com exemplos concretos para implementá-los.

### **Configuração de Ambientes na Humanitec**

Os ambientes na Humanitec são definidos e gerenciados pelo **Orquestrador da Plataforma**, que utiliza arquivos de configuração (como o `score.yaml`) para provisionar os recursos necessários. A configuração dos ambientes pode ser feita diretamente via API, CLI (`humctl`), ou integrando com pipelines CI/CD.

### **Elementos Necessários para Configuração**

1. **Definições de Recursos:**
    - O ambiente precisa de definições claras sobre os recursos que ele usará:
        - Workloads (aplicações).
        - Bancos de dados.
        - Segredos (por exemplo, credenciais).
        - Configurações de DNS.
2. **Mapeamento de Segredos:**
    - Segredos sensíveis, como chaves de API, devem ser configurados no **Secret Store** e referenciados nos arquivos de configuração do ambiente.
3. **Ambientes de Origem e Clonagem:**
    - Ambientes podem ser clonados a partir de ambientes existentes (por exemplo, criar um ambiente efêmero a partir de `development`).

### **Configuração para Ambientes Efêmeros**

### **Exemplo 1: Criando um Ambiente Efêmero no Pull Request**

O objetivo é criar um ambiente efêmero automaticamente sempre que um Pull Request (PR) for aberto.

1. **Pipeline CI/CD para PRs:**
    - Configure o pipeline CI/CD para acionar a criação do ambiente:
        
        ```yaml
        steps:
          - name: Create ephemeral environment
            script:
              - humctl env create --name pr-${PR_ID} --clone development
              - humctl deploy --env pr-${PR_ID} --file score.yaml
        ```
        
2. **Clonagem do Ambiente de Desenvolvimento:**
    - O comando `humctl env create` clona o ambiente `development`, replicando as configurações de recursos.
3. **Implantação Automática:**
    - O comando `humctl deploy` implanta as alterações no ambiente efêmero com base no arquivo `score.yaml`.
4. **Exclusão Automática:**
    - Configure o pipeline para excluir o ambiente após o merge:
        
        ```yaml
        steps:
          - name: Delete ephemeral environment
            script:
              - humctl env delete --name pr-${PR_ID}
        ```
        

### **Exemplo do Arquivo `score.yaml`:**

Este arquivo define as configurações do ambiente efêmero:

```yaml
app:
  name: my-service
  type: web

resources:
  database:
    type: postgres
    size: small

  dns:
    type: internal
    hostname: pr-${PR_ID}.my-app.local
```

### **Configuração para Ambientes Persistentes**

### **Exemplo 2: Criando um Ambiente de QA**

1. **Solicitação Manual ou Automática:**
    - Um desenvolvedor pode criar um ambiente persistente chamado `qa`:
        
        ```bash
        humctl env create --name qa --clone development
        ```
        
2. **Ajuste de Configurações:**
    - Após clonar o ambiente, ajuste as configurações específicas para QA:
        - Aumente os recursos disponíveis (CPU, memória).
        - Configure endpoints de API diferentes.
    - Isso pode ser feito no `score.yaml`:
        
        ```yaml
        app:
          name: my-service
          type: web
        
        resources:
          database:
            type: postgres
            size: medium
        
          dns:
            type: public
            hostname: qa.my-app.com
        ```
        
3. **Deploy para QA:**
    - Após criar o ambiente, faça o deploy:
        
        ```bash
        humctl deploy --env qa --file score.yaml
        ```
        
4. **Manutenção Contínua:**
    - Ambientes persistentes, como QA, podem ser atualizados conforme necessário com novas configurações ou alterações de workload:
        
        ```bash
        humctl update --env qa --file updated-score.yaml
        ```
        

### **Configuração de Promoção entre Ambientes**

### **Exemplo 3: Promovendo Workloads**

Para mover workloads entre ambientes (por exemplo, de `qa` para `staging`):

1. **Pipeline de Promoção:**
    - Configure o pipeline CI/CD para acionar a promoção:
        
        ```yaml
        steps:
          - name: Promote workload
            script:
              - humctl promote --source qa --target staging
        ```
        
2. **Verificação de Recursos no Alvo:**
    - O Orquestrador ajustará automaticamente os recursos no ambiente de destino (`staging`):
        - Aumentará a capacidade de banco de dados.
        - Configurará DNS público.
3. **Atualização do Ambiente de Destino:**
    - Após a promoção, o ambiente de destino estará atualizado e pronto para testes.

### **Casos Avançados: Promoções e Testes**

### **Exemplo 4: Promoção Condicional**

- Use condições para permitir promoções somente após passar nos testes:
    
    ```yaml
    steps:
      - name: Run tests
        script:
          - run-tests.sh
      - name: Promote workload (conditional)
        script:
          - humctl promote --source qa --target staging
        when:
          condition: success
    ```
    

### **Exemplo 5: Rollbacks Automatizados**

- Configure rollbacks no caso de falha na implantação em produção:
    
    ```yaml
    steps:
      - name: Deploy to production
        script:
          - humctl deploy --env production --file score.yaml
      - name: Rollback (on failure)
        script:
          - humctl rollback --env production
        when:
          condition: failure
    ```
    

### **Integração com Observabilidade**

1. **Logs e Métricas:**
    - Os ambientes criados ou promovidos podem ser monitorados usando ferramentas de observabilidade integradas, como Prometheus e Grafana.
    - A Humanitec suporta observabilidade por padrão, capturando logs e métricas de cada ambiente.
2. **Alertas em Tempo Real:**
    - Configure alertas para detectar falhas em ambientes efêmeros ou persistentes.

### **Resumo**

A Humanitec oferece ferramentas altamente flexíveis para criar, gerenciar e promover ambientes, sejam eles efêmeros ou persistentes. O uso de comandos CLI como `humctl`, arquivos de configuração (`score.yaml`) e integração com pipelines CI/CD automatiza todo o processo, permitindo que sua equipe se concentre no desenvolvimento e na qualidade, sem perder tempo com configurações manuais de infraestrutura.
