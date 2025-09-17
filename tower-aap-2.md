# 📘 Tutorial & Documentação — Migração Tower → AAP 2.4

## 🎯 Objetivo

Realizar a **migração (lift and shift)** de projetos, job templates, inventários e workflows da instância **Ansible Tower (Next)** para a instância **Ansible Automation Platform 2.4 (AAP – Bradesco)**.

O processo deve ser **automatizado, auditável e versionado**, mantendo os objetos íntegros e minimizando intervenção manual.

---

## 📋 Escopo da Migração

* ✅ **Projetos** (Git ou manual)
* ✅ **Job Templates**
* ✅ **Workflows**
* ✅ **Inventários**
* ✅ **Credenciais** (recriadas no AAP, pois não são exportadas)
* ✅ **Execução automatizada via playbooks**

⚠️ **Não incluso**: RBAC detalhado (usuários, times, permissões) — deve ser validado manualmente conforme políticas do Bradesco.

---

## 📦 Estrutura de Projeto

Organizar todos os arquivos de migração em um repositório Git:

```
migracao-tower-aap/
├── inventories/
│   ├── tower.yml
│   └── aap.yml
├── playbooks/
│   ├── export-tower.yml
│   ├── import-aap.yml
│   ├── sync.yml
│   └── create-credentials.yml
├── requirements.yml
└── README.md
```

---

## 🔧 Dependências

1. **Coleção `awx.awx`**

   ```bash
   ansible-galaxy collection install awx.awx
   ```

   Ou via arquivo:

   ```yaml
   # requirements.yml
   collections:
     - name: awx.awx
       version: 22.7.0
   ```

2. **Acesso API** ao Tower e ao AAP (usuário admin ou com permissões equivalentes).

3. **Execução via Ansible** em máquina de jump ou DevOps runner.

---

## 📋 Etapas da Migração

### 1. Inventário do Tower

Antes de migrar, é essencial levantar tudo o que existe no Tower (Next).

📌 **Comandos úteis com `awx-cli`:**

```bash
# Listar projetos
awx-cli projects list

# Listar Job Templates
awx-cli job_templates list

# Listar credenciais
awx-cli credentials list

# Listar inventários
awx-cli inventories list

# Exportar tudo de uma vez (JSON)
awx-cli export --all > tower_backup.json
```

Esse arquivo `tower_backup.json` deve ser **versionado em Git** e usado como referência de auditoria.

---

### 2. Preparação no AAP 2.4

No AAP 2.4, os jobs rodam em **Execution Environments (EEs)**, containers que substituem os antigos **virtualenvs** do Tower.

📄 `execution-environment.yml`

```yaml
version: 1
dependencies:
  galaxy: requirements.yml
  python: requirements.txt
```

* `requirements.yml` → coleções Ansible necessárias
* `requirements.txt` → dependências Python extras

🔨 **Build e push do EE**

```bash
ansible-builder build -t ee-bradesco:1.0
podman push ee-bradesco:1.0 registry-interno/ee-bradesco:1.0
```

Esse EE deve ser configurado no **Controller do AAP** antes da importação dos job templates.

---

### 3. Configuração de Inventários

#### `inventories/tower.yml`

```yaml
all:
  hosts:
    tower:
      awx_host: "https://tower.next.interno"
      awx_username: "admin"
      awx_password: "SENHA_TOWER"
```

#### `inventories/aap.yml`

```yaml
all:
  hosts:
    aap:
      awx_host: "https://aap.bradesco.interno"
      awx_username: "admin"
      awx_password: "SENHA_AAP"
```

---

### 4. Exportação de Objetos (Tower)

📄 `playbooks/export-tower.yml`

```yaml
- name: Exportar dados do Tower
  hosts: tower
  gather_facts: no
  tasks:
    - name: Exportar todos os objetos
      awx.awx.export:
        all: true
      register: export_data

    - name: Salvar em arquivo JSON
      copy:
        content: "{{ export_data | to_nice_json }}"
        dest: "./tower_export.json"
```

Execução:

```bash
ansible-playbook -i inventories/tower.yml playbooks/export-tower.yml
```

---

### 5. Importação de Objetos (AAP)

📄 `playbooks/import-aap.yml`

```yaml
- name: Importar dados para o AAP
  hosts: aap
  gather_facts: no
  tasks:
    - name: Carregar JSON exportado do Tower
      slurp:
        src: "./tower_export.json"
      register: tower_json

    - name: Importar objetos no AAP
      awx.awx.import:
        data: "{{ tower_json['content'] | b64decode | from_json }}"
```

Execução:

```bash
ansible-playbook -i inventories/aap.yml playbooks/import-aap.yml
```

---

### 6. Execução Encadeada (Export + Import)

📄 `playbooks/sync.yml`

```yaml
- import_playbook: export-tower.yml
- import_playbook: import-aap.yml
```

Execução direta:

```bash
ansible-playbook -i inventories/tower.yml -i inventories/aap.yml playbooks/sync.yml
```

---

### 7. Criação de Credenciais no AAP

Credenciais **não são exportadas** por segurança → precisam ser recriadas.

📄 `playbooks/create-credentials.yml`

