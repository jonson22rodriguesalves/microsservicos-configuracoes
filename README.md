* TechTaste - Configuration Repository
* Sistema de configuração externalizada para a plataforma TechTaste, desenvolvido para Alura - BBTS 2026, que centraliza
* as propriedades de todos os microsserviços utilizando o Spring Cloud Config Server.
 
* 📋 Visão Geral
* Este repositório contém os arquivos de configuração .properties para todos os componentes da arquitetura de 
* microsserviços da plataforma TechTaste, permitindo:
 
* ✅ Configuração Externalizada - Propriedades gerenciadas fora do código fonte
* ✅ Versionamento Centralizado - Todos os ambientes em um único repositório
* ✅ Atualização Dinâmica - Refresh de configurações sem reiniciar serviços
* ✅ Padronização - Configurações consistentes em todos os microsserviços
* ✅ Segurança - Credenciais gerenciadas via variáveis de ambiente
 
* 🏗️ Estrutura do Repositório
* config-repository/
* ├── api-gateway.properties      # Gateway de API
* ├── ms-pagamentos.properties    # Microsserviço de Pagamentos
* ├── ms-pedidos.properties       # Microsserviço de Pedidos
* ├── ms-usuarios.properties      # Microsserviço de Usuários
* ├── service-registry.properties # Service Registry (Eureka Server)
* └── application.properties      # Configurações comuns
 
* 📄 Descrição dos Arquivos de Configuração
* 
* 🔷 application.properties - Configurações Comuns
* Propriedades compartilhadas entre todos os microsserviços.
 
* Propriedade	Valor	Descrição
* eureka.client.register-with-eureka	true	Registra o serviço no Eureka
* eureka.client.fetch-registry	true	Busca registry de outros serviços
* eureka.client.service-url.defaultZone	http://localhost:8081/eureka	URL do Eureka Server
* server.port	0	Porta aleatória para evitar conflitos
* eureka.instance.instance-id	${spring.application.name}:${random.int[1,100]}	ID único da instância
 
* 🔷 api-gateway.properties - API Gateway
* Configuração do gateway central que expõe os microsserviços.
 
