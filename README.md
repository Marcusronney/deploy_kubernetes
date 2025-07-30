# deploy_kubernetes

**Kubernetes (K8S)**

 Antes de iniciarmos, resumo - O que é Kubernetes?

Desenvolvida pela Google em 2014 e mantido pela CNCF (Cloud Native Computing Foundation), Kubernetes (também chamado de K8s) é uma plataforma de código aberto criado com objetivo de ser um orquestrador de containers, automatizando a implantação, o escalonamento e a gestão de aplicações em containers.

***Nome Curioso, o apelido k8s veio do padrão i18n (a letra "K" seguida por oito letras e o "s" no final)***


### **Arquitetura Kubernetes**

Um Cluster K8s segue o modelo construido por control plane / workers, onde o control plane é o cérebro do cluster e o trabalho bruto é executado dentro dos Workers.

![Arquitetura Kubernetes](imagens/arquitetura.svg)

## Cluster

Um Cluster Kubernetes é um conjunto de nodes(nós) que trabalham em conjunto para executar os Pods.

O Cluster é composto por Nodes que podem ser tanto **control plane** quanto **workers**. 

Uma ótimoa analogia seria uma orquestra, onde temos uma pessoa regendo a orquestra, que é o control plane, e temos as pessoas musicistas, que estão executando os instrumentos, que são os workers.

-----

**Control-plane:** Controle do Cluster!

O Control Plane é o cérebro do cluster Kubernetes. Ele gerencia o estado desejado de todo ocluster, ou seja, define o que deve estar rodando, onde, como e quando.

Dentro do **control-plane** possuímos outros componentes:

  **etcd:** Banco de dados chave-valor que armazena todo estado do cluster.

  **API-server:** Trabalhando com JSON sobre HTTP, ele é a porta de entrada para gerenciar o Kubernetes. Para facilitar o gerenciamento, podemos integrar ao utilitário kubectl para facilitar a administração via requisições REST.

  **Scheduler:** Responsável pelo nó onde irá armazenar os Pods com base na quantidade de recursos disponíveis em cada nó.

  **Controller-manager:** Executa tarefas que são usadas para monitorar e executar o estado do Pod. Se um pod foi definido com 10 réplicas, o controller-manager irá ler o último estado setado no etcd e o atual estado do Pod, se divergentes, ele irá tentar concilar o pod com o estado estabelecido no etcd.

 ---

**Workers:** Trabalho bruto!

Os worker nodes são os servidores que rodam os containers da aplicação (ou seja, os pods).

  **Kubelet:** Agent que conversa diretamente com o control-manager e é executado em todos nós dos Workers com a função de garantir que os containers estejam rodando conforme ao definido no control-plane.

  **Kube-proxy:** Trabalhando como um roteador dos Pods, tem a função de rotear os pacotes de rede dos nós e pods usando IPtables ou IPVS.



arquitetura
````
[Control Plane]
├─ kube-apiserver       ← Interface de controle
├─ etcd                 ← Armazena estado
├─ kube-scheduler       ← Decide onde rodar os pods
└─ kube-controller-mgr  ← Garante o estado desejado

[Worker Nodes]
├─ kubelet              ← Executor de pods
├─ kube-proxy           ← Roteamento de rede
└─ Container Runtime    ← Executa containers
````

**Exemplo:** Falamos para o k8s que precisamos de mais instâncias do Prometheus. O Control-plane irá dizer "Quero 4 instâncias" e os Works irão executar a tarefa.

FLUXO
````
kubectl apply -f app.yaml ───▶ kube-apiserver ───▶ etcd (salva estado)
                                  │
                                  ▼
                        kube-scheduler decide node
                                  │
                                  ▼
                   kubelet no node executa pod com o container runtime
                                  │
                                  ▼
                       kube-proxy gerencia o tráfego para os pods
````

# Cluster



----------

### Gerenciamento dos Containers

**Pod - Menor unidade do Kubernetes**

Para lidar com os containers, o K8s trabalha com uma abstração chamada de Pod. K8s não acessa o container diretamente, ele organiza dentro de um grupo denominados **Pods**, onde um Pod pode ter vários containers dividindo recursos de memórias, CPU, volumes, endereços...

---------------------------------

### **Deployment:** 