```yaml
- name: Criar credenciais no AAP
  hosts: aap
  gather_facts: no
  tasks:
    - name: Criar credencial de Máquina (SSH)
      awx.awx.credential:
        name: "cred-ssh-app"
        description: "Acesso SSH padrão para servidores Linux"
        organization: "Default"
        credential_type: "Machine"
        inputs:
          username: "ansible"
          ssh_key_data: "{{ lookup('file', '~/.ssh/id_rsa') }}"
          become_method: "sudo"

    - name: Criar credencial Git (SCM)
      awx.awx.credential:
        name: "cred-gitlab"
        description: "Token GitLab para sync de projetos"
        organization: "Default"
        credential_type: "Source Control"
        inputs:
          username: "gitlab-user"
          password: "{{ lookup('env', 'GITLAB_TOKEN') }}"

    - name: Criar credencial HashiCorp Vault
      awx.awx.credential:
        name: "cred-hashicorp-vault"
        description: "Integração com HashiCorp Vault"
        organization: "Default"
        credential_type: "Vault"
        inputs:
          url: "https://vault.interno.bradesco"
          token: "{{ lookup('env', 'VAULT_TOKEN') }}"

    - name: Criar credencial AWS
      awx.awx.credential:
        name: "cred-aws"
        description: "Credenciais AWS (IAM User)"
        organization: "Default"
        credential_type: "Amazon Web Services"
        inputs:
          username: "{{ lookup('env', 'AWS_ACCESS_KEY_ID') }}"
          password: "{{ lookup('env', 'AWS_SECRET_ACCESS_KEY') }}"
```

Execução:

```bash
ansible-playbook -i inventories/aap.yml playbooks/create-credentials.yml
```

---

### 8. Testes e Validação

1. Executar **job templates principais** no AAP.
2. Validar saída via **ansible-navigator**.
3. Testar workflows ponta-a-ponta.
4. Conferir vínculos entre:

   * Projetos → Repositórios Git
   * Inventários → Hosts e variáveis
   * Credenciais → Usuários, chaves, tokens
   * Execution Environments → dependências Python/Collections

---

### 9. Checklist Final de Migração

* [ ] Projetos importados e apontando para repositórios corretos
* [ ] Inventários revisados e testados (hosts e variáveis)
* [ ] Job Templates funcionando com credenciais recriadas
* [ ] Workflows testados ponta-a-ponta
* [ ] Execution Environments configurados e publicados
* [ ] Credenciais sensíveis recriadas manualmente ou via playbook
* [ ] Documentação da migração registrada no Git/Notion

---

## 📊 Benchmark & Estratégia (Case Red Hat)

### Timeline e Esforço

* **Janela de migração:** meses, não semanas (Red Hat levou \~7 meses).
* **Execução por ondas:** pilotos, smoke tests, corte gradual.

### Riscos e Mitigações

| Risco                       | Sintoma                | Mitigação                                       |
| --------------------------- | ---------------------- | ----------------------------------------------- |
| Incompatibilidade venv → EE | Job falha pós-migração | Congelar coleções, EEs por domínio, smoke tests |
| Dependências de SO ausentes | Módulos falham         | Declarar em `requirements.txt` + rebuild do EE  |
| Workflows quebram           | IDs mudam              | Reimportar/relinkar nós                         |
| Congelamento longo          | Negócio parado         | Migrar por ondas, pilotos controlados           |
| Vazamento de credenciais    | Segredos em YAML       | Usar Vault/env vars, PR review                  |

### Plano de Testes

* **Fase 0:** inventário + baseline
* **Fase 1:** compatibilidade (lint, análise)
* **Fase 2:** piloto em EEs por domínio
* **Fase 3:** execução templates críticos + workflows ponta-a-ponta
* **Fase 4:** checklist final + auditoria

### Checklist venv → EE

1. Mapear venvs antigos.
2. Agrupar por domínio.
3. Declarar deps (`requirements.yml` / `requirements.txt`).
4. Build/tag dos EEs.
5. Push para registry privado.
6. Atribuir EEs a templates.
7. Smoke test por domínio.
8. Política de ciclo de vida (patch mensal, rebuild trimestral).

### Referências

* Red Hat Blog: \[Planning our migration to AAP2]
* Red Hat Blog: \[How we did it]
* Docs oficiais: \[Migrating virtualenvs to Execution Environments]
* Steampunk Blog: \[AAP2 upgrade prep & planning]

### Outreach (ação recomendada)

* **Red Hat Brasil** (CSA/SA responsável pelo Bradesco)
* **Parceiros locais** (integradores com prática AAP)

Modelo de e-mail já pronto (vide versão anterior).

---

## ⚠️ Pontos de Atenção

* **IDs de objetos** podem mudar entre Tower e AAP → revisar workflows
* **Collections** precisam estar instaladas nos EEs
* **RBAC** pode diferir entre instâncias (Next vs Bradesco)
* **Logs** no AAP são baseados em `ansible-navigator` (diferente do Tower)

---

## 📌 Conclusão

Este guia fornece um processo **end-to-end** para migrar objetos do Tower para o AAP 2.4, cobrindo:

* Inventário inicial
* Preparação de Execution Environments
* Exportação/importação automatizada
* Recriação de credenciais
* Testes, benchmark e checklist final

👉 A partir daqui, **versionar tudo no Git** (`migracao-tower-aap`) 


