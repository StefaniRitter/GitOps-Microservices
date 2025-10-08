# Projeto GitOps na prática
Este projeto foi desenvolvido como parte do programa de bolsas em Cloud & DevSecOps da Compass UOL. Ele demonstra uma pipeline de entrega contínua (CD) moderna e totalmente automatizada, aplicando os princípios de GitOps para implantar um conjunto de microserviços em um ambiente local de Kubernetes.

## Objetivo Principal
Automatizar a implantação de um conjunto de microserviços (Online Boutique) em Kubernetes local usando Rancher Desktop, controlado por GitOps com ArgoCD, a partir de um repositório público no GitHub.

## Tecnologias e Ferramentas Utilizadas

* Rancher Desktop com Kubernetes habilitado e dockerd(moby) como Container Engine;
* Kubectl configurado e apontando para o cluster do Rancher Desktop;
* ArgoCD instalado no cluster;
* Conta no GitHub com repositório público;
* Git instalado;
* Docker funcionando localmente.
  
## Etapa 1: Criação do Repositório no GitHub 

O projeto utiliza a seguinte organização de arquivos, onde os Manifests do Kubernetes são o código de infraestrutura monitorado pelo ArgoCD:
```
gitops-microservices/  
└── k8s/ 
└── online-boutique.yaml 
```

## Etapa 2: Instalação do ArgoCD no Cluster Local (Rancher Desktop)

### Etapa 2.1: Criação do Namespace

Com o Rancher Desktop aberto e o status do cluster mostrando "Kubernetes is running" (pronto para uso), o primeiro passo é criar um Namespace dedicado para o ArgoCD.

Para isso, execute o seguinte comando no seu terminal:
```
kubectl create namespace argocd
```

Saída esperada:

```
namespace/argocd created
```

### Etapa 2.2: Aplicação dos Manifests de Instalação

No terminal, rode o seguinte comando:
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/refs/heads/master/manifests/install.yaml
```
* `kubectl apply`: É o comando padrão do Kubernetes (via kubectl) usado para criar e atualizar recursos do cluster com base em um arquivo de configuração.
* `-n argocd`: É a flag que especifica onde os recursos devem ser criados. Nesse caso, direciona o kubectl para aplicar todos os manifestos dentro do Namespace argocd.
* `-f <URL>`: A flag -f (de file) instrui o kubectl a buscar as configurações em um local externo, que, neste caso, é a URL de um arquivo YAML. Aqui, é crucial usar a **URL RAW** (raw.githubusercontent.com) para obter o conteúdo YAML **puro**.

A saída desse comando será parecida com isso:
```
namespace/argocd created
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
```

Para verificar se os Pods foram criados com sucesso, é só executar o seguinte comando:
```
kubectl get pods -n argocd
```

Você verá algo assim: 
```
NAME                                               READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                    1/1     Running   0          39s
argocd-applicationset-controller-898c766b6-jfcn7   1/1     Running   0          39s
argocd-dex-server-66585dc685-927r6                 1/1     Running   0          39s
argocd-notifications-controller-7c584f65cc-f45b2   1/1     Running   0          39s
argocd-redis-8667b66f58-shh7p                      1/1     Running   0          39s
argocd-repo-server-68f8c484db-g8mpz                1/1     Running   0          39s
argocd-server-5cc7485d86-2rkgj                     1/1     Running   0          39s
```

## Etapa 3: Instalação da CLI do ArgoCD (opcional)

### Etapa 3.1: Instalação do executável
A CLI (Interface de Linha de Comando) permite interagir com o servidor ArgoCD. O comando abaixo utiliza o PowerShell (Invoke-WebRequest) para baixar o binário correto para o Windows.
```
Invoke-WebRequest -Uri "https://github.com/argoproj/argo-cd/releases/latest/download/argocd-windows-amd64.exe" -OutFile argocd.exe
```

* `Invoke-WebRequest`: Baixa o conteúdo de uma URL. É a forma nativa do PowerShell de fazer requisições HTTP (como o curl no Linux).
* `-Uri <URL>`: Endereço de Origem. Nesse caso, especifica o link para o binário do ArgoCD compilado para Windows (amd64).
* `-OutFile argocd.exe`: Nome do Arquivo de Destino. Salva o arquivo baixado no diretório atual com o nome argocd.exe

### Etapa 3.2:  Adicionando a CLI do ArgoCD ao PATH

Após baixar o argocd.exe, é recomendado movê-lo para um diretório que esteja incluído na variável de ambiente PATH do sistema. 

O trecho de código a seguir cria uma pasta C:\bin se ela ainda não existir:
```
if (-not (Test-Path C:\bin)) { New-Item -Path C:\bin -ItemType Directory }
```


Para mover o executável para a nova pasta, é só rodar o seguinte comando:
```
Move-Item -Path .\argocd.exe -Destination C:\bin\argocd.exe
```

* IMPORTANTE: Para que o Windows encontre o argocd.exe no C:\bin, é necessário adicionar C:\bin à variável de ambiente PATH do sistema.

### Etapa 3.3: Verificação

Para verificar se a instalação deu certo, rode o seguinte comando:
```
argocd version
```

A saída deve ser algo parecido com isso:
```
argocd: v3.1.8+becb020
  BuildDate: 2025-09-30T16:04:21Z
  GitCommit: becb020064fe9be5381bf6e5818ff8587ca8f377
  GitTreeState: clean
  GoVersion: go1.24.6
  Compiler: gc
  Platform: windows/amd64
{"level":"fatal","msg":"Failed to establish connection to localhost:8080: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.","time":"2025-10-08T08:59:10-03:00"}
```

O erro **"Failed to establish connection to localhost:8080..."** é um erro esperado nessa fase! Acontece que, ao rodar o comando `argocd version`, a CLI tenta se conectar ao servidor do ArgoCD para verificar a versão dele também (e não apenas a versão do cliente). Por padrão, a CLI tenta se conectar ao endereço localhost:8080, o que causa um erro, pois o servidor ArgoCD ainda não está escutando na porta 8080 do localhost. Esse erro será resolvido na próxima etapa!

## Etapa 4: Acessar ArgoCD localmente 

Para acessar a interface web (UI) e a CLI do ArgoCD a partir do navegador/terminal, é necessário abrir uma conexão entre o localhost e o Service do ArgoCD.

### Etapa 4.1: Encaminhamento de Porta (Port Forwarding)

Execute este comando no terminal:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
O comando acima encaminha o tráfego da porta 443 do serviço (svc) chamado `argocd-server` no Namespace `argocd` para a porta 8080 no localhost.
É necessário manter o terminal **aberto**, pois o port-forward é um processo contínuo e deve ficar rodando.

Nesse momento, já é possível ver o ArgoCD rodando no navegador através do endereço `https://localhost:8080`:


<img width="1916" height="886" alt="image" src="https://github.com/user-attachments/assets/4b7722b9-b20d-4ccc-929e-53f00305371d" />


### Etapa 4.2: Obtendo a Senha Inicial

Se você conseguir rodar o ArgoCD via navegador ou terminal como mostra na etapa anterior, verá que é necessário informar usuário e senha para o login. 
O usuário padrão do ArgoCD é o `admin`, e a senha inicial está armazenada em um secret. Para pegar a senha, basta executar o seguinte comando em um segundo terminal:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | %{[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_))}
```
Copie e cole a saída, que será sua senha temporária.

### 4.3. Acesso e Login

Com o Port Forwarding ativo (mantenha o primeiro terminal aberto), existem duas formas de acessar o servidor: pela interface web (UI) e pela CLI.

1. Acesso via Interface Web (UI)

* Acessar a UI: Abra o navegador e vá para https://localhost:8080.
* Ignorar Certificado: Você receberá um aviso de segurança. Ignore-o (prossiga para a página) para acessar a UI.
* Fazer Login: Informe o usuário admin e a senha que você copiou na etapa anterior.

<img width="377" height="487" alt="image" src="https://github.com/user-attachments/assets/b8d747e0-1215-464b-9f68-43b403654f20" />

E pronto! O ArgoCD está configurado e a interface deve aparecer:

<img width="1908" height="892" alt="image" src="https://github.com/user-attachments/assets/ba6bccef-39b0-406d-bf6d-8cf94d904f51" />

 
2. Acesso via Terminal (CLI)

Para logar no servidor ArgoCD e começar a gerenciá-lo pela linha de comando, execute:
```
argocd login localhost:8080
```

O ArgoCD irá avisar sobre o certificado autoassinado (o que é normal em ambiente local) e pedirá suas credenciais:
```
WARNING: server certificate had error: tls: failed to verify certificate: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Username: admin
Password:
'admin:login' logged in successfully
Context 'localhost:8080' updated
```

## Etapa 5: Criar o App no ArgoCD
