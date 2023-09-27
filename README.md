# ImersaoKeycloakChatgpt
integration of applications. This application. where the users logs via SSO, connect to ChatGPT

In the shoulders of Giants. All merit for , who is the provider of this code. I've just adapted the yaml file for worlinkg without grafana.
The objective of this tutorial is manually practicing kubernetes.


This "project" is based on class 2 of @fabricioveronez Devops Imerson, i did some adaptations, Im using virtual machine xubuntu 22, with 40GB disk and 3or 4 GB RAM.
The purpose is adapting with the main commands (kubectl, k3d, docker, etc).
Im using too Dbeaver, for testing the database, and postman (test api)

also show how to compile the docker images and uploading to my docker repository

take notes of the dependencies: k3d, kubect, docker account on, on the terminal, etc
take notes the dependencies, k3d, kubectl, account docker logged

install docker:
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
(session install-using-the-repository)


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

getting back to the process

cd .. (get bach to the directory idc-ms-chatgpt)

creating account in the openia and copy API, notes here:
create account
https://platform.openai.com/
personal
view api keys
create new secret key
copy the a key

or, also, check the usage, in the menu usage

and put the api at field value, in the deployment yaml of the chatservice, example:
name: OPENAI_API_KEY

value: "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"

kubectl apply -f k8s/deploy-chatservice2.yaml


-----------------

testing dbeaver

due we are running clusterIP, you have to use the port forward

kubectl port-forward po/chatservice-mysql-77d4ffc69d-4l2gl 3306:3306 #depois só mudar o nome do pod

so in dbeaver, press new database connection

mysql

database chat_service

password root

test connection - if you get an error of publick key, go to tab driverproperties and in the flag
allowpublicKeyRetrieval, set true

dont problem if we dont have tables for now, because later (when we execute the application) will be execute the operation of create tables, the migration for example

done the test, you can cancel  the port forward

-----------------

continuing

-----------------
test in postman

be logged in postman, otherwise you have to create it

create new collection

file new
it will show endpoint types for connection, so choose gRPC

at field search an API, select import a protofile

it will be in chatservice/proto
next
import as a api
using new api
go back to tab of request
field select a method - ChatStream
enter a url - grpc://localhost:50051

tab authorization / type API Key:
Key: Authorization
Value: 123456 
(it is what is defined in the aplication code, file chat-service-client.ts line 6)

tab messagem - POST (está api/chat.http):

{
    "chat_id": "a14fc851-7e95-49b5-a5b5-44479434becd",
    "user_id": "1",
    "user_message": "continue"
}


it will no work yet, because, if you type "kubectl get svc"
it will show is clusterIp, in other words, you have to expose manually, via port forward

kubectl port-forward svc/chatservice 50051:50051

if not work yet (ANOTE THE ERROR HERE) its becaus you have to kill the service pod, because it did not recognized the mysql
kubectl get po
kubectl delete po chatservice-7c6777bc56-662vq

the pode will be created again

do the invoke in the postman
and, after, you can cancel
and you can cancel this port forward also

-----------------

continuing

-----------------

uping keycloak

add at /etc/hosts
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

tab credentials

copy client secret
put in the file deploy-webapp at
        - name: KEYCLOAK_CLIENT_SECRET
          value: cGx2n9Dix3LKwYz2V0fKg58afHFhu8Dq

Users
Add User
Username: choose
email: choose

create
aba credentials
set password
123456
temporary OFF
save

-----------------

continuando

-----------------


kubectl apply -f k8s/deploy-webapp2.yaml

if it did not work at first time, do this command:
kubectl delete po webapp-8b7f7c6cb-ls8lg (exemplo, pois o nome do pod vai estar diferente)

and test in 
localhost:3000/


it was not working,
but after I tested at a cleaner enviroment, in the next day, without dirt, so it worked
