### **Tutorial Completo: Provisionando Recursos Azure com Crossplane e ArgoCD**

Este tutorial cobre a configuração de um ambiente Kubernetes usando Kind, a integração de Crossplane com o provider Azure, e a configuração de ArgoCD para gerenciar recursos Azure através de GitOps.

---

### **Pré-requisitos**

- **Kind**: Para criar o cluster Kubernetes local.
- **kubectl**: Para interagir com o cluster Kubernetes.
- **ArgoCD CLI**: Para gerenciar aplicações ArgoCD.
- **Helm**: Para instalar o Crossplane.
- **Conta Azure**: Para criar recursos e fornecer credenciais.

---

## **Passo 1: Criar Repositório no GitHub**

1. **Crie um repositório privado no GitHub** com o nome desejado, por exemplo, `poc-crossplane-argocd`.
2. **Gere um Personal Access Token (PAT)**:
    - Acesse: [GitHub PAT Settings](https://github.com/settings/tokens).
    - Defina os escopos:
        - `repo` (acesso ao repositório).
        - `read:org` (se necessário).
    - Anote o token gerado.
3. Exporte as variáveis de ambiente para uso posterior:
    
    ```bash
    export GITHUB_PAT=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    export URL_REPO=https://github.com/<seu-usuario>/poc-crossplane-argocd
    export GITHUB_USERNAME=<seu-usuario>
    
    ```
    

---

## **Passo 2: Configurar o Cluster Kubernetes**

1. **Crie o arquivo `config.yaml` para o Kind**:
    
    ```yaml
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    name: "delaware"
    containerdConfigPatches:
    - |-
      [plugins."io.containerd.grpc.v1.cri".registry]
        config_path = "/etc/containerd/certs.d"
    nodes:
    - role: control-plane
      image: kindest/node:v1.25.9@sha256:c08d6c52820aa42e533b70bce0c2901183326d86dcdcbedecc9343681db45161
      kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
            system-reserved: "cpu=300m,memory=5Mi,ephemeral-storage=1Gi"
            kube-reserved: "cpu=200m,memory=5Mi,ephemeral-storage=1Gi"
            eviction-hard: "memory.available<1Gi,nodefs.available<10%"
      extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
    
    ```
    
2. **Crie o cluster**:
    
    ```bash
    kind create cluster --config config.yaml
    
    ```
    
3. **Instale o ingress-nginx**:
    
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
    
    ```
    
4. **Valide o cluster**:
    
    ```bash
    kubectl get nodes
    kubectl get pods -n ingress-nginx
    
    ```
    

---

## **Passo 3: Instalar o ArgoCD**

1. **Instale o ArgoCD**:
    
    ```bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    
    ```
    
2. **Configure o acesso via ingress**:
    
    Crie o arquivo `ingress.yaml`:
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: argocd-server-ingress
      namespace: argocd
      annotations:
        kubernetes.io/ingress.class: nginx
        nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
        nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    spec:
      rules:
      - host: delaware-argocd.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  number: 443
    
    ```
    
    Aplique o ingress:
    
    ```bash
    kubectl apply -f ingress.yaml
    
    ```
    
3. **Configure a senha do ArgoCD**:
    - Obtenha a senha inicial:
        
        ```bash
        kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
        
        ```
        
    - Altere a senha:
        
        ```bash
        argocd account update-password
        
        ```
        

---

## **Passo 4: Instalar o Crossplane**

1. **Adicione o repositório Helm e instale o Crossplane**:
    
    ```bash
    helm repo add crossplane-stable https://charts.crossplane.io/stable
    helm install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace
    
    ```
    
2. **Instale o provider Azure**:
    
    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: pkg.crossplane.io/v1
    kind: Provider
    metadata:
      name: provider-azure
    spec:
      package: xpkg.upbound.io/crossplane-contrib/provider-azure:v0.20.1
    EOF
    
    ```
    
3. **Crie o segredo com credenciais do Azure**:
    
    Gere o arquivo `azure-creds.json` com o seguinte conteúdo:
    
    ```json
    {
      "clientId": "<seu-client-id>",
      "clientSecret": "<seu-client-secret>",
      "tenantId": "<seu-tenant-id>",
      "subscriptionId": "<seu-subscription-id>",
      "activeDirectoryEndpointUrl": "https://login.microsoftonline.com/",
      "resourceManagerEndpointUrl": "https://management.azure.com/"
    }
    
    ```
    
    Crie o segredo no Kubernetes:
    
    ```bash
    kubectl create secret generic azure-secret -n crossplane-system --from-file=creds=./azure-creds.json
    
    ```
    
4. **Configure o ProviderConfig**:
    
    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: azure.crossplane.io/v1beta1
    kind: ProviderConfig
    metadata:
      name: default
    spec:
      credentials:
        source: Secret
        secretRef:
          namespace: crossplane-system
          name: azure-secret
          key: creds
    EOF
    
    ```
    

---

## **Passo 5: Configurar o ArgoCD**

1. **Crie o segredo para o GitHub**:
    
    ```bash
    kubectl apply -f - <<EOF
    apiVersion: v1
    kind: Secret
    metadata:
      name: poc-crossplane-argocd-gitops-credentials
      namespace: argocd
      labels:
        argocd.argoproj.io/secret-type: repository
    type: Opaque
    stringData:
      name: poc-crossplane-argocd
      url: ${URL_REPO}
      username: ${GITHUB_USERNAME}
      password: ${GITHUB_PAT}
    EOF
    
    ```
    
2. **Configure o AppProject**:
    
    ```bash
    kubectl apply -f - <<EOF
    apiVersion: argoproj.io/v1alpha1
    kind: AppProject
    metadata:
      name: azure-project
      namespace: argocd
    spec:
      description: Project to manage Azure resources
      sourceRepos:
      - ${URL_REPO}
      destinations:
      - namespace: crossplane-system
        server: https://kubernetes.default.svc
      clusterResourceWhitelist:
      - group: azure.crossplane.io
        kind: ResourceGroup
    EOF
    
    ```
    
3. **Configure a aplicação do ArgoCD**:
    
    ```bash
    kubectl apply -f - <<EOF
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: azure-resource-group
      namespace: argocd
    spec:
      project: azure-project
      source:
        repoURL: ${URL_REPO}
        targetRevision: HEAD
        path: azure-resource-group
      destination:
        server: https://kubernetes.default.svc
        namespace: crossplane-system
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    EOF
    
    ```
    

---

## **Passo 6: Validar a Implementação**

1. **Sincronize o ArgoCD**:
    
    ```bash
    argocd app sync azure-resource-group
    
    ```
    
2. **Verifique o status do recurso**:
    
    ```bash
    kubectl get resourcegroups -n crossplane-system
    
    ```
---

## Diagrama draft

![iac-crossplane drawio](https://github.com/user-attachments/assets/34a9b8d6-1721-442d-a59d-f4a6e711ec32)

---

## Testes Lab

![image](https://github.com/user-attachments/assets/d746b93a-d5fd-4027-b4c9-f6fd389a8829)

![image](https://github.com/user-attachments/assets/7ed02e87-30ef-4e7f-aab9-152a714f0430)

![image](https://github.com/user-attachments/assets/8dc8738a-f5ba-47d5-9b64-d11b6267ddcc)

