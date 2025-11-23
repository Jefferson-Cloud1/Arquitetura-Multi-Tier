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

2. *criação do cluster ECS*
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
 - È essencial que o tráfego recebido não sobrecarrege somente um container para que seu container a saúde continue integra.

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Config_service_2.png)

2.5 *testando acesso ao website*
 - Testando o site via endereço ip da Task
 - o SG configurado permite acesso HTTP e HTTPs
 - Neste cenário meu acesso estava sendo feito via porta 80 pois ainda não havia criado o certificado ACM

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/testing_acess_ip_public_ecs.png)]

3. *Criação do API Rest*
 - Utilizar o API para direcionar o tráfego ao Webserver
 - Possibilita integração ao WAF, diferente do API HTTP que não é possível
 - Além de que o API Rest disponibiliza diversas funcionalidades para sua API:Cache, integração com WAf...

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_api_rest.png)

3.1 *Criação do Método API*
 - criei o método Get, permitindo somente acesso ao recurso
 - Implementei o endpoint aonde será direcionado o tráfego (Endereço ip da interface de rede do container)
 - O tipo de integração utilizei o HTTP

  ![Arquitetura-Multi-Tier](https://github.com/Jefferson-Cloud1/Arquitetura-Multi-Tier/blob/main/Creating_method_api.png)

3.2 *alterando método de resposta da API*
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

3.7 *criando ACM para associar ao nome de domínio personalizado*
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

  > **⚠️ Atenção:** Na imagem conseguimos visualizar o website, o domínio acessado no browser e as infomações de que o site é seguro(Criptografado).

  
  



 


