# Exemplo Kubernetes:

Nesse exemplo, criamos um pequeno deployment do nginx no kubernetes e o expomos via LoadBalancer. Vamos usar o microk8s como instalação do kubernetes, você pode instala-lo seguindo os passos [neste link](https://microk8s.io/).

Cria o namespace `development`

```SHELL
$> microk8s.kubectl create namespace development
namespace/development created
```

Retorna a lista de namespaces para mostrar que o novo namespace foi criado:

```SHELL
$> microk8s.kubectl get namespaces
NAME              STATUS   AGE
kube-system       Active   64m
kube-public       Active   64m
kube-node-lease   Active   64m
default           Active   64m
development       Active   35s
```

Cria um deployment, usando o arquivo `deployment.yaml`.

```SHELL
$> microk8s.kubectl -n development apply -f deployment.yaml
deployment.apps/exemplo created
```

Retorna o deployment acompanhando sua evolução.

```SHELL
$> microk8s.kubectl -n development get deployment exemplo -o wide -w
NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
exemplo   2/4     4            2           19s   exemplo      nginx    role=exemplo-backend
exemplo   3/4     4            3           21s   exemplo      nginx    role=exemplo-backend
exemplo   4/4     4            4           22s   exemplo      nginx    role=exemplo-backend
```

Cria um serviço interno que aponta para o deployment.

```SHELL
$> microk8s.kubectl -n development apply -f service.yaml
service/exemplo-service created
```

Pegando o serviço interno, vemos que ele não possui IP externo, e só pode ser acesso de dentro do clusterm seja pelo IP, seja pelo domínio `exemplo-service.development.svc`.

```SHELL
$> microk8s.kubectl -n development get service/exemplo-service
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
exemplo-service   ClusterIP   10.152.183.182   <none>        80/TCP    38s
```

Criamos um serviço com IP externo, do tipo LoadBalancer, que expõe o serviço pra fora do cluster?

```SHELL
$> microk8s.kubectl -n development apply -f external-service.yaml
service/exemplo-external-service created
```

Pegamos o IP externo, e podemos ver a página carregada ao acessá-lo:

```SHELL
$> microk8s.kubectl -n development get svc/exemplo-external-service -o wide -w
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE     SELECTOR
exemplo-external-service   LoadBalancer   10.152.183.73   10.0.0.10     80:31352/TCP   3m54s   role=exemplo-backend
```