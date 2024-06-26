# Kubernetes/ k8s

## Criando cluster k8s com o 'kind'

Instale o kind em sua máquina: https://kind.sigs.k8s.io/docs/user/quick-start/

Instale o kubernetes em sua máquina: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

---

Para subirmos o nosso cluster com as configurações do `kind.yaml`, digitamos no terminal:
`kind create cluster --config=k8s/kind.yaml --name=fullcycle`.


Para checar se ele está rodando, utilize o comando: `kubectl cluster-info --context kind-fullcycle` ou
`docker ps`.

E para ver os nodes: `kubectl get nodes`.

---

## Subindo nossa aplicação go

Abra um terminal na raiz do projeto e digite: `docker build -t guilhermemmnn/hello-go .`

Agora rode: `docker run --rm -p 80:80 guilhermemmnn/hello-go`.

E se você abrir o seu navegador e digitar localhost, você verá **Hello Guilherme**.

Caso queira, você pode subir essa imagem no seu DockerHub: `docker push {usuario}/hello-go`

Agora, vamos criar nosso primeiro pod: `kubectl apply -f k8s/pod.yaml`, podemos
verificar os pods com o comando: `kubectl get pods`.

Podemos acessar nosso pod com o comando: `kubectl port-forward pod/goserver 8000:80`,
que vai direcionar a porta 8000 da nossa máquina para a porta 80 do pod selecionado, agora
abra o seu navegador e digite `localhost:8000`.

> Para deletar o pod, utilize o comando: `kubectl delete pod goserver`.

---

### O que é ReplicaSet ?

- **Obs: Caso você queira subir mais de um pod, podemos usar o ReplicaSet e para subir
ele, certifique-se que não tenha nenhum pod rodando e siga os passos abaixo:**

No terminal digite: `kubectl apply -f k8s/replicaset.yaml`.

Se você rodar o comando: `kubectl get pods`, você verá os 2 pods rodando.
> Caso você delete um pod, ele criará automaticamente outro, pois no arquivo
> `replicaset.yaml` foi configurado para rodar no mínimo 2 pods.

### Problema do ReplicaSet

Caso você gere e suba uma nova versão da sua imagem docker e já tenha ajustado a versão no `replicaset.yaml`
você deve deletar todos os pods que estão rodando a versão antiga para ele baixar a nova.

> Você pode ver qual imagem cada pod está rodando com o comando: `kubectl describe pod {nome_do_pod}`.
> 
> E deletá-lo com o comando: `kubectl delete pod {nome_do_pod}`.

#### Como solucionar o problema do ReplicaSet ?

Podemos utilizar o famoso **Deployment**, ele fica no topo da hierarquia sendo:

**Deployment -> ReplicaSet -> Pod**

> Antes de iniciar com o Deployment, certifique-se que não tenha o replicaset anterior rodando,
> para isso utilize o comando `kubectl get replicaset`.
>
> Caso tenha utilize o comando: `kubectl delete replicaset {nome_da_replicaset}`.
> 
> Logo após vamos criar o nosso **Deployment** com o comando: `kubectl apply -f k8s/deployment.yaml`.
> 
> E para verificarmos ele rodando: `kubectl get deployments`.

**E a vantagem é** que caso você **altere a versão da imagem** no `deployment.yaml`, basta você digitar o comando:
`kubectl apply -f k8s/deployment.yaml` e ele criará um novo replicaset com novos pods na nova versão.

---

### Rollout de versões antigas

Caso você queira voltar um deployment antigo, você tem a possibilidade.

Com o comando: `kubectl rollout history deployment {nome_do_deployment}`, conseguimos
ver as versões/revisions que já foram utilizadas no deployment.

E para fazer o rollout desse deployment para a versão passada,
utilize o comando: `kubectl rollout undo deployment {nome_do_deployment}`.

> Dica: Caso queira fazer o rollout de uma versão específica, você pode utilizar
> o comando: `kubectl rollout undo deployment {nome_do_deployment} --to-revision={numero_revision}`.

---

## Services

A service é a porta de entrada da nossa aplicação e serve para acessarmos nossos pods. <br>
Funciona como um _**load-balancer**_ em que ela faz o gerenciamento dos acessos e direciona para nossos pods

