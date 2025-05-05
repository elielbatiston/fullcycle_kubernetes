docker build -t batistondeoliveira/hello-go:v9.6 .
docker push batistondeoliveira/hello-go:v9.6
=========================
Comandos importantes
=========================
1. Create a cluster
* kind create cluster => Criar um cluster
kubectl cluster-info --context kind-kind => inicio uma comunicação com o kubernetes no contexto kind-kind

* kind create cluster --config=k8s/kind.yaml --name=fullcyle => cria um cluster utilizando a configuração e com o nome fullcycle

2. Exibir os nodes / clusters / hpa
* kubectl get nodes => exibe todos os nodes disponiveis

* kind get clusters => exibe todos os clusters criados com o kind

* kubectl get hpa => Exibe as informações de auto scale.

* kubectl get storageclass => Verifica o volume

* kubectl get pvc => Verifica o Persistent Volume Claim

* kubectl get ns => Exibe todos os namespaces que tenho

* kubectl get po -n cert-manager (-n de namespace) => Exibe todos os namespaces com o nome cert-manager

* kubectl get certificates => Exibe os certificados

3. Excluir um cluster/pod/replicaset
* kind delete clusters <nome_cluster> => deleta o cluster, pegar o nome com o comando kind get
* kubectl delete pod <nom_pod> => delete o pod
* kubectl delete replicaset <nome_replicaset> => delete a replicaset

4. Verificar as configurações
* cd ~/.kube/ => entrar na pasta do kubernetes onde tem os arquivos de configuracao

5. Listar clusters disponíveis
* kubectl config get-clusters
* kubectl config view => ver as configurações do meu cluster

6. Mudar o contexto
* kubectl config get-contexts => Ver os contextos
* kubectl config current-context => Ver qual o contexto atual
* Criar novo contexto, vide seção 36
* kubectl config use-context <nome_cluster>

7. Extensões VSCode
* Kubernetes

8. Aplicar um arquivo de configuração
* kubectl apply -f <nome_arquivo> 
exemplo: kubectl apply -f k8s/pod.yaml

* kubectl apply -f <diretorio>  para aplicar todos
exemplo: kubectl apply -f k8s/

* kubectl apply -f <nome_arquivo> -n=<nome_namespace> => Aplica em um namespace específico. Vide seção 35
exemplo: kubectl apply -f k8s/deployment.yaml -n=dev

9. Ver os Pods/Replicasets/Service rodando
* kubectl get pods
* kubectl get replicaset
* kubectl get service ou * kubectl get svc

10. Acessar a aplicação diretamente do Pod
* kubectl port-forward pod/goserver 8081:8080

11. Exibe informações do Pod
* kubectl describe pod <nome_pod>

12. Hierarquia
Deployment > ReplicaSet > Pod

13. Ver historico de versoes
* kubectl rollout history <tipo_objeto> <nome_objeto>
exemplo: kubectl rollout history deployment goserver

14. Voltar ultima versao
* kubectl rollout undo <tipo_objeto> <nome_objeto>
exemplo: kubectl rollout undo deployment goserver

15. Voltar para uma versão específica
* kubectl rollout undo <tipo_objeto> <nome_objeto> --to-revision=<id_revision>
exemplo: kubectl rollout undo deployment goserver --to-revision=1

16. Tipos de Services
* ClusterIP
* NodePort => Antigo - Facilita para acessar o cluster de fora da rede. Libera a porta em todos os meus nodes. Sabendo o ip do node, eu consigo acessá-lo pq a porta será igual para todos
* LoadBalancer => gera um ip externo para acessar a sua aplicação de fora. É usado quando você utiliza um cluster gerenciado ou um cluster que está conectado diretamente a um provedor de nuvem. Quando você quer expor o seu serviço para a internet você usará esse tipo de serviço

17. Proxy (Gera um proxy da minha maquina com o cluster Kubernetes)
* kubectl proxy --port=X
exemplo: kubectl proxy --port=8080

18. Entrar dentro do Pod
* kubectl exec -it <nome_pod> -- bash
exemplo: kubectl exec -it goserver-dc76f547f-9vqw9 -- bash

19. Aplicar uma configuração e ficar assistindo o pods
* kubectl apply -f k8s/deployment.yaml && watch -n1 kubectl get pod

20. Liveness 
Verifica se nossa aplicação está saudavel

21. Readness
Verifica quando a aplicação está pronta para receber trafego, se não tiver pronta, ele desvia o trafego

22. StartupProbe
Funciona como o Readness mas somente na inicialização. O Readness e Liveness só irão funcionar depois desse