Um dos principais controllers do K8s, ele gerencia o ciclo de vida do Pod por completo, garantindo que um Pod tenha o número de réplicas determinadas dentro dos workers do cluster.

Ele também cuida de outras caracteristicas do Pod, como Portas, imagem, atualizações, Volumes e variáveis de ambiente.

Outra caracteristica é que ele também se integra e gerencia o ReplicaSet para garantir alta disponibilidade, portanto, ao criar um Deployment, automaticamente é criado um **ReplicaSet**.

**Como criar um Deployment?**

Para criar um Deployment, é necessário um arquivo YAML definindo todos os parâmetros de configuração.

**deployment.yaml**
````
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
````

**Explicação:**

No campo **Kind:** estou definindo o tipo de objeto como Deploymeny e a versão da API em **apiVersion:**
````
apiVersion: apps/v1
kind: Deployment
````

Aqui defino os **metada:** e dou etiquetas aos objetos com **labels:**. Assim eu consigo utilizar essas etiquetas para configurações posteriormanete.
````
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
````

Aqui defino o número de réplicas para 3.
````
spec:
  replicas: 3
````

Aqui estou definindo que todos os Pods que possuírem a etiqueta "nginx-deployment" serão gerenciados por este deployment.
````
selector:
  matchLabels:
    app: nginx-deployment
````

Este campo **strategy** defini qual estratégia será usada para Update dos Pods, quando vázio **{}** a estratégia definida por padrão é **RollingUpdate**.

**RllingUpdate** é utilizada para atualizar os Pods de um Deployment de forma gradual, ou seja, ela atualiza um Pod por vez, ou um grupo de Pods por vez.
````
strategy: {}
````

Definindo o Template dos Pods que serão criados pelo Deployment. Template é o modelo que os Pods devem ser construídos. 

Ao definir no campo acima "** spec: \ selector: \ matchLabels: \ app: nginx-deployment**" o Deployment irá procurar por todos os Pods com o rótulo "**nginx-deployment**". No campo templates: eu defino o mesmo rótulo para o deployment se conectar aos Pods criados.
````
template:
  metadata:
    labels:
      app: nginx-deployment
````

Aqui estou definindo que os Pods terão a imagem do **Nginx**.
````
  spec:
    containers:
    - image: nginx
      name: nginx
````
Aqui eu defino os Recursos de Memória e Hardware.
No campo **limits** é definido a quantidade máxima que cada container pode usar. E em **requests** é definido a quantidade reservada de CPU e Memória.
````
      resources:
        limits:
          cpu: "0.5"
          memory: 256Mi
        requests:
          cpu: 0.25
          memory: 128Mi
````

**Para aplicar o deployment:** *kubectl apply -f nome_do_deployment.yaml

````
kubectl apply -f deployment.yaml
````

Verificando o deployment criado:
````
kubectl get deployments -l app=nginx-deployment
````

![nginx](imagens/nginxdeploymeny.png)


Posso também verificar todos os Pods que o Deployment está gerenciando.

````
kubectl get pods -l app=nginx-deployment
````
![nginx](imagens/nginx2.png)

----------------------

### **ReplicaSets:** 

Responsável por garantir a quantidade de Pods em execução dentro do **nó**. Ao criar um deployment, atumaticamente um ReplicaSet é criado e esse ReplicaSet irá criar os Pods listados dentro do Deployment.

**Listando os Pods do ReplicaSet criados pelo Deployment nginx-deployment**
````
kubectl get replicasets -l app=nginx-deployment
````
Como o ReplicaSet cria os Pods, podemos ver a quantidade de Pods criados.

![replicaset](imagens/nginxreplica.png)

**Listando todos ReplicaSet do Cluster:**

````
kubectl get replicasets
````

**Apagando ReplicaSet**
````
kubectl delete replicaset nginx-replicaset
````
-----------------------

# DaemonSet

DaemonSet tem a função de manter uma cópia do Pod em todos os Nós do Cluster. Muito usado para tarefas de infraestrutura, um exemplo de uso é quando precisamos manter um Pod de monitoramento em todos os Nós do Cluster.

Portanto, se um cluster possuir 3 nós, o DemonSet irá garantir que todos os nós executem uma réplica do Pod.

**Criando um DaemonSet**


````
nano node-exporter-daemonset.yaml
````