### Criando service com ClusterIP

Esse tipo de Service é utilizado para permitir a comunicação entre diferentes Pods e outros componentes do cluster.

O **IP** não é acessível de **_fora_** do cluster, você pode utilizar o **nome da service** para os pods conseguirem se comunicar e
o **ClusterIP** distribui o tráfego entre os Pods que estão associados ao Service (Load balancer).

No terminal, rode o comando: `kubectl apply -f k8s/service.yaml`.
Assim que criado, podemos checar a service com: `kubectl get svc`.

**Como acessar o cluster de fora?**<br>
Temos que fazer um direcionamento de porta para conseguirmos acessar o ip do cluster, para isso
utilize o comando: `kubectl port-forward svc/goserver-service 8000:80`.

Então podemos acessar via navegador com localhost:8080 ou com o comando: `curl http://localhost:8000`.

### Como direcionar a porta do nosso container ?

As vezes, criamos nossa service na porta `X` e nosso pod/aplicação está na porta `Y`, como é feito esse
direcionamento de portas?

**E a resposta é SIMPLES:**

- Utilizamos o `targetPort` em nosso `service.yaml`:

![target-port.png](readme_images%2Ftarget-port.png)

> **Ou seja**, o `port` é a porta da nossa service, e `targetPort` é a porta do nosso container que está a aplicação.

---

### Acessando a api do Kubernetes por Proxy

Ele vai gerar um proxy da nossa máquina para a api do Kubernetes e assim podemos ter acesso na api do Kubernetes.

Para isso, utilizamos o comando: `kubectl proxy --port=8080`. <br>
E podemos acessar pelo navegador: `localhost:8080`, você verá alguns endpoints que o kubernetes oferece,
e para acessar o endpoint da nossa service que foi criada acima, digite:
`localhost:8080/api/v1/namespaces/default/services/goserver-service`.

---

### Como funciona o tipo NodePort da service ?

**Serve para expor uma porta para fora do Cluster.**

Utilizado mais para demonstração, pois ele libera uma porta para todos os nodes da service, ou seja,
caso alguém tenha o IP de um dos nodes(máquinas) e a porta, ele conseguirá entrar dentro da service,
que redirecionará para uma `targetPort`.

- Abaixo podemos ver como é feito a criação de uma service com o tipo **NodePort**:

![img.png](readme_images/img.png)

> Podemos escolher uma nodePort entre **30000** à **32767**.

---

### Como funciona o tipo Load Balancer da service ?

**O tipo LoadBalancer cria um ip externo para acessarmos a service**

Muito utilizado para expor um serviço na internet com auxílio de um provedor de nuvem (AWS, Google Cloud).

Para criarmos, certifique-se de ter deletado a service do exemplo anterior, para isso use o comando:
`kubectl delete service goserver-service`.

E para criar: `kubectl apply -f k8s/serviceLoadBalancer.yaml`.

> Caso tivessemos rodando essa service em uma AWS, ele geraria um `EXTERNAL-IP`, que permitiria a conexão
> de qualquer pessoa que tivesse esse **IP**.
> ![img.png](readme_images/loadbalancer.png)

---

## O que são variáveis de ambiente

Com variáveis de ambiente conseguimos proteger dados sensíveis como senhas, ips, etc... no arquivo yaml e
puxar na nossa aplicação, como na imagem abaixo:

> **Aqui estamos puxando as variáveis de `NAME` e `AGE`.**
![img.png](readme_images/imagem-go.png)

> Aqui estamos **configurando as variáveis e seus values** no arquivo `deployment.yaml`, certifique-se de gerar
> uma imagem do seu **Dockerfile** e subir ele no _DockerHub_ e ajustar no arquivo do `deployment.yaml`.
![img.png](readme_images/deployment.png)

- Aplique essa configuração no Cluster com o comando: `kubectl apply -f k8s/deployment.yaml`.
- Faça o redirecionamento da porta do cluster com a sua máquina: `kubectl port-forward svc/goserver-service 8000:80`.
- Acesse pelo navegador: `localhost:8000` e veja a mágica acontecer!

## Como apartar as variáveis de ambiente?

Podemos criar um arquivo `yaml` com o **kind=ConfigMap**, igual ao arquivo `configmap-env.yaml` e
setar nossas variáveis lá dentro.

