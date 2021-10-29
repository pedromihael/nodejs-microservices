🇧🇷 pt-br

# Microsserviços com Node.js

Este é um trabalho prático da disciplina de **Computação Distribuída P01**, ministrada por **Roberto Aragy**, desenvolvido pelo acadêmico **Pedro Mihael Viana Paes RGA 2018.1906.103-5**, com objectivo de compor a nota final da disciplina.

Neste presente repositório, encontram-se links para os repositórios de cada um dos componentes do sistema: os serviços e o API Gateway, além do frontend. Basta clicar no submódulo que você será redirecionado para o repositório correspondente.

A motivação para a separação em módulos (funcionalidade do GitHub) é a separação de branchs, issues e PRs mais bem organizada, colaborando para uma melhor manutenção do código posteriormente.

### Objetivo do Trabalho

Desenvolver um sistema distribuído, composto por domínios, onde cada domínio deve estar correlacionado a um tipo de entidade do banco de dados, tornando o consumo e produção de dados de cada tipo de entidade independente.

### Arquitetura Escolhida

Para este trabalho, foi escolhida a arquitetura de microsserviços, os quais estão disponíveis para o frontend através de um middleware com endpoint único, a fim de garantir a distribuição dos sistemas, possibilidade de escalonamento e replicação caso necessário, mas com sua complexidade a nível de rede mascarada por este intermediador, chamado aqui de API Gateway. Este middleware trata as requisições da seguinte maneira:

![Arquitetura](assets/fig01.jpg?raw=true "fig01")

Os domínios disponíveis são:

- `/incidents/[param]`
- `/projects/[param]`
- `/providers/[param]`
- `/reliabilities/[param]`
- `/severities/[param]`

O frontend faz requisições para o enpoint do API Gateway, na porta 3333. Nele, a rota é identificada e a requisição é passada adiante para o serviço correspondente ao domínio da rota, o qual retorna os dados solicitados. Como são microsserviços, cada um possui sua própria instância de banco de dados, caso contrário, seriam apenas "micro-monolitos". 

Incidentes criados causam impactos em projetos e provedores. Para lidar com esta regra de negócio, canais do **Apache Kafka** foram criados (implantados juntamente ao API Gateway) e por eles, através de **pub/sub**, os serviços estabelecem intercomunicação. Um incidente é registrado, um projeto é avisado que precisa alterar sua confiabilidade, assim como o provedor relacionado.

### Estrutura de Diretórios

Os diretórios estão organizados da seguinte maneira:

![Diretórios](assets/fig02.jpg?raw=true "fig02")

A organização dos diretórios está baseada no Domain Driven Design, na Clean Architecture e adaptada para trabalhar com o padrão de repositórios. Para este trabalho, apenas o repositório de Postgres foi implementado, mas esta organização permite o uso de outros bancos, bastando apenas a implementação dos métodos básicos como feito com o de Postgres, e sua importação para uso com os useCases, uma vez que o repositório utilizado pelos useCases é uma dependência injetada nestes.

### Escopo do Trabalho

Com intuito de controlar a confiabilidade de projetos e provedores de serviços de software, este sistema simula o registro de incidentes acontecidos em produção, estabelecendo severidades de 1 a 4 para cada incidente. 

- Projetos são criados com fornecimento de serviços de provedores e enviados para produção.
- Incidentes acontecem e são registrados, atribuindo a estes uma severidade de 1 a 4. 
- A cada incidente registrado, o projeto correspondente e o provedor relacionado tem sua confiabilidade diminuídas proporcionalmente ao peso da severidade, sem a chance de retornar ao que era inicialmente.
- Confiabilidades de projetos abaixo de 95 são consideradas abaixo da meta. O mesmo acontece para confiabilidades de provedores abaixo de 98.
- As confiabilidades-meta podem ser alteradas a gosto do administrador do sistema.

### Relacionamentos

- Um projeto possui um provedor.
- Um provedor possui um ou mais projetos.
- Um projeto possui nenhum ou vários incidentes.
- Um incidente possui uma única severidade.
- Uma confiabilidade-meta pertence a vários projetos.
- Uma confiabilidade-meta pertence a vários provedores. 

### Tecnologias utilizadas

- Backend (Javascript ^ES6)
  - [x] Node.js com Express
  - [x] Morgan para log de requisições
  - [x] Knex para query building
  - [x] Postgres como banco de dados
  - [x] Apache Kafka para intercomunicação entre microsserviços via pub/sub

- Frontend (Javascript ^ES6)
  - [x] React JS
  - [x] Styled Components
  - [x] Material Design by Google

- Devops
  - [x] Docker
  - [x] Docker-compose

### Testes dos Microsserviços no Postman

Dentro do diretório `collections` neste repositório, encontra-se uma collection do Postman com as requisições de CRUD utilizadas para testar os microsserviços.


### Como executar 

Você vai precisar de Docker e Docker-compose instalados na sua máquina.

1. Crie uma pasta para aglomerar os microsserviços localmente.
2. Dentro dela, clone cada um dos componetes em pastas separadas, incluindo o frontend e a API Gateway.
3. Dentro de cada pasta, execute `docker-compose up`. 
4. Caso existam logs de falha de conexão com a porta 9092, reinicie o container da API Gateway com o mesmo comando acima, o Kafka pode não ter inicializado corretamente.
5. Após inicializar o frontend, abra-o em `localhost:3000`.
6. Para testar com o postman, aponte para `localhost:3333`.

### Dúvidas

Caso restem dúvidas, peço que mande-me uma mensagem através do [Linkedin](https://linkedin.com/in/opedropaes) ou no email pedromvpaes@gmail.com.