node-exporter-daemonset.yaml
````
apiVersion: apps/v1 # Versão da API do Kubernetes do objeto
kind: DaemonSet # Tipo do objeto
metadata: # Informações sobre o objeto
  name: node-exporter # Nome do objeto
spec: # Especificação do objeto
  selector: # Seletor do objeto
    matchLabels: # Labels que serão utilizadas para selecionar os Pods
      app: node-exporter # Label que será utilizada para selecionar os Pods
  template: # Template do objeto
    metadata: # Informações sobre o objeto
      labels: # Labels que serão adicionadas aos Pods
        app: node-exporter # Label que será adicionada aos Pods
    spec: # Especificação do objeto, no caso, a especificação do Pod
      hostNetwork: true # Habilita o uso da rede do host, usar com cuidado
      containers: # Lista de contêineres que serão executados no Pod
      - name: node-exporter # Nome do contêiner
        image: prom/node-exporter:latest # Imagem do contêiner
        ports: # Lista de portas que serão expostas no contêiner
        - containerPort: 9100 # Porta que será exposta no contêiner
          hostPort: 9100 # Porta que será exposta no host
        volumeMounts: # Lista de volumes que serão montados no contêiner, pois o node-exporter precisa de acesso ao /proc e /sys
        - name: proc # Nome do volume
          mountPath: /host/proc # Caminho onde o volume será montado no contêiner
          readOnly: true # Habilita o modo de leitura apenas
        - name: sys # Nome do volume 
          mountPath: /host/sys # Caminho onde o volume será montado no contêiner
          readOnly: true # Habilita o modo de leitura apenas
      volumes: # Lista de volumes que serão utilizados no Pod
      - name: proc # Nome do volume
        hostPath: # Tipo de volume 
          path: /proc # Caminho do volume no host
      - name: sys # Nome do volume
        hostPath: # Tipo de volume
          path: /sys # Caminho do volume no host
````

Aplicando o DaemonSet
````
kubectl apply -f node-exporter-daemonset.yaml
````

Verificando o DaemonSet criado
````
kubectl apply -f node-exporter-daemonset.yaml
````

Exibir Pods gerenciados pelo DaemonSet
````
kubectl get pods -l app=node-exporter
````



--------------------

# Probes

Probes são uma forma de monitorar os Pods automaticamente com a finalidade de saber o estado de vida do Pod. Através do status do Pod, decisções são tomadas como por exemplo: Reiniciar containers, remover containers, saber se deve ou não enviar dados para container.


As Probes são dividas em 3 tipos: **LivenessProbe**, **ReadinessProbe** e **StartupProbe**.

### LivenessProbe.

A LivenessProble é responsável por verificar a integridade do Pod, através dela é possível ver oque está rodando dentro do Pod e se está respondendo conforme o esperado.

### ReadinessProbe.

ReadinessProbe é usada para verificar se o container está pronto para receber tráfego e requisições da rede externa.

### StartupProbe

StartupProbe é responsável por verificar se o container foi incializado corretamente e se está pronto para operar. Ela é semelhante a readinessProbe, a única diferença é que ela é executado uma única vez no começo de vida do conteiner.



````
nano nginx-liveness.yaml
````
nginx-liveness.yaml
````
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
        livenessProbe: # Aqui é onde vamos adicionar a nossa livenessProbe
          tcpSocket: # Aqui vamos utilizar o tcpSocket, onde vamos se conectar ao container através do protocolo TCP
            port: 80 # Qual porta TCP vamos utilizar para se conectar ao container
          initialDelaySeconds: 10 # Quantos segundos vamos esperar para executar a primeira verificação
          periodSeconds: 10 # A cada quantos segundos vamos executar a verificação
          timeoutSeconds: 5 # Quantos segundos vamos esperar para considerar que a verificação falhou
          failureThreshold: 3 # Quantos falhas consecutivas vamos aceitar antes de reiniciar o contai
````




-------------------

**Services:** Função de expor os Pods na rede permitindo uma comunicação da rede Externa para dentro da Lan do Cluster.

---------


### **Tipos de Services:**

    ClusterIP: Padrão ao ser deployado no cluster, tem a função de expor serviços somente dentro do próprio cluster, fazendo uma comunicação interna entre os serviços.

    exemplo:

    spec:
        type: ClusterIP
        ports:
          - port: 80
           targetPort: 8080