> E com um pequeno ajuste no nosso arquivo `deployment.yaml` podemos capturar a variável do nosso arquivo
`configmap-env.yaml`:
>
> ![img.png](readme_images/env.png)

**Agora iremos subir esse ajuste no Kubernetes, para isso:**

- Rode o comando: `kubectl apply -f k8s/configmap-env.yaml` e como mudamos o deployment precisamos subir
ele também: `kubectl apply -f k8s/deployment.yaml`

- Para testar, precisamos conectar uma porta da nossa máquina com a do Cluster: `kubectl port-forward svc/goserver-service 8000:80`

### E se tivermos muitas variáveis?

Caso tenha _**muitas variáveis de ambiente**_, nosso arquivo `deployment.yaml` ficaria **muito poluído** se
seguirmos no exemplo acima, para isso podemos utilizar o seguinte formato:
![img.png](readme_images/env-2.png)

> Com o envFrom, chamamos nosso arquivo `configmap-env.yaml` com o nome do seu metadata e assim o nosso
> `deployment.yaml` terá TODAS as variáveis do arquivo `configmap-env.yaml`. 
> 
> E para aplicar, basta rodar os comandos novamente: `kubectl apply -f k8s/configmap-env.yaml` e
> `kubectl apply -f k8s/deployment.yaml`.

### Agora vamos criar um volume para as variáveis

Suponha que sua aplicação precise ler um arquivo `.txt` e esse arquivo é **dinâmico**, para evitar toda vez que
esse arquivo tiver uma alteração e você precisar **buildar** a aplicação e **subir** no DockerHub, você pode injetar
o **ConfigMap** na sua aplicação:

- Para isso, criamos uma função na nossa aplicação **Go** que vai ler um arquivo `.txt`:
![img.png](readme_images/img_3.png)

- Em seguida, vamos criar nosso arquivo **.yaml** que terá o texto dinâmico:
![img_1.png](readme_images/img_1.png)

- Em seguida, configuramos nosso `deployment.yaml`:
> ![img_2.png](readme_images/img_2.png)
> Importante que a sua imagem do hello-go esteja com a nova função de ler arquivo.

- E por fim aplicamos essas configurações no nosso Cluster:
`kubectl apply -f k8s/configmap-family.yaml`. <br>
`kubectl apply -f k8s/deployment.yaml`. <br>
`kubectl port-forward svc/goserver-service 8000:80`.
- E assim, conseguimos ver os nomes pelo navegador: `localhost:8000/configmap`.

> Caso você queira acessar o pod para ver se os arquivos foram criados, você pode rodar o comando:
> `kubectl exec -it {nome_do_pod} -- bash`. <br>
> Ou caso queira ver o log de erro, utilize: `kubectl logs {nome_do_pod}`.

## Como guardar uma secret ?

Para guardar dados sensíveis como senhas e usuários, podemos utilizar o arquivo `secret.yaml` com o **kind=Secret**
e temos que criptografar os dados em `base64`, você pode utilizar o comando: `echo "guilherme" | base64` e caso esteja
utilizando o _Windows_, rode esse comando pelo terminal do **_git Bash_**

- Após configurar sua `secret.yaml`, vamos configurar o arquivo `deployment.yaml`:
![img.png](readme_images/img_4.png)


- E por fim seguimos o mesmo passo a passo: <br>
  `kubectl apply -f k8s/secret.yaml`. <br>
  `kubectl apply -f k8s/deployment.yaml`. <br>
  `kubectl port-forward svc/goserver-service 8000:80`.

- E assim, conseguimos ver o nosso usuário pelo navegador: `localhost:8000/secret`.

---

## Health check

### Criando endpoint Healthz

Vamos criar um endpoint na nossa aplicação Go para ver de tempos em tempos se ela está saudável.

Para isso, criamos uma função em nosso `server.go` bem simples para checar o tempo de início da nossa aplicação, e caso
a vida útil da nossa aplicação passe de 25 segundos, ele dará um erro **500** e ao acessar pelo `localhost:8000/healthz`,
veremos a duration da nossa aplicação.

> Lembre sempre de checar se o build da imagem do server.go está com a função `Healthz`.

### Utilizando o Liveness

