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