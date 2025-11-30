# Arquitetura-Multi-Tier
Este projeto foi criado com o objetivo de replicar um ambiente de infraestrutura real de pequenas e médias empresas, utilizando serviços como Route 53, ACM, API Gateway, ECS (Fargate), EC2 e RDS (MySQL). A arquitetura implementada segue boas práticas de segurança e de design recomendadas pela AWS, fornecendo uma base sólida, escalável e confiável para aplicações modernas.

## **Diagrama do Projeto**

   ![Architecture.png](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Architecture.png)

## **Principais Serviços utilizados**

1. **Route 53**
 - Serviço nativo de DNS da AWS
 - Utilizado para registradar domínios de sites e mapeamento dos DNS Records
 - Possui funcionalidade de roteamento. Diferente de dos serviços de DNS comuns
 - Roteamento simples, ponderado, failover, Geolocalização...

2. **ECS (Elastic Container Service)**
 - Serviço nativo de orquestração de containers da AWS
 - Utilizado para criar microserviços de containers em cima de uma instância EC2 ou Fargate (Utilizado em cenários aonde o cliente não quer gerenciar o container)
 - Possui funções como: TASK Definition, Task, Service.

3.  **RDS (Relational Database Service)**
 - Serviço de DataBases gerenciado pela AWS
 - Possui variades de Bancos de Dados que podem ser criados
 - Facilita na criação de Bancos de Dados e automatiza backups, manutenção, implementação de path
    
4. **API Gateway**
 - Serviço nativo da AWS para criar API´s
 - Utilizado para criar comunicação entre serviços
 - Permite criar API´s HTTP, Rest, WebSockets...
 - Pode ser utilizado para larga escala de acessos e conectar com diversos serviços
 
5. **ACM (AWS Certification Manager)**
 - Serviço para criar certificados de criptografia nativo da AWS
 - Cria, armazena e renova certificados públicos e privados para suas aplicações 
 - Gerenciamento de certificados de SSL/TLS solicitados na AWS
---

# **Criação do Projeto**

1. *Criando Uma Zona de Hospedagem para o Domínio*
 - Vá no Route 53 e Criei uma Zona de Hospedagem para seu domínio

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Captura%20de%20tela%202025-11-12%20063750.png)

 - Insirá o seu domínio e escolha qual Zona de Hospedagem você irá utilizar

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Captura%20de%20tela%202025-11-12%20063836.png)

  > **⚠️ Atenção:** Caso você queira ter acesso do seu domínio via internet, utilize uma zona pública, pois assim será possível ser feito este acesso. Se não, utilize uma zona privada para que o roteamento seja feito somente para sua rede privada.

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Captura%20de%20tela%202025-11-12%20064137.png)
---

2. *Criação do cluster ECS*
 - O cluster é o agrupamento de tasks ou serviço na camada lógica
 - Possibilita a escolha da infraestrutura que será utilizada para os containers 
 - Neste projeto implementei o fargate pois não quero gerenciar minhas tasks

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Cluster_ecs.png)

2.1 *Task Definition*
 - Task Definition é o recurso do ECS onde iremos configurar nosso container
 - Escolher o tipo de inicialização dos containers, sistema operacional, Tamanho da cpu e memória que serão utilizados
 - Podemos criar e associar funções a tasks caso seja necessario se comunicar com outros recursos

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Config_task_1.png)

2.2 *Detalhes do container* 
 - Utilizei httpd:latest como imagem docker
 - Uma imagem simples do Apache, mas que permiti visualizar a execução do container

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Config_task_2.png)

2.3 *Criando o Service*
 - o Service é utilizado como gerenciador de Task´s
 - As task´s são criar, removidas e gerenciadas de maneira automatica pelo service. Garantindo que o usuário não se preocupe em criar de maneira manual e gerencia-lá.

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Config_service_1.png)

   > **⚠️ Atenção:** As configurações do Service devem correspoder as mesmas configuradas anteriormente para que o cluster funcione corretamente!