O **liveness probe** é responsável por verificar se um contêiner está em execução corretamente. <br>
Ele é utilizado para **determinar** se um contêiner **precisa ser reiniciado**.

Vamos configurá-lo em nosso `deployment.yaml`:
![img.png](readme_images/img_5.png)

E podemos ir checando a reinicialização do nossos pods pelo comando: `kubectl get pods`.
> Como nossa aplicação está configurada para emitir um erro 500 depois de 25 segundos no ar,
> o nosso pod será reiniciado após esse período **+** o tempo que roda nosso teste no `deployment.yaml`.

### Utilizando o Readiness

Ele é usado para **garantir** que um contêiner esteja **totalmente inicializado** e pronto para lidar com solicitações
antes de ser incluído no balanceamento de carga.

- Para isso, configuramos nossa aplicação Go em que, caso tenha menos de 10 segundos de vida, ela emita o erro 500:
![img.png](readme_images/app-go-adjust.png)

- Logo após modificar a aplicação, geramos o build e o push da imagem.
- E trocamos a versão no nosso `deployment.yaml` e fazemos a inserção do `readiness`:
![img_1.png](readme_images/deployment-file.png)

> O **readiness** contém as mesmas funções do **liveness**, porém o liveness é para garantir que o container está saudável
> e o readiness para garantir que o container esteja pronto para requisições.

### Combinando o Liveness com o Readiness

Para utilizarmos ambos juntos, precisamos deixar que os testes do liveness e readiness estejam conciliados em tempos, pois o
readiness desvia o tráfego caso tenha erro e o liveness recria o pod caso a aplicação esteja com erro.

### Qual a melhor maneira de combinarmos o liveness e readiness?

Temos o famoso `startupProbe`, que é um tipo de sonda utilizada no Kubernetes para verificar o estado inicial
de um contêiner quando ele é iniciado.

Ou seja, só depois que ele verificar que o container está pronto, que ele vai liberar os testes do **liveness e readiness**.
![img.png](readme_images/startup-probe.png)

> Ele possui as mesmas configurações que o **liveness e readiness**, porém devemos nos atentar ao seguinte detalhe que é o
`failureThreshold`, devemos configurar pelo tempo máximo que nossa aplicação demora para subir, e com isso podemos remover
a função de `initialDelaySeconds` no **liveness e readiness**.
 
---

### Instalando o metrics-server

O metrics-server é um componente do Kubernetes que coleta métricas de recursos de um cluster, como CPU e memória,
e disponibiliza essas informações para que possam ser utilizadas por outros componentes, como o Horizontal Pod Autoscaler (HPA). 

Site: https://github.com/kubernetes-sigs/metrics-server

Porém, precisamos fazer uma alteração no deployment do metrics-server para rodar localmente, pois temos que remover o
protocolo TLS que vem por padrão, para isso:

- Caso você esteja utilizando esse projeto, ele já estará configurado na pasta do k8s com o nome de `metrics-server.yaml`.
- Ou baixe o `.yaml` acessando: https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

> Precisamos adicionar esse argumento `- --kubelet-insecure-tls`, para que ao rodar o deployment o protocolo TLS
> fique desabilitado. <br>
> **Obs:** Precisamos desabilitar somente para rodar localmente.
![img.png](readme_images/metrics-server-args.png)

- E seguimos a receita de bolo com os comandos: `kubectl apply -f metrics-server.yaml`.
- Podemos checar se a metrics-server está funcionando com o comando: `kubectl get apiservices`.
- Ela estará com o nome de `v1beta1.metrics.k8s.io`.

### Delimitando recursos nas aplicações

Precisamos setar recursos em nossas aplicações, para que o pod não consuma todos os recursos do Node e consequentemente
do Cluster.

**Para isso:**
Vamos em nosso arquivo `deployment.yaml` (Onde temos a imagem da nossa aplicação), e vamos setar nossos recursos:
![img.png](readme_images/delimitando-recursos.png)

> Para vermos isso na prática, aplique a configuração: `kubectl apply -f k8s/deployment.yaml`. <br>
> 
> E utilize o comando: 
`kubectl top pod {nome_pod}`.
![img.png](readme_images/command-top-pod.png)

---

### O que é HPA ?

