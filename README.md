# guacamole


#Etapa 1: extrair as imagens necessárias

#Primeiro, vamos extrair as imagens necessárias do Docker Hub:

sudo docker pull docker.io/guacamole/guacd
sudo docker pull docker.io/guacamole/guacamole

#Verifique se as imagens foram baixadas:

sudo docker images

#Etapa 2: crie uma rede

#Crie uma rede para nossos contêineres de Guacamole:

sudo docker network create guacamole-network

#Etapa 3: configurar o banco de dados

#Execute um contêiner MySQL para Guacamole na rede que acabamos de criar:
 sudo docker run --name guacamoledb --network guacamole-network -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=guacdb -d mysql/mysql-server

#Etapa 4: inicializar o banco de dados

#Crie um diretório para Guacamole e defina as permissões apropriadas e verifique se as permissões estão corretas

sudo mkdir -p /opt/guacamole/mysql
sudo chown -R $USER:$USER /opt/guacamole
cd /opt/guacamole
ls mysql

#Gere o SQL para inicializar o banco de dados:

sudo docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > /opt/guacamole/mysql/01-initdb.sql

#Copie o SQL de inicialização para o contêiner do banco de dados:

sudo docker cp /opt/guacamole/mysql/01-initdb.sql guacamoledb:/docker-entrypoint-initdb.d

#Etapa 5: configurar o banco de dados

#Acesse o contêiner do banco de dados:

sudo docker exec -it guacamoledb bash

#Dentro do contêiner, execute os seguintes comandos:

cd /docker-entrypoint-initdb.d/
mysql -u root -p

#Digite a senha que você definiu anteriormente (neste caso, ‘password’). Então execute:

use guacdb;
source 01-initdb.sql;

#Em seguida, verifique todas as tabelas com o seguinte comando:

show tables;

#Em seguida, crie um usuário administrador e defina uma senha com o seguinte comando:

create user guacadmin@'%' identified by 'password';
grant SELECT,UPDATE,INSERT,DELETE on guacdb.* to guacadmin@'%';

#Em seguida, libere os privilégios e saia do shell do MySQL com o seguinte comando:

flush privileges;
exit;

#Exit the container:

exit

#Etapa 6: execute o servidor Guacamole

#Inicie o servidor Guacamole, certifique-se de incluir a rede:

sudo docker run --name guacamole-server --network guacamole-network -d guacamole/guacd

#Verifique os registros para ver se o guacamole apresentou algum problema:
sudo docker logs --tail 10 guacamole-server

#Etapa 7: execute o cliente Guacamole

#Inicie o cliente Guacamole dentro da mesma rede, estamos rodando na porta 8090, você pode rodar em qualquer porta que desejar se o sistema permitir:

sudo docker run --name guacamole-client --network guacamole-network -e MYSQL_DATABASE=guacdb -e MYSQL_USER=guacadmin -e MYSQL_PASSWORD=password -e GUACD_HOSTNAME=guacamole-server -e MYSQL_HOSTNAME=guacamoledb -d -p 8090:8080 guacamole/guacamole

#Verifique os logs (substitua ‘xxx’ pelo ID real do contêiner):

sudo docker logs xxx

#Etapa 8: verifique a instalação

#Verifique se a porta está aberta:

ss -altnp | grep :8090

#Etapa 9: acesse o Guacamole

#Agora você pode acessar o Guacamole navegando para:

http://localhost:8090/guacamole/#

#Use as credenciais padrão:
#Usuário: guacadmin
#Senha: guacadmin
