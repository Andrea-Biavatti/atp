# atp

> ¯\_(ツ)_/¯

## Specs:

Quantidade total de usuários da aplicação: 2.000.

Quantidade de usuários simultâneos: 200.


### Aplications:

>Cada usuário cadastrado utiliza 1 GB de espaço de armazenamento.

>Vamos trabalhar com discos
internos no servidor configurados em RAID 1.

>Cada usuário simultâneo utiliza 1 GB de memória no servidor de aplicação.

>Cada grupo de cem usuários simultâneos consome 20 vCPUs (considere que cada CORE de processador
equivale a 4 vCPUs).

>Cada grupo de cem usuários simultâneos consome 10 GbE de largura de banda de rede.


### DB

>Cada usuário cadastrado utiliza 5 GB de espaço de armazenamento. Vamos trabalhar com discos
internos no servidor configurados em RAID 5.

>Cada usuário simultâneo utiliza 2 GB de memória no servidor de banco de dados.

>Cada grupo de cem usuários simultâneos consome 40 vCPUs (considere que cada CORE de processador
equivale a 4 vCPUs).

>Cada grupo de cem usuários simultâneos consome 10 GbE de largura de banda de rede

## 

Planilha de calculo
> https://drive.google.com/file/d/1rNQVHcR5xK_JJpK3asnv8P4jpxjieOnL/view?usp=sharing


# Projeto 

Kubernetes auto gerenciado - AKS 

Para o projeto, foi utilizada a nuvem Azure com o recurso de serviço autogerenciado do Kubernetes (AKS). Foi escolhido o AKS pela facilidade em não se ter que criar os servidores de nós e master na mão, somente se preocupando em subir a aplicação. 

Porque Kubernetes? 
muito se fala em subirmos aplicações em um servidor de app e em outro de banco de dados, mas muita das vezes as pessoas ficam recesosas devido aos custos, uma vez que o servidor rodando tem um valor X e é imutavel. 

Conforme a tabela abaixo temos os valores das instancias requiridas pelas especificações e o valor das mesmas rodando em AKS

| Service type      | Name      | Description      | Estimated monthly cost  | 
| ------------- |:-------------:| -----:| -----:|
|Azure Kubernetes Service (AKS)     | Kubernetes (aplicação) | 1 D2 v3 (2 vCPUs, 8 GB de RAM) nós x 730 Horas; PAGO CONFORME O USO; 0 discos de SO gerenciados – E40, 0 clusters | 85,41|
| Virtual Machines    | SRVApplications     |   1 D96 v5 (96 vCPUs, 384 GB de RAM) x 730 Horas; Linux – Ubuntu| 2733,42|
| Virtual Machines | SRVDatabses     | 1 D96 v5 (96 vCPUs, 384 GB de RAM) x 730 Horas; Linux – Ubuntu|1962,29|
| Azure Database for PostgreSQL | DBaaS | Camada Propósito geral, 1 Gen 5 (45 vCore) x 730 Horas, Armazenamento de 10TB| 138,471





Como podemos ver, os valores são bem mais acessiveis em comparação a somente subirmos os servidores.

Seguindo o modelo (aks.jpg) podemos ver que temos alguns recursos, dentro do nosso cluster temos um namespace, que é a separação virtual das aplicações, logo após ele temos um Ingress, onde está referenciado o "mywebdomain.com", após a requisição chegar nele, será redirecionada para o Service que é responsavel por escolher um Pod (menor parte do cluster, um container), temos posteriormente um ReplicaSet, que é responsavel por criar e replicar os nossos Pods seguindo o padrão pŕe determinado, ao final temos um HPA que é o responsavel por gerenciar os recursos dos nossos pods e prepar os padrões para autoscaling. 

Os pods se conectam ao DB como serviço via connection string e os dois estão no mesmos resource group.

#### Exemplo de configuração do HPA

>`{{- if .Values.autoscaling.enabled }}
    apiVersion: autoscaling/v2beta1
    kind: HorizontalPodAutoscaler
        metadata:
        name: {{ .Release.Name }}
    spec:
        scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: {{ .Release.Name }}
        minReplicas: {{ .Values.autoscaling.minReplicas }}
        maxReplicas: {{ .Values.autoscaling.maxReplicas }}
    metrics:
        {{- if .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
        {{- end }}
    {{- if .Values.autoscaling.targetMemoryUtilizationPercentage }}
    - type: Resource
      resource:
        name: memory
        targetAverageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
    {{- end }}
{{- end }}`


### Como criar o cluster


Esse comando realiza a criação do cluster na aks, passando como parametro o resource group e o nome do meu cluster. 

>az aks create --resource-group Estudo --name Estudo-Prod --node-count 3 --enable-addons monitoring --generate-ssh-keys

Realiza a instalação do cli para utilização do aks
>az aks install-cli

Utilizado para se conectar com as credenciais geradas no cluster

>az aks get-credentials --resource-group Estudo --name Estudo-Prod

Cria um novo namespace (separação virtual entre deploys)
>kubectl create namespace Estudo-front-homo


#### Estudo

Para explificação, foi criado seguindo esse tutorial

>https://docs.microsoft.com/pt-br/azure/aks/kubernetes-walkthrough