O **Horizontal Pod Autoscaler (HPA)** é um recurso do Kubernetes que permite ajustar automaticamente o número de réplicas 
de um conjunto de pods em execução com base na utilização da CPU ou de outras métricas definidas.

Para isso, criamos o arquivo **hpa.yaml**, e configuramos para ele auto-escalar nosso Deployment que está em `deployment.yaml`.

Logo após, aplique as configurações com: `kubectl apply -f k8s/hpa.yaml`.

> Conseguimos ver nosso hpa com o comando: `kubectl get hpa`.


#### Teste de stress com Fortio

Você pode utilizar o Fortio para criar um teste de stress do seu Cluster, para isso, digite o comando abaixo:

- `kubectl run -it fortio --rm --image=fortio/fortio -- load -qps 800 -t 120s -c 70 "http://goserver-service/healthz"`

- Em outro terminal, veja a utilização do CPU pelo comando: `kubectl get hpa`.

> Caso você queira ver melhor o teste de stress, diminua os recursos no `deployment.yaml` e aumente o `-qps` e o `-c` do
> fortio e ele criará mais replicas.

---

## Volumes persistentes

Os volumes persistentes em **Kubernetes** são recursos que permitem que os dados armazenados em um contêiner persistam mesmo
após o contêiner ser **_encerrado ou reiniciado_**.

### O que é StorageClass ?

`StorageClass` é um recurso do Kubernetes que define diferentes classes de armazenamento para os volumes persistentes
usados por pods em um cluster.

- Para reservamos um espaço do armazenamento, utilizamos o kind `PersistentVolumeClaim` e passamos algumas configurações,
elas estão em `pvc.yaml`.
- Aplicamos com o comando: `kubectl apply -f k8s/pvc.yaml`.
- E podemos checar com: `kubectl get pvc`, porém vemos que ele está `Pending`, e para isso precisamos fazer o bind com o `storageclass`.

> Precisamos adicionar esse **_Claim_** em nosso `deployment.yaml`, para isso setamos as configurações em `volumes` e `volumeMounts`.
![img.png](readme_images/pvc-help.png)

- E por fim, aplicamos a configuração: `kubectl apply -f k8s/deployment.yaml`.
- E vemos que mudou em: `kubectl get pvc`.

**Importante:** Em nosso `deployment.yaml` configuramos o path no pod que será feito a _persistência dos dados_, então tudo
que estiver dentro do path `/go/pvc` **não será perdido** caso o pod seja deletado.

### O que é StatefulSet ?

StatefulSet é um recurso do Kubernetes que permite a criação de pods com **identidades únicas e estáveis**, garantindo que
cada pod tenha um **estado específico e persistente**.

Ele é utilizado para aplicações que necessitam de **armazenamento persistente** e de **identidades únicas**, como bancos
de dados e sistemas de mensageria.

> Utilizamos o arquivo `statefulset.yaml` para criar uma imagem do mysql em nossos pods em sequência, igual a uma pilha,
ou seja, cada replica desse pod será criado 1 por 1.

### O que é headless service ?

Um serviço **headless** no Kubernetes é um tipo de serviço que **não possui um cluster IP** associado a ele. Em vez disso,
ele retorna diretamente os **endereços IP dos pods** que fazem parte do serviço.

Isso é útil em cenários onde é necessário se comunicar diretamente com cada instância do pod, por exemplo,
em bancos de dados distribuídos.

#### Na prática:

**Foi utilizado o arquivo** `mysql-service-h.yaml`.

> Dentro do bash de um pod foi realizado um teste de ping para a service do mysql, e logo após, foi feito um ping direcionado
> para o nome de uma das replicas dos pods em específico.
![img.png](readme_images/headless-example.png)


### Criando volumes dinamicamente com statefulset

Caso você necessite criar um volume para cada replica no statefulset, você pode utilizar o seguinte código no arquivo
`statefulset.yaml`:

![img.png](readme_images/volumes-dinamicos.png)

E poderá ver os volumes criados em: `kubectl get pvc`.

**_Obs:_** Mesmo que você delete o pod, o volume não será perdido e assim que o pod for recriado, ele terá o volume atrelado.

---

## Ingress

