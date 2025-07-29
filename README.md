# deploy_kubernetes
Entendendo de deployando Kubernetes


**Kubernetes (K8S)**

 Antes de iniciarmos, resumo - O que é Kubernetes?

Desenvolvida pela Google em 2014 e mantido pela CNCF (Cloud Native Computing Foundation), Kubernetes (também chamado de K8s) é uma plataforma de código aberto criado com objetivo de ser um orquestrador de containers, automatizando a implantação, o escalonamento e a gestão de aplicações em containers.

***Nome Curioso, o apelido k8s veio do padrão i18n (a letra "K" seguida por oito letras e o "s" no final)***


### **Arquitetura Kubernetes**

Um Cluster K8s segue o modelo construido por control plane / workers, onde o control plane é o cérebro do cluster e o trabalho bruto é executado dentro dos Workers.

![Arquitetura Kubernetes](imagens/arquitetura.svg)

**Control-plane:** Controle do Cluster!

O Control Plane é o cérebro do cluster Kubernetes. Ele gerencia o estado desejado do cluster, ou seja, define o que deve estar rodando, onde, como e quando.

Dentro do **control-plane** possuímos outros componentes:

**etcd:** Banco de dados chave-valor que armazena todo estado do cluster.

**API-server:** Trabalhando com JSON sobre HTTP, ele é a porta de entrada para gerenciar o Kubernetes. Para facilitar o gerenciamento, podemos integrar ao utilitário kubectl para facilitar a administração via requisições REST.

**Scheduler:** Responsável pelo nó onde irá armazenar os Pods com base na quantidade de recursos disponíveis em cada nó.

**Controller-manager:** Executa tarefas que são usadas para monitorar e executar o estado do Pod. Se um pod foi definido com 10 réplicas, o controller-manager irá ler o último estado setado no etcd e o atual estado do Pod, se divergentes, ele irá tentar concilar o pod com o estado estabelecido no etcd.

 

**Workers:** Trabalho bruto!

Os worker nodes são os servidores que rodam os containers da aplicação (ou seja, os pods).

**Kubelet:** Agent que conversa diretamente com o controll-manager e é executado em todos nós dos Workers com a função de garantir que os containers estejam rodando conforme ao control-plane.

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



### Gerenciamento dos Containers (Pod - Menor unidade do Kubernetes)

**Pod:**
Para lidar com os containers, o K8s trabalha com uma abstração chamada de Pod. K8s não acessa o container diretamente, ele organiza dentro de um grupo denominados **Pods**, onde um Pod pode ter vários containers dividindo recursos de memórias, CPU, volumes, endereços...


**Deployment:** Um dos principais controllers do K8s, ele gerencia o ciclo de vida do Pod por completo, garantindo que um Pod tenha o número de réplicas determinadas dentro dos workers do cluster. Ele também cuida de outras caracteristicas do Pod, como Portas, imagem, atualizações, Volumes e variáveis de ambiente, outra caracteristica é que ele também ajuda se integra ao ReplicaSet para garantir alta disponibilidade.


**ReplicaSets:** Responsável por garantir a quantidade de Pods em execução dentro do **nó**.

**Services:** Função de expor os Pods na rede permitindo uma comunicação da rede Externa para dentro da Lan do Cluster.

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



Deploy KinD
````
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
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


Como o KinD utiliza Docker, com o comando **Docker ps** podemos visualizar o container criado.

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

Deployment nginx
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

**Deletando CLUSTER KIND**

````
kind delete cluster --name meu-cluster
````
![delete](imagens/deletes.png)

Cluster KinD Excluído!

-----------
### Criando Cluster personalizados via manifest YAML

Outra forma de trabalhar com o Kind é através de manifest .yaml onde podemos definindo parametros de porta, redes, quantidades de nodes e Workers.

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
        hostPort: 30080    # 🔥 necessário para acessar NodePort 30080

  - role: worker
````

Deploy

Irei realizar o deploy com o comando kind create cluster --name nome_do_cluster --config meu_manifest.yaml
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

Manifest nginx_kubernetes.yaml
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
![namespace](imagens/curl.png)

**LEMBRE-SE** - No manifest para deploy do cluster *kind-config.yaml* eu passei o parametro para expor a porta:

````
- containerPort: 30080
        hostPort: 30080  
````
Acesso Web Nginx:

![namespace](imagens/nginx.png)




----------
