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
