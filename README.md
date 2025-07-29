# deploy_kubernetes
Entendendo de deployando Kubernetes


**Kubernetes (K8S)**

 Antes de iniciarmos, resumo - O que é Kubernetes?

Desenvolvida pela Google em 2014 e mantido pela CNCF (Cloud Native Computing Foundation), Kubernetes (também chamado de K8s) é uma plataforma de código aberto criado com objetivo de ser um orquestrador de containers, automatizando a implantação, o escalonamento e a gestão de aplicações em containers.

**Nome Curioso, o apelido k8s veio do padrão i18n (a letra "K" seguida por oito letras e o "s" no final)**


**Arquitetura Kubernetes**



![Arquitetura Kubernetes](imagens/arquitetura.svg)

**Control-plane:** Controle do Cluster!

O Control Plane é o cérebro do cluster Kubernetes. Ele gerencia o estado desejado do cluster, ou seja, define o que deve estar rodando, onde, como e quando.


**Workers:** Trabalho bruto!

Os worker nodes são os servidores que rodam os containers da aplicação (ou seja, os pods).


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








