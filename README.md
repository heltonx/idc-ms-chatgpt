# ImersaoKeycloakChatgpt
integration of applications. This application. where the users logs via SSO, connect to ChatGPT

In the shoulders of Giants. All merit for , who is the provider of this code. I've just adapted the yaml file for worlinkg without grafana.
The objective of this tutorial is manually practicing kubernetes.


This "project" is based on class 2 of @fabricioveronez Devops Imerson, i did some adaptations, Im using virtual machine xubuntu 22, with 40GB disk and 3or 4 GB RAM.
The purpose is adapting with the main commands (kubectl, k3d, docker, etc).
Im using too Dbeaver, for testing the database, and postman (test api)


also show how to compile the docker images and uploading to my docker repository


take notes of the dependencies: k3d, kubect, docker account on, on the terminal, etc
anotar as dependencias, k3d, kubectl, conta docker logada


-----


type: df -h

if the disk spaces aren't that empty, 
type: 
docker system prune -af --volumes

but use that with wisdom, because its a powerfull command

-----



create cluster:

k3d cluster create meucluster --servers 1 --agents 1 -p "3000:30000@loadbalancer" -p "8080:30001@loadbalancer"

kubectl get nodes

watch kubectl get po



------

doing the build:

cd chatservice
sudo docker built -t heltonx/imersao-chatservice:v1 .

check if the image was created
docker image ls

upload the image (not necessary, if it is already downloaded, and referenced in the yaml, but its to learn)
docker logout
docker login

docker push heltonx/imersao-chatservice:v1 

uploading with the latest also, because its a best practice upload with the latest also

docker tag heltonx/imersao-chatservice:v1 heltonx/imersao-chatservice:latest

docker push heltonx/imersao-chatservice:latest

check in the docker web account your repositories
https://hub.docker.com

-------

voltando ao processo

cd .. (voltar para a pasta idc-ms-chatgpt)


processo de criar conta na openia e puxar a APIKey anotar aqui
criar conta no openia
https://platform.openai.com/
personal
view api keys
create new secret key
copiar a key

ou também ver o uso, no menu usage


e colocar a api no campo value do deployment do arquivo do chatservice, exemplo:
name: OPENAI_API_KEY
value: "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"


kubectl apply -f k8s/deploy-chatservice2.yaml


-----------------
teste do dbeaver

como tá rodando clusterIP tem que utilizar o port forward
kubectl port-forward po/chatservice-mysql-77d4ffc69d-4l2gl 3306:3306 #depois só mudar o nome do pod

dai no dbeaver dar um new database connection
mysql
database chat_service
senha root
test connection - se der erro de publick key ir na aba driverproperties e na flag
allowpublicKeyRetrieval setar como true

não tem problema nao ter tabela agora, pois só depois quando executar a aplicação vai ser feita
a operação de criar as tabelas, a migration por exemplo

feito o teste, pode cancelar o port forward
-----------------

continuando

-----------------
teste no postman

estar logado no postman, senao tem conta criar


create new collection

file new
vai mostrar os tipos de endpoint pra conectar, escolher gRPC

no campo search an API, selecionar importar um protofile
vai estar em chatservice/proto
next
import as a api
using new api
volta na aba da request
campo select a method - ChatStream
enter a url - grpc://localhost:50051

aba authorization / type API Key:
Key: Authorization
Value: 123456 (que é o ques está definido no código da aplicação arwuivo chat-service-client.ts linha 6)


aba messagem - POST (está api/chat.http):

{
    "chat_id": "a14fc851-7e95-49b5-a5b5-44479434becd",
    "user_id": "1",
    "user_message": "continue"
}

nao vai funcionar ainda, pois se der um kubectl get svc
vai mostrar que é clusterip, ou seja tem que expor manualmente, via port forward

kubectl port-forward svc/chatservice 50051:50051

se ainda nao for (ANOTAR ERRO AQUI) é porque tem que matar o pod do serviço, pois nao reconheceu o mysql
kubectl get po
kubectl delete po chatservice-7c6777bc56-662vq

o pod sera criado novamente

dar um invoke no postman
depois cancelar
e pode cancelar esse port forward também

-----------------

continuando

-----------------

subindo keycloak

adicionar no /etc/hosts
127.0.0.1 keycloak

kubectl apply -f k8s/deploy-keycloak2.yaml 

http://localhost:8080/

administration console

admin
admin

clients

create client

client id: gpt-webapp
name: nao importa
next
Client authentication ON
next
Valid redirect URIs: *
Web origins: *

Next
Save

aba credentials

copiar client secret
colocar no arquivo deploy-webapp em
        - name: KEYCLOAK_CLIENT_SECRET
          value: cGx2n9Dix3LKwYz2V0fKg58afHFhu8Dq

Users
Add User
Username: escolher
email: escolher

create
aba credentials
set password
123456
temporary OFF
salvar

-----------------

continuando

-----------------


kubectl apply -f k8s/deploy-webapp2.yaml

se nao der de primeira dar um 
kubectl delete po webapp-8b7f7c6cb-ls8lg (exemplo, pois o nome do pod vai estar diferente)

e testar em 
localhost:3000/

no ultimo teste abriu a janela da aplicação mas quando dava enter
depois testei novmente em um ambiente mais "limpo", no outro dia, sem sujeira de testes do mesmo dia, ai deu certo