2.4 *Configurando Replicação e Rede no service*
 - Nesta sessão temos questões importante sobre a quantidade de replicação da Task
 - O meu projeto preferi criar 1 replica para que eu consiga testar meu web server
 - Caso seu projeto seja maior, eu indico que você crie mais replicas para que o tráfego seja distribuido de forma equilibrada entre eles, garantindo alta disponibilidade em seu projeto.
 - É essencial que o tráfego recebido não sobrecarrege somente um container para que seu container a saúde continue integra.

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Config_service_2.png)

2.5 *Testando acesso ao website*
 - Testando o site via endereço ip da Task
 - o SG configurado permite acesso HTTP e HTTPs
 - Neste cenário meu acesso estava sendo feito via porta 80 pois ainda não havia criado o certificado ACM

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/testing_acess_ip_public_ecs.png)]

3. *Criação do API Rest*
 - Utilizar o API para direcionar o tráfego ao Webserver
 - Possibilita integração ao WAF, diferente do API HTTP que não é possível
 - Além de que o API Rest disponibiliza diversas funcionalidades para sua API:Cache, integração com Waf...

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_api_rest.png)

3.1 *Criação do Método API*
 - criei o método Get, permitindo somente acesso ao recurso
 - Implementei o endpoint aonde será direcionado o tráfego (Endereço ip da interface de rede do container)
 - O tipo de integração utilizei o HTTP

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_method_api.png)

3.2 *Alterando método de resposta da API*
 - Está alteração foi feita para que o usuário consiga visualizar o web site de maneira amigável
 - No momento em que seu usuário acessa o site, ele consegue visualizar seu website, caso não seja feita está alteração, ele irá receber informações em JSON

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Config_metodo_resposta.png)

3.3. *Alterando resposta integrada da API*
 - Está alteração também deve ser fewita para que a alteração anterior funcione
 - Neste caso eu irei remover o modelo de mapeamento que está na imagem abaixo

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Config_resposta_integrada.png)

3.4 *Deploy do stage da API*
 - Faça o deploy da api após todas as configurações anteriores da API
 - Faremos o deploy para testar o endpoint da API

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Deploy_Stage_api.png)

3.5 *Testando o Deploy*
 - o Método de Get eu implementei diretamente no / Da API
 - Com esse método meus usuário não precisam colocar a URL(domain) /Stage da API
 - Eles só precisam colocar HTTP://jeffcloudlab.com e terão acesso ao site 

3.6 *Criando nome de domínio personalizado para API* 
 - Criei um nome de domínio personalidado para ser associado ao DNS Records tipo A do meu domínio
 - Este nome irá fazer o mapeamento para minha API já criada
 - E direcionar os usuário até o Endpoint

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_domain_Api_1.png)

 - Criado como público porque será acessado pela internet...

3.7 *Criando ACM para associar ao nome de domínio personalizado*
 - Ele precisa ser criado para associar ao domínio
 - No momento em que os usuário acessarem o site, ele estará sendo trafegado via HTTPS

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_acm.png)

3.8 *Registrando o certificado ACM no domínio Jeffcloudlab.com
 - O domínio após o registro só será acessado via HTTPS
 - Agora o Client-server terá acesso seguro e privado

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_registre_domain.png)

3.9 *Concluindo a criação do domínio personalizado*
 - Podemos finalizar a criação do domínio personalizado
 - Caso você tenha criado o certificado, não será possível que continue a criação

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_domain_api_2.png)

4. *Registrando o Domínio no Route 53
 - Precisamos criar o domínio no Route 53
 - Ele será essencial para quando o usuário inserir o domínio no browser, o Route 53 faça o mapeamento do domínio para o DNS Record Tipo A
 - Ele retorna o endpoint do API para que o cliente consiga acessar o Website

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Associate_domain_api_route53_2.png)

