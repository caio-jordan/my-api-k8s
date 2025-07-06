🚀 Rodando uma API .NET no Kubernetes com Minikube
==================================================

Este guia é para iniciantes que desejam publicar uma API ASP.NET Core em um cluster local usando o Minikube e acessá-la a partir de outros dispositivos na mesma rede.

📦 Pré-requisitos
-----------------

*   Windows 10 ou 11
*   Docker instalado
*   [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
*   [Minikube](https://minikube.sigs.k8s.io/docs/start/)
*   Visual Studio Code (ou editor de sua preferência)
*   Projeto ASP.NET Core Web API

✅ Etapas resumidas
------------------

1.  Criar a imagem Docker da API
2.  Rodar o Minikube com porta externa liberada
3.  Aplicar o Deployment e Service no Kubernetes
4.  Liberar a porta no firewall do Windows
5.  Acessar via outro dispositivo na rede local

⚙️ Etapa 1 – Buildar imagem Docker DENTRO do Minikube
-----------------------------------------------------

Abra o PowerShell e ative o Docker dentro da VM do Minikube:

    minikube -p minikube docker-env | Invoke-Expression

Agora, crie a imagem da sua API:

    docker build -t my-api:k8s .

☸️ Etapa 2 – Iniciar o Minikube com a porta liberada
----------------------------------------------------

    minikube delete
    minikube start --driver=docker --ports=30080:30080

📁 Etapa 3 – Criar arquivos deployment.yaml e service.yaml
----------------------------------------------------------

### deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-api
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: my-api
      template:
        metadata:
          labels:
            app: my-api
        spec:
          containers:
          - name: my-api
            image: my-api:k8s
            imagePullPolicy: Never
            ports:
            - containerPort: 80

### service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: my-api-service
    spec:
      type: NodePort
      selector:
        app: my-api
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
          nodePort: 30080

🚀 Etapa 4 – Aplicar no Kubernetes
----------------------------------

    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml

Verifique se os pods estão Running:

    kubectl get pods

🔥 Etapa 5 – Liberar a porta 30080 no Firewall do Windows
---------------------------------------------------------

1.  Pressione Win + R, digite `wf.msc`, pressione Enter
2.  Vá em **Regras de Entrada**
3.  Clique em **Nova Regra...**
4.  Selecione **Porta**
5.  Escolha **TCP** e digite: `30080`
6.  Marque **Permitir a conexão**
7.  Marque os perfis **Privado** e **Público**
8.  Nome: **Minikube Porta 30080**
9.  Finalize

🌐 Etapa 6 – Acessar de outro dispositivo
-----------------------------------------

Verifique o IP local da sua máquina:

    ipconfig

Procure o adaptador da sua rede (ex: 192.168.15.13)

Em outro dispositivo conectado à mesma rede Wi-Fi (ex: celular), acesse no navegador:

    http://192.168.15.13:30080/swagger

🧪 Verificações úteis
---------------------

Verifique se a porta está escutando:

    netstat -a -n | findstr :30080

Esperado:

    TCP    0.0.0.0:30080      0.0.0.0:0      LISTENING

Verifique se o Minikube está com IP:

    minikube ip

Verifique a URL exposta:

    minikube service my-api-service --url

✅ Pronto!
---------

Sua API ASP.NET Core agora está rodando em Kubernetes via Minikube e acessível na rede local. Perfeito para testes com dispositivos reais, como celular ou outro PC.