* Propriedade	Valor	Descrição
* server.port	8082	Porta fixa para acesso externo
* spring.cloud.gateway.discovery.locator.enabled	true	Habilita roteamento automático
* spring.cloud.gateway.discovery.locator.lower-case-service-id	true	Utiliza IDs em lowercase
* Funcionalidade: O gateway expõe automaticamente rotas como /{service-id}/** para todos os serviços registrados no Eureka.
 
* 🔷 ms-pagamentos.properties - Microsserviço de Pagamentos
* Configuração do serviço de processamento de pagamentos.
 
* Propriedade	Valor	Descrição
* spring.datasource.url	jdbc:postgresql://localhost:5432/ms-pagamentos-db	Banco de dados PostgreSQL
* spring.datasource.username	${USER}	Usuário via variável de ambiente
* spring.datasource.password	${PASSWORD}	Senha via variável de ambiente
* spring.jpa.hibernate.ddl-auto	update	Atualização automática do schema
 
* 🔷 ms-pedidos.properties - Microsserviço de Pedidos
* Configuração do serviço de gerenciamento de pedidos com resilience4j.
 
* Propriedade	Valor	Descrição
* spring.datasource.url	jdbc:postgresql://localhost:5432/ms-pedidos-db	Banco de dados PostgreSQL
* spring.jpa.show-sql	true	Exibe SQL no log (para desenvolvimento)
* spring.rabbitmq.addresses	${AMQP}	Endereço RabbitMQ via variável
* resilience4j.circuitbreaker.instances.verificaAutorizacao.slidingWindowSize	3	Tamanho da janela deslizante
* resilience4j.circuitbreaker.instances.verificaAutorizacao.slidingWindowType	COUNT_BASED	Tipo de janela baseada em contagem
* resilience4j.circuitbreaker.instances.verificaAutorizacao.minimumNumberOfCalls	5	Mínimo de chamadas para avaliação
* resilience4j.circuitbreaker.instances.verificaAutorizacao.waitDurationInOpenState	10s	Tempo de espera no estado aberto
 
* 🔷 ms-usuarios.properties - Microsserviço de Usuários
* Configuração do serviço de gestão de usuários e autenticação.
 
* Propriedade	Valor	Descrição
* spring.datasource.url	jdbc:h2:mem:usuarios-db	Banco H2 em memória
* spring.rabbitmq.addresses	${RABBIT}	Endereço RabbitMQ via variável
* spring.mail.host	smtp.gmail.com	Servidor SMTP
* spring.mail.port	587	Porta SMTP
* spring.mail.username	${EMAIL}	Email via variável de ambiente
* spring.mail.password	${PASSWORD}	Senha via variável de ambiente
* spring.mail.properties.mail.smtp.auth	true	Autenticação SMTP
* spring.mail.properties.mail.smtp.starttls.enable	true	TLS habilitado
 
* 🔷 service-registry.properties - Service Registry
* Configuração do servidor Eureka para descoberta de serviços.
 
* Propriedade	Valor	Descrição
* server.port	8081	Porta fixa para o Eureka Server
* eureka.client.register-with-eureka	false	Não se registra como cliente
* eureka.client.fetch-registry	false	Não busca registry de outros
 
* 🔧 Variáveis de Ambiente Necessárias
* Variável	Descrição	Onde Utilizada
* USER	Usuário do banco de dados	ms-pagamentos.properties, ms-pedidos.properties
* PASSWORD	Senha do banco de dados/email	ms-pagamentos.properties, ms-pedidos.properties, ms-usuarios.properties
* EMAIL	Email para envio de notificações	ms-usuarios.properties
* AMQP	Endereço do RabbitMQ	ms-pedidos.properties
* RABBIT	Endereço do RabbitMQ	ms-usuarios.properties
 
* 📊 Arquitetura de Configuração
````mermaid
graph TB
    subgraph "Configuração Centralizada"
        Git[GitHub Repositório]
        Config[Spring Cloud Config Server]
        Git -->|Pull| Config
    end
    
    subgraph "Microsserviços"
        Gateway[API Gateway<br/>:8082]
        Pedidos[ms-pedidos<br/>:0]
        Pagamentos[ms-pagamentos<br/>:0]
        Usuarios[ms-usuarios<br/>:0]
    end
    
    subgraph "Infraestrutura"
        Eureka[Eureka Server<br/>:8081]
        Rabbit[RabbitMQ]
        Postgres[(PostgreSQL)]
        H2[(H2 Database)]
    end
    
    Config -->|Configuração| Gateway
    Config -->|Configuração| Pedidos
    Config -->|Configuração| Pagamentos
    Config -->|Configuração| Usuarios
    
    Gateway -->|Registro| Eureka
    Pedidos -->|Registro| Eureka
    Pagamentos -->|Registro| Eureka
    Usuarios -->|Registro| Eureka
    
    Pedidos -->|Circuit Breaker| Gateway
    Pedidos -->|Messaging| Rabbit
    Usuarios -->|Messaging| Rabbit
    
    Pedidos -->|JDBC| Postgres
    Pagamentos -->|JDBC| Postgres
    Usuarios -->|H2| H2

    classDef config fill:#FF9800,stroke:#333,stroke-width:2px,color:#fff
    classDef service fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff
    classDef infra fill:#4CAF50,stroke:#333,stroke-width:2px,color:#fff
    classDef cloud fill:#9E9E9E,stroke:#333,stroke-width:2px,color:#fff
    
    class Git,Config config
    class Gateway,Pedidos,Pagamentos,Usuarios service
    class Eureka,Rabbit,Postgres,H2 infra

````

* 🚀 Como Utilizar
* 1. Configurar o Spring Cloud Config Server
* # bootstrap.yml do Config Server
* spring:
* cloud:
* config:
* server:
* git:
* uri: https://github.com/seu-usuario/config-repository
* default-label: main
 
* 2. Configurar os Microsserviços
* # bootstrap.yml de cada serviço
* spring:
* application:
* name: ms-pedidos  # Nome deve corresponder ao arquivo .properties
* cloud:
* config:
* uri: http://config-server:8888
 
* 3. Atualizar Configurações em Runtime
* # Enviar POST para atualizar configurações
* curl -X POST http://localhost:8082/actuator/refresh
 
* 🔍 Troubleshooting
* Problema	Causa Comum	Solução
* Configuração não carregada	Service name incorreto	Verificar spring.application.name
* Variáveis de ambiente não resolvidas	Variáveis não definidas	Definir todas as variáveis listadas
* Conexão com Eureka falha	Eureka não está rodando	Iniciar Eureka na porta 8081
* RabbitMQ não conecta	AMQP variável inválida	Verificar formato host:port
* Circuit Breaker não funciona	Configuração incorreta	Verificar resilience4j.* propriedades
 
* 📚 Boas Práticas Implementadas
* ✅ Separação por Ambiente: Propriedades específicas por serviço
* ✅ Sensíveis Não Versionados: Credenciais via variáveis de ambiente
* ✅ Configuração Padronizada: Propriedades comuns no application.properties
* ✅ Resiliência: Circuit Breaker configurado para tolerância a falhas
* ✅ Descoberta Automática: Integração com Eureka em todos os serviços
* ✅ Atualização Dinâmica: Suporte a refresh de configurações
 
 
* 🔗 Integrações
* Componente	Propósito	Serviços que Utilizam
* Eureka Server	Service Discovery	Todos os serviços
* PostgreSQL	Banco de dados relacional	ms-pagamentos, ms-pedidos
* H2 Database	Banco em memória	ms-usuarios
* RabbitMQ	Mensageria assíncrona	ms-pedidos, ms-usuarios
* Gmail SMTP	Envio de emails	ms-usuarios
 
* 📝 Notas Adicionais
* Spring Boot 3.5.16: Todos os serviços utilizam a versão mais recente do Spring Boot
* Java 21: Compatível com LTS mais recente
* Portas Dinâmicas: server.port=0 evita conflitos entre instâncias
* Instance ID: Formato {app-name}:{random} para identificação única
* Refresh Scope: @RefreshScope para beans que utilizam configurações