23. Instalar o metrics server
http://github.com/kubernetes-sigs/metrics-server

1- Baixar o https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml em uma pasta de preferencia.

Exemplo: pasta k8s/ executar wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

2- Embora não precise, mas para não confundir, renomeie o arquivo components.yaml baixado para metrics-server.yaml

3- Adicionar um argumento para poder ficar inseguro 
Ver issue: http://github.com/kubernetes-sigs/metrics-server/issues/525
Está no Comment: https://github.com/kubernetes-sigs/metrics-server/issues/525#issuecomment-746809546
Argumento a ser adicionado: - --kubelet-insecure-tls

3- kubectl apply -f metrics-server.yaml

4- Verificar se tudo está funcionando 
Execute o comando: kubectl get apiservices
Note que vai ter o novo service kube-system/metrics-server conforme abaixo

NAME                                   SERVICE                      AVAILABLE   AGE
v1beta1.metrics.k8s.io                 kube-system/metrics-server   True        71s

Como podemos observar o available está true então está tudo certo.

24. Recursos do sistema (resources)
* requests: Qual o mínimo para ele funcionar. É definido em cpu e memory

Unidade de medida para falar de cpu chamada vCPU. Quanto mais máquinas temos com vCPU mais a máquina é potente em poder de processamento.

vCPU tem 1000m (milicores) Normalmente chamado de share cpu ou seja, quanto consigo usar dessa CPU. 
Se eu defino que minha aplicação utiliza 500m então estou dizendo que de uma cpu inteira eu vou utilizar apenas metade
Se eu colocar 0.5, também significa que estou dizendo que estou utilizando a metade da minha cpu

OBS: Não tem como chegar a esses números sem testar. Vai ter que fazer benchmarks, teste de stress, etc pra ver como a aplicação se comporta para chegar no melhor número possível.

Lembrando que requests é o mínimo de recurso necessário para nossa aplicação funcionar, então, estamos reservando a quantidade de memória e cpu somente pra nossa aplicação.

OBS: Se colocar uma quantidade exagerada onde não tem recurso suficiente então ele vai ficar PENDENTE até ter recursos para ele provisionar

* limits: Até onde esse POD pode utilizar de recursos no nosso cluster.

** CPU **
IMPORTANTE: Evite que a soma de todos os limites ultrapasse a quantidade disponivel de recursos no seu cluster.

Porem, você pode pensar que nem sempre q todo o limite vai estar utilizado ao mesmo tempo então vou ter que utilizar mais máquina e a minha máquina vai ficar ociosa porque eu não estou sempre no limite.

Então nesse caso, você vai apostar mais ou menos igual ao overbooking. Você coloca a venda mais passagens do que cabe no voo porque você acredita que nem todo mundo vai comparecer e se comparecer você faz algum sorteio, faz alguma coisa para alguém desistir pra você conseguir resolver esse tipo de problema.

Então, você pode pegar uma média e ver que em momentos de pico ele está utilizando um máximo X então, você faz um calculo para que a soma dos limits chegue a esse valor X pra você ter algo seguro e ter um equilibrio em consumo de máquina, maquina que vc vai deixar ociosa e também a utilização desse camarada.

** Memory **
Já não consegue fazer o mesmo com a CPU pois memória é algo mais fixo.
Então você não consegue deixar passar do limite pq CPU vc consegue estourar um pouquinho pra mais ou pra menos mas ele consegue resolver a situação.
Já na memoria não consegue resolver a situação tão facilmente porque é algo muito limitado ou seja você não consegue passar um pouquinho mais em relação a quantidade total de memória.

IMPORTANTE: Se tivermos várias replicas, temos que multiplicar o número de replicas pelo limit de cpu e memória para sabermos de fato quanto nossa aplicação está consumindo.

25. Ver o consumo de um POD
* kubectl top pod <nome_pod>

exemplo: kubectl top pod goserver-86755b87df-9sdt4