O Ingress é um **recurso do Kubernetes** que permite gerenciar o acesso externo aos serviços dentro de um cluster.

Ele atua como um _**controlador de tráfego**_, direcionando as solicitações dos clientes para os serviços apropriados
com base em regras de roteamento definidas.

O Ingress **facilita** a configuração de `regras de roteamento`, `balanceamento de carga` e `terminação de SSL` para os
serviços, tornando mais fácil o acesso externo aos aplicativos implantados no cluster Kubernetes.

**Exemplo:**

Temos 10 **microserviços** em nosso cluster.

Para facilitar a comunicação entre eles fora do cluster, podemos utilizar o**ingress** que funcionará como um
`LoadBalancer` e fornecerá um **único IP** para acessar todos os microserviços, precisando apenas alterar o
`caminho do endpoint` e o `corpo da requisição`.

Podemos criar o `service.yaml` com o type `ClusterIP`, pois ele não precisará de um IP externo para ser acessado, isso
economizará nos custos da aplicação.

> O arquivo `ingress.yaml` é um exemplo básico de configuração do ingress, mas é recomendado ver o módulo `Ingress` 
> do curso de Kubernetes do FullCycle.

---

### O que é Cert-manager ?

O **Cert-manager** é uma ferramenta open-source que ajuda a automatizar a gestão de `certificados TLS` em ambientes Kubernetes.

Ele facilita a **emissão**, **renovação** e **revogação de certificados SSL/TLS**, permitindo que os desenvolvedores se concentrem
mais no desenvolvimento de aplicativos e menos na administração de certificados de segurança.

> É utilizado para sua aplicação WEB ter o protocolo `HTTPS`

Para fazer a instalação em seu provedor cloud, utilize: https://cert-manager.io/docs/installation/

Para gerar os certificados TLS, podemos utilizar o arquivo: `cluster-issuer.yaml` e ajustar o `ingress.yaml`.

> Para mais informações, acesse o módulo de **Cert-manager** no curso de Kubernetes na FullCycle.

---

## O que é Namespaces ?

**Namespace** é um recurso do Kubernetes que permite dividir o cluster em `espaços virtuais separados`,
onde cada recurso pode ser isolado e ter sua própria visão dos recursos disponíveis.

- Com o comando: `kubectl get ns`, podemos ver os namespaces do nosso cluster.


- Com o comando: `kubectl get pods -n={namespace}`, podemos ver todos os pods de um determinado namespace.


- Com o comando: `kubectl create ns {nome}`, podemos criar um novo namespace.


- Para aplicar um deploy em um namespace, utilize o comando: `kubectl apply -f {arquivo.yaml} -n={namespace}`.

#### Criando contextos por namespaces Dev e Prod

Em situações do cotidiano, podemos ter contextos de prod e dev, porém podemos acabar realizando o deployment no contexto
indevido, para isso, podemos criar um contexto em nosso cluster e garantir que os deploys serão feitos em um único contexto.

- Com o comando: `kubectl config view`, podemos checar o nome e usuário do cluster.
- Para criar um contexto dev, utilize: `kubectl config set-context dev --namespace=dev --cluster={nome_cluster} --user={usuario_cluster}`.
- Para aplicar o contexto dev, utilize: `kubectl config use-context dev`.
- Para checar se o contexto é dev, utilize: `kubectl config current-context`.

---

### Service Accounts é importante ?

Eles são usados para conceder permissões específicas aos Pods para acessar recursos dentro do cluster,
como **APIs**, **volumes** e outros serviços.

Não é recomendado usar a Service Account `default` em um ambiente de produção, pois ela possui permissões amplas
e genéricas dentro do cluster, o que pode representar um risco de segurança para sua aplicação.

### Criando Service Account 

Para criar uma **Service Account** com hierarquia de privilégios, você pode utilizar o arquivo em
`k8s/namespaces/security.yaml` e nele criaremos a Service Account e setaremos algumas roles de exemplo.

Aplique essa configuração com o comando: `kubectl apply -f k8s/namespaces/security.yaml`.

E aplicaremos essas roles no `k8s/namespaces/deployment.yaml` na linha 14: `serviceAccountName: server`.

**_Obs:_** Para ver o ApiGroup do recurso que você quer limitar, rode o comando: `kubectl api-resources`.