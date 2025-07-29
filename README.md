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

Dentro do control-plane possuímos outros componentes:

etcd: Banco de dados chave-valor que armazena todo estado do cluster.

API-server: Trabalhando com JSON sobre HTTP, ele é a porta de entrada para gerenciar o Kubernetes. Para facilitar o gerenciamento, podemos integrar ao utilitário kubectl para facilitar a administração via requisições REST.

Scheduler: Responsável pelo nó onde irá armazenar os Pods com base na quantidade de recursos disponíveis em cada nó.

Controller-manager: Executa tarefas que são usadas para monitorar e executar o estado do Pod. Se um pod foi definido com 10 réplicas, o controller-manager irá ler o último estado setado no etcd e o atual estado do Pod, se divergentes, ele irá tentar concilar o pod com o estado estabelecido no etcd.

 

**Workers:** Trabalho bruto!

Os worker nodes são os servidores que rodam os containers da aplicação (ou seja, os pods).

Kubelet: Agent que conversa diretamente com o controll-manager e é executado em todos nós dos Workers com a função de garantir que os containers estejam rodando conforme ao control-plane.

Kube-proxy: Trabalhando como um roteador dos Pods, tem a função de rotear os pacotes de rede dos nós e pods usando IPtables ou IPVS.



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

Exemplo: Falamos para o k8s que precisamos de mais instâncias do Prometheus. O Control-plane irá dizer "Quero 4 instâncias" e os Works irão executar a tarefa.

fluxo
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


# DEPLOY