> **⚠️ Atenção:** O Endpoitn associado deve ser o nome de domínio personalizado que foi criado no API. E ele deve aparecer no momentoem que você clicar nele! Caso ele não apareça, recarregue a página ou refaça a configuração.

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Associate_domain_api_route53_1.png)

4.1 *Testando acesso ao site via domínio raiz*
 - Irei acessar o website inserindo o nome do domínio no Browser

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Testing_acess_url_browser.png)

  > **⚠️ Atenção:** Na imagem é possível visualizar o website, o domínio acessado no browser e as infomações de que o site é seguro(Criptografado).

4.2 *Criando uma ACL WAF*
 - O Waf é um serviço de proteção para a camada de aplicação (camada 7 do modelo OSI)
 - Ele permite criar pacote de regras que serão utilizadas para filtrar quem terá acesso aos recursos que estão exposto à internet
 - ACL contra ataques SQLInject, bloqueio geográfico e endereços IP

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_waf_api.png)

 - Escolhi o API pois o tráfego dos usuários passaram por ele

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_waf_api_2.png)

 - Escolhi esse pacotes de regras pois eu irei fazer testes de acessos via geolocalização, onde eu bloquearia o acesso de qualquer usuário do brasil
 - Porém a AWS oferece o pacote que melhor se adapta ao necessário de uma workload que utiliza API

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Acess_web-API.png)

 - Acessando o web-site antes de aplicar a regra que bloqueia acessos com origem do Brasil

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Dashboard_waf.png)

 - Neste Dashboard podemos ver 3 acessos que vinheram do Brasil (meus acessos)
 - Este dashboard integra informações essenciais para que possamos vizualizar ataques ou acessos

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Edit_role_Waf.png)

 - Agora neste dashboar eu alterei a regra de acesso via geolocalização
 - Está regra bloqueia todo acesso com origem do brasil

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Acess_denied_waf.png) 

 - Tentei acesso pelo domínio registrado no Route 53 e podemos ver que recebi Forbidden (acesso negado)

---
5. *Provisionando uma máquina EC2 (Back-End)*
 - Utilizar uma instância EC2 permite que eu tenha total controle e gerencie a instância
 - Configurando ele na mesma VPC que os outros recursos, garantindo comunicação entre eles!

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_ec2%5D.png)

  > **⚠️ Atenção:** Está máquina está em uma sub-rede privada pois ela está executando o back-end do meu projeto. Com isso, não deve ser permitido que a internet tenha acesso. Garatindo maior segurança para seus serviços mais críticos.


  > **⚠️ Explicação sobre o Back-End:** Não foi possível realmente criar uma conexão bem estruturada do ECS com o EC2 porque: Ao utilizar uma image Docker simples como a que estou utilizando, não é possível fazer alterações nela, provisionando uma conexão via API entre eles. É uma imagem simples que só foi utilizada para fins de teste de execução! O que poderia ser feito neste cenário é: Criação de uma API Private, implementar com um Endpoint vpc, vpc link e NLB garantindo as boas práticas de segurança da AWS pois por se tratar de uma conexão crítica, não será possível implementar essa conexão em uma camada pública. Porém, a AWS limitou o provisonamento do ELB e não me permitiu criar! Não consegui de nenhuma forma alterar ou retirar essa limitação. Os recursos estão na mesma VPC, utilizando a Route Table padrão, todos coneseguem se conectar de forma privada e segura utilizando também o SG(Security Group).

5.1 *Criando endpoints para acessar a instância via SSM*
 - Utilizar o ssm facilita na conexão e o gerenciamento de acesso ao recurso
 - Não precisa gerenciar Key Pair ou liberar acesso via SSH no security Group
 - A comunicação é segura e podemos gerenciar as sessões ou iniciar as sessões através do próprio System Manager

  ![Arquitetura-Multi-Tie](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/creating_endpoint_ssm.png)

 - SSM: Endpoint que recebe os comandos que serão executados ao serem solicitados
 - SSmessages: Cria um canal interativo entre seu Browser e o Agente SSM que está instalado na sua máquina
 - EC2messages: Utilizado para envio de menssagens entre o EC2 e o SSM