26. Ferramenta para teste de stress
* Fortio (http://github.com/fortio/fortio)
* k6

Versão 1.21 do kubectl
-----------------------
kubectl run -it --generator=run-pod/v1 fortio --rm --image=fortio/fortio -- load -qps 800 -t 120s -c 70 "http://goserver-service/healthz"

Versão nova
-----------------------
kubectl run -it fortio --rm --image=fortio/fortio -- load -qps 800 -t 120s -c 70 "http://goserver-service/healthz"
800 requisições por segundo
120s tempo
70 conexoes simultaneas

OBS: Vide tabela no github do fortio.

Assistir o hpa: watch kubectl get hpa

27. Volumes
Semelhante ao docker
a-) Volumes persistentes
 Forma Static:
    Pool de storage: Eu crio um pool de storage de 1TB
    Claim: 50Gi ou seja de 1TB que eu tenho, disponibilizo 50Gi para um determinado volume
 
 StorageClass: Especificação q faz que eu tenha um drive para que dinamicamente eu possa provisionar volumes/espaço em disco para uma determinada aplicação

 Claim -> StorageClass -> Disponibilizar o espaço que eu preciso. Por exemplo: BlockStorage

28. Headless service
Serviço criado sem IP que é um apontamento de DNS então significa que qdo eu crio um headless service mandando mysql-master, ele vai apontar para o meu POD master, se eu colocar mysql-slave-1, ele vai apontar para o slave 1

Feito todas as configurações, eu posso fazer os seguintes comandos.

* kubectl get po => ver todos os pods
* kubectl exec -it <nome_pod> -- bash => entrar em um pod do meu sistema
* ping mysql-h => Executar um ping no meu serviço mysql-h e ele vai mostrar o ip e de qual pod está pingando.
* ping mysql-0.mysql-h => Pingar direto no pod mysql-0 do serviço mysql-h

29. StatefulSet - Volumes dinamicos
Cria volumes dinamicamente para cada replica.

30. Ingress (Utilizado na nuvem)
Ele trabalha parecido com o nginx
Ele é o ponto unico de entrada nas aplicações
Quando chega /admin, ele manda pro micro serviço de admin
Quando chega /login, ele manda pro micro serviço de login e assim por diante

31. Ingress nginx
ingress-nginx helm chart (http://kubernetes.github.io/ingress-nginx/deploy)

Com isso não precisa do service como loadbalance e ele pode ser como clusterIP então vc economiza uma grana
 
32. Certificados
Como instalar (Cert-Manager):
http://cert-manager.io/docs/installation/kubernetes

33. Namespace
Separação virtual lógica que fazemos no nosso cluster k8s pra onde fazemos instalação de cada coisa
Em todo nosso exemplo, estávamos utilizando o namespace default.

Util para separar o projeto em Prod e Stage pois você pode dizer que o namespace Prod tem mais recurso que o de Stage, embora seria mais interessante ter cluster para cada um, mas com baixo orçamento é válido.

Separa a parte de segurança, acesso, etc.

34. Criar um namespace
* kubectl create ns <nome_namespace>
exemplo: kubectl create ns dev

35. Executar yaml no namespace criado
* kubectl apply -f <nome_arquivo> -n=<nome_namespace>
exemplo: kubectl apply -f k8s/deployment.yaml -n=dev

ou posso colocar direto no arquivo deployment.yaml com a tag namespace

36. Contextos por namespace
Util para separar os contextos e evitar que você execute um arquivo yaml em um namespace errado. (Precisa sempre trocar de contexto)

cat ~/.kube/config => ver as minhas configurações
* kubectl config view => ver as configurações do meu cluster
* kubectl config set-context dev --namespace=dev --cluster=kind-fullcycle --user=kind-fullcycle=> cria um contexto chamado dev (Atencao: kind-fullcycle é o nome do cluster e user que está configurado no kubectl config view)
* kubectl config set-context prod --namespace=prod --cluster=kind-fullcycle --user=kind-fullcycle=> cria um contexto chamado prod (Atencao: kind-fullcycle é o nome do cluster e user que está configurado no kubectl config view)

37. Service Account
Quando subimos uma aplicação, de alguma forma ela precisa de uma permissão pra rodar dentro do k8s então, toda vez que eu subo um pod, que subo um deployment ou qq coisa desse tipo, esse cara ta sendo executado no k8s. Toda vez q entro no POD de alguma forma ele precisa ter credenciais para acessar.

* kubectl get serviceaccounts

A service accounts default permite fazer tudo. Então se alguém acessar sua aplicação q usa a service account padrão, ela vai conseguir deletar seu pod, vai conseguir zoar sua aplicação inteira.

38. Criando um service account
Ver arquivo security.yaml

* kubectl api-resources => Ver todos os recursos (Ver o api group)

39. ClusterRole
Se eu quiser aplicar as roles para o cluster como um todo, então eu utilizo o ClusterRole
Vide alteração do security.yaml

40. Ver eventos
* kubectl get events -n <nome_pod>

41. Ver logs
* kubectl logs -f <nome-pod> -n <nome-namespace>