-

    NodePort: Geralmente usada entre as portas 30000 - 32767, tem a função de expor o pod para a rede Externa.

    exemplo:

    spec:
      type: NodePort
      ports:
        - port: 80
          targetPort: 8080
          nodePort: 30080



-

    LoadBalander: Usado para expor aplicações diretamente na Wan como um balanceador de carga. 

    exemplo:

    spec:
      type: LoadBalancer
      ports:
        - port: 80
          targetPort: 8080




**Portas K8s**
````
| Componente           | Porta     | Protocolo | Descrição                  |
| -------------------- | --------- | --------- | -------------------------- |
| `kube-apiserver`     | 6443      | HTTPS     | Principal API do cluster   |
| `etcd`               | 2379–2380 | HTTPS     | Armazena estado do cluster |
| `kube-scheduler`     | 10259     | HTTPS     | Comunicação interna        |
| `controller-manager` | 10257     | HTTPS     | Gerencia controladores     |
| `kubelet`            | 10250     | HTTPS     | Comunicação com o nó       |
| `kube-proxy`         | 10256     | HTTP      | Gerencia regras de rede    |
````

------------


# DEPLOY CLUSTER KUBERNETES

Agora que já foi explicado os principais componentes do K8s. Irei criar meu cluster K8s localmente usando o KinD.

[KinD](https://github.com/kubernetes-sigs/kind) (Kubernetes IN Docker) é uma ferramenta que permite rodar clusters Kubernetes inteiros dentro de containers Docker, de forma rápida, leve e local. Ele criado principalmente para desenvolvimento, testes e CI/CD, permitindo simular um ambiente Kubernetes sem precisar de VMs, cloud, ou uma instalação pesada. Outra forma de efetuar deploy do K8s localmente é com o [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download).

Irei instalar o KinD seguindo os requisitos da documentação, como ele usa containers Docker para criar os clusters, irei instalar o Docker.

Instalando Docker:
````
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io
sudo systemctl enable --now docker
````

Permissão ao usuário para usar o Docker
````
sudo usermod -aG docker $USER
newgrp docker
````

![Docker](https://img.icons8.com/?size=100&id=22813&format=png&color=000000)

Docker Instalado!

![Docker](imagens/dockerversion.png)


------------------------

Deploy KinD
````
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
````
Desligando swap (recomendo em ambientes K8s)
````
sudo swapoff -a
````

Rode ***kind version*** para verificar a instalação.

![Docker](imagens/kindversion.png)


KinD instalado com sucesso!

Para facilitar o gerenciamento do K8s, irei utilizar o utilitário ***kubectl***.

Instalando kubectl:
````
curl -LO https://dl.k8s.io/release/v1.33.0/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
````
````
kubectl version --client
````

![kubectl](imagens/kubectlversion.png)


Dependências instaladas com sucesso! Agora irei iniciar o processo de criação do Cluster.

````
kind create cluster --name meu-cluster
````
Cluster Criado.

![kind](imagens/kind.png)

Verificando o Cluster:
````
kind get clusters
````

![docker](imagens/clusters.png)


Como o KinD utiliza Docker, com o comando "**Docker ps**" podemos visualizar o container criado.

![docker](imagens/dockerps.png)

Verificando informações do Cluster:
````
kubectl cluster-info
````
O Control Plane está rodando localmente (Kind usa localhost com uma porta aleatória).O DNS interno (CoreDNS) está ativo — essencial para comunicação entre pods.

![kind](imagens/info.png)


Verificando Nodes(Nós):
````
kubectl get node
````
Um único node (control-plane), com a versão v1.29.2. Está com status Ready, ou seja: funcional e aceitando workloads.

![kind](imagens/node.png)

Verificando todos os Pods:
````
kubectl get pods -A
````
Aqui posso ver todos os Pods em Status "Running" no meu cluster.

![kind](imagens/pods.png)

````
Pod	Função
coredns-xxxx	Resolve nomes DNS dentro do cluster
etcd-meu-cluster-control-plane	Banco de dados do cluster
kube-apiserver-meu-cluster...	API do cluster
kube-controller-manager-...	Controla estados de recursos
kube-scheduler-...	Decide onde rodar pods
kube-proxy	Roteamento de rede
kindnet	Plugin de rede (CNI) usado pelo Kind
local-path-provisioner	Provisionador de volumes persistentes localmente
````

**Cluster Rodando!**

Agora irei instalar uma aplicação do nginx como teste de deployment com o comando ***kubectl create nome_do_pod --imagem=***.

**Deployment nginx**
````
kubectl create deployment nginx --image=nginx
````
Ao utilizar ***kubectl create*** o container nginx é baixado diretamente do Docker Hub, que é o repositório público padrão de imagens Docker. Ao passar parametro "--image=nginx" a imagem será buscada em "*docker.io/library/nginx:latest*".

O Kubectl envia o comando para criar o Deployment para o KubeAPI dentro do Control-plane, o control-plane envia a solicitação para o Kubelet receber dentro do Worker. Após a requisição ser recebida, o container runtime irá começar a baixar a imagem direto do Docker Hub.


Ao criar um novo deployment, o status do container fica como "ContainerCreating".

![deployment](imagens/deployment.png)

Após esperar alguns segundos, o status muda para "Running".

![deployment](imagens/deployment2.png)

Criando Serviço na porta 80 como NodePort.
````
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get svc
````

![service](imagens/service.png)


Aplicação Deployada e Service criados com sucesso!

Agora irei deletar os pods criados.

**Deletando deployment:** kubectl delete deployment nginx  

**Deletando service:** kubectl delete service nginx  


Posso verificar se ainda existe algum deployment ou service do nginx com **kubectl get ALL**

Podemos notar que todos pods nginx foram excluídos.

![service](imagens/delete.png)

-------------------


**Deletando CLUSTER KIND**

````
kind delete cluster --name meu-cluster
````

![delete](imagens/deletes.png)

Cluster KinD Excluído!

-----------
### Criando Cluster personalizados via manifest YAML

Outra forma de trabalhar com o Kind é através de manifest **.yaml** onde podemos definir parâmetros de porta, redes, quantidades de nodes e Workers.

Irei criar um novo arquivo .yaml definindo as configurações do Cluster.

````
nano kind-config.yaml
````
kind-config.yaml
````
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: "192.168.2.63"  # IP do host físico
  apiServerPort: 17443
  podSubnet: "10.244.0.0/16"
  serviceSubnet: "10.96.0.0/12"

nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
      - containerPort: 443
        hostPort: 443
      - containerPort: 6443
        hostPort: 17445
      - containerPort: 30080
        hostPort: 30080  

  - role: worker
````

Deploy

Irei realizar o deploy com o comando "**kind create cluster --name nome_do_cluster --config meu_manifest.yaml**"
````
kind create cluster --name meu-cluster --config kind-config.yaml
````

**Cluster deployado via manifest, os Pods estão sendo criados.**

![Kind](imagens/kind2.png)



----------------------

### NAMESPACE

Como já sabemos, Docker ou outros tipos de containers são isolamentos de recursos, onde o Linux mente para a aplicação fazendo ela executar como se estivesse em outra máquina com seus próprios recursos de soft e hardware.

As Namespace são as responsáveis por este papel, elas são uma divisão lógica do Linux onde permite isolar recursos, dentro do Cluster podemos usar namespaces para ter vários ambientes, pods, services, deployments...

Podemos verificar todas as namespaces do cluster:
````
kubectl get namespaces
````
Ao efetuar deploy do KinD, ele já cria todas as namespaces necessários para o cluster automaticamente.
![namespace](imagens/namespace.png)

Para verificar os Pods de uma namespace: **kubectl get pods -n nome_da_namespace

Verificando pods da namespace kube-system:
````
kubectl get pod -n kube-system
````

![namespace](imagens/kubesystem.png)


Para verificar todos pods de todas namespaces:
````
kubectl get pods -A
````

-----------------------------------

### Deploy Nginx no Cluster via manifest YAML

````
nano nginx_kubernetes.yaml
````

**Manifest nginx_kubernetes.yaml**
````
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: nginx

---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: nginx
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080  # Porta acessível no host
````
**Realizando o deployment:** kubectl apply -f nome_do_manifest.yaml
````
kubectl apply -f nginx_kubernetes.yaml
````
![namespace](imagens/deploynginx.png)

Com o comando **kubectl get all -n nginx** eu posso visualizar todos os pods da namespace nginx.

````
kubectl get all -n nginx
````

**Deployment** e **Services** nginx criados com sucesso.
![namespace](imagens/allnginx.png)


Testando Pods Nginx:

Com os Pods em Running, o Service está apontando para a porta 30080 e o Type está como NodePort, ou seja, o Pod do Nginx está sendo exposto para rede.
````
curl https://127.0.0.1:30080
````

Curl retornando a página web do nginx.

![namespace](imagens/curl.png)

**LEMBRE-SE** - No manifest para deploy do cluster *kind-config.yaml* eu passei o parametro para expor a porta:

````
- containerPort: 30080
        hostPort: 30080  
````
**Acesso Web Nginx:**

![namespace](imagens/nginx.png)

----------

# Volumes no Kubernetes

Os Volumes no Kubernetes são diretórios dentro do Pod utilizados para armazenar os dados. Existem 2 tipos de volumes, os **ephemeral volumes** e os **persistent volumes**.

**Ephemeral Volumes** = São volumes criados e destruídos junto com o Pod, qualquer problema com o Pod ele será excluído junto.

**Persistent** = São volumes que são criados e não são destruídos quando o Pod morre, eles são persistidos e os dados são mantidos mesmo quando o Pod é excluído.



## Volumes Storage Class

**Storage Class** é um objeto que descreve classes de armazenamento disponível no Cluster. Ela é útil para gerenciar diferentes tipos de armazenamento e também pode ser usada para definir políticas de retenção, provisionamento e outras características de armazenamento.

Cada StorageClass é definida com um provisionador, que é responsável por criar PersistentVolumes dinamicamente conforme necessário. Os provisionadores podem ser internos (fornecidos pelo próprio Kubernetes) ou externos (fornecidos por provedores de armazenamento específicos).

Para ver a lista completa de provisionadores, consulte a documentação do Kubernetes no link https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner.

Verificando o Storage Class disponível no Cluster:
````
kubectl get storageclass
````

Como estou no KinD, o padrão é o "rancher.io/local-path" que cria volumes **PersistentVolume** no diretório do host.

![storageclass](imagens/storageclass.png)

Posso usar o **Describe** para ver os detalhes do Storage Class:
````
kubectl describe storageclass standard
````

![storageclass](imagens/storage2.png)


## PV - Persistent Volume

PV é um objeto no K8s que representa um armazenamento físico dentro do cluster. Ele pode ser um HD, SSD, NAS ou algum serviço Cloud como, por exemplo o AWS EBS.

Ele é utilizado para fornecer armazenamento durável, os dados são armazenados mesmo quando o Pod morre, reiniciados ou movidos de Node.

Consultar Persistent Volume disponível:
````
kubectl get pv -A
````
Como o PV não é ativo por default, nenhum armazenamento PV é encontrado.

![pv](imagens/pv.png)


### Criando Armazenamento PV - Persistent Volume



pv.yaml
````
apiVersion: v1 # Versão da API do Kubernetes
kind: PersistentVolume # Tipo de objeto que estamos criando, no caso um PersistentVolume
metadata: # Informações sobre o objeto
  name: meu-pv # Nome do nosso PV
  labels:
    storage: local
spec: # Especificações do nosso PV
  capacity: # Capacidade do PV
    storage: 1Gi # 1 Gigabyte de armazenamento
  accessModes: # Modos de acesso ao PV
    - ReadWriteOnce # Modo de acesso ReadWriteOnce, ou seja, o PV pode ser montado como leitura e escrita por um único nó
  persistentVolumeReclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
  hostPath: # Tipo de armazenamento que vamos utilizar, no caso um hostPath
    path: "/mnt/data" # Caminho do hostPath, do nosso nó, onde o PV será criado
  storageClassName: standard # Nome da classe de armazenamento que será utilizada
  ````

Aplicando pv.yaml
````
kubectl apply -f pv.yaml
````

![pv](imagens/pv2.png)


Listando PV do Cluster
````
kubectl get pv
````

![pv](imagens/pv3.png)

Visualizando os detalhes do PV
`````
kubectl describe pv meu-pv
````

![pv](imagens/pv4.png)