5.2 *Acessando a instância via SSM*
 - Demora alguns minutos para ser liberado o acesso
 - Recomendo esperar alguns minutos após implementação dos Endpoints para o acesso
 - Reinicie a página e terá a permissão para acessar

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/connecting_ec2_ssm.png)

5.2 *Terminal Linux*
  
   ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/session_ec2_ssm.png)

5.3 *Criando NAT gateway 
 - NAT é um serviço que permite que recursos que estão na camada privada possam ter acesso a internet
 - O recurso tem acesso a internet, mas não permite que o tráfego da internet chegue ao recurso
 - Ele está associado em uma sub-rede pública dedicada somente ao NAT Gateway
 - Utilizado para download de patch para os recursos e fazer download de pacotes necessários

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_Nat.png)

5.4 *Configurando a tabela de roteamento para o NAT Gateway*
 - Para que o recurso que esteja na camada privada tenha acesso a internet é necessário criar uma tabela de rota
 - Essa tabela de rota irá redirecionar o recurso para chegar ao NAT Gateway e direcionar para o IGW

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Route_table_nat.png)

  > **⚠️ Atenção:** Essa configuração teve que ser feita para que fosse baixado o repositório de linha de comando do banco de dados MySql, pois está instância EC2 irá se conectar ao DB e será utiliziado para fazer manipulação de tabela de dados.

5.5 *Baixando Linha de comando do MySql*
 - É necessário que seja baixado a linha de comando do MySQL na EC2
 - Para que a instância consiga executar os comandos no momento de manipular a tabela de dados

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Download_comandLine_db.png)

  > **⚠️ Atenção:** Inseri o comando: Sudo dnf install mariadb105 -y. Este comando permite que eu baixe e instale de maneira automatica com permissões de root.

6.0 *Criando um Banco de Dados no RDS* 
 - O RDS é um serviço que disponibiliza diversos bancos de dados
 - Facilita o provisionamento, automatiza tarefas: Horário para manutenção, backup, Patch
 - Possibilita a criação e alta-disponibilitade do DB, podendo replicar em AZ´s ou utilizar duas AZ´s onde 1 delas será executada, enquanto a outra estará em Standby

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_db.png)

 - Escolhi o MySql DB

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_db_2.png)

 - Escolhi o modelo do banco de dados como sandbox para que não me gere gastos exorbitantes

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_db_3.png)

 - Inserir o nome do banco de dados e a senha que será utilizado para acessar

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_db_4.png)

 - Escolhi o tipo de instância básico e a quantidade de armazenamento mínima para que não ultrapasse meu orçamento

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_db_5.png)

 - Escolhi a mesma VPC em que meus recursos estavam alocados! e não permiti acesso ao público porque só quem terá acesso será minha instância que está em uma sub-rede privada!

6.1 *Acessando o DB via EC2*
 - Necessário inserir alguns comando no terminal para que seja permitido acesso ao db
 - mysql -h endpoint -P 3306 -u admin -p -> esse comando pede para inserir o endpoint do db e o nome do db. Após inserir todas essas informações, insirá a senha que foi criada anteriormente

 ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Comand_db_ssm_ec2.png)

## **Benefícios deste projeto**

 - Alta disponibilidade e escalabilidade usando ECS Fargate, EC2 e API Gateway.
 - Segurança reforçada, com WAF, sub-redes privadas, IAM, ACM e SSM.
 - Arquitetura real de mercado, seguindo o padrão multi-tier (front-end, back-end e banco).
 - Melhor desempenho e confiabilidade, com Route 53, NAT Gateway e VPC bem segmentada.
 - Gerenciamento simplificado, com SSM e integração automatizada entre serviços.
 - Ambiente profissional ideal para portfólio e entrevistas técnicas.


 


