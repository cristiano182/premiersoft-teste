# Documentação de Design do Microserviço

## Visão Geral

Este documento detalha um microserviço projetado para lidar com grandes volumes de dados e garantir escalabilidade e manutenção. Ele inclui endpoints robustos de API para criação e leitura de dados, suportados por implantação conteinerizada e Kubernetes para orquestração.

1. **Introdução**
2. **Requisitos e Objetivos**
3. **Visão Geral da Arquitetura**
4. **Pilha de Tecnologia**
5. **Containerização**
6. **Implantação no Kubernetes**
7. **Pipeline de CI/CD**
8. **Estratégia de Testes**

## 1. Introdução
As duas funcionalidades seguintes são propostas para o microserviço:
- **POST /data**: Aceitar e validar entrada JSON antes de persistir em um banco de dados.
- **GET /data**: Recuperar e retornar os dados armazenados.

O microserviço foi projetado para implementar uma arquitetura REST **sem estado**, com um design em camadas para separação de responsabilidades.

## 2. Requisitos e Objetivos

### Requisitos Funcionais
- Aceitar entrada JSON via uma API REST.
- Validar e persistir os dados.
- Recuperar dados armazenados.

### Objetivos de Desempenho
Um objetivo chave do design é manter tempos de resposta abaixo de **500ms**. Além disso, o sistema deve ser capaz de suportar bilhões de registros com menos de **10% de degradação de desempenho**.

## 3. Visão Geral da Arquitetura

### **Diagrama: Arquitetura em Camadas**
![Arquitetura em Camadas](./diagrams/api-diagram.png)

### Componentes
1. **Camada de API**
   - Controladores que lidam com requisições HTTP.
   - Realiza validação dos dados de entrada.
   - Formata as respostas.
2. **Camada de Serviço**
   - Singletons que gerenciam a lógica de negócios.
   - Interface com a camada de banco de dados para operações CRUD.
3. **Camada de Banco de Dados**
   - Armazenamento persistente de dados.
   - Suporta operações escaláveis de leitura e escrita.

## 4. Pilha de Tecnologia

### Justificativas
- **Banco de Dados:** PostgreSQL
  - Suporta alta escalabilidade e conformidade com ACID.
  - Suporte extensivo para indexação e otimização de consultas.
- **Framework:** NestJS (Node.js)
  - Leve, I/O não bloqueante.
  - Estrutura opinativa para manutenibilidade.
- **Containerização:** Docker
  - Garante portabilidade e consistência.
- **Orquestração:** Kubernetes
  - Suporta escalabilidade horizontal e tolerância a falhas.

## 5. Containerização


### Dockerfile
```Dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

CMD [ "npm", "run", "start:dev" ]
```

### Explicação
O Dockerfile utiliza uma imagem base leve do Node.js para garantir desempenho ideal e uma pegada reduzida. Ele começa configurando o diretório de trabalho dentro do contêiner para manter uma estrutura limpa e organizada. Os arquivos relacionados ao pacote são copiados primeiro para aproveitar o mecanismo de cache do Docker, o que permite que as dependências sejam instaladas de maneira eficiente usando o npm. O código-fonte da aplicação é então copiado para o contêiner, seguido pela exposição da porta necessária para permitir o acesso externo. Por fim, a aplicação é iniciada usando o script de produção do NestJS, garantindo que ela seja executada conforme esperado em um ambiente de produção.

## 6. Implantação no Kubernetes

### Estratégia de Implantação
- Escalonamento automático de Pods horizontal baseado no uso de CPU.
- PersistentVolumeClaims para armazenamento de banco de dados.

### Exemplos de Arquivos YAML

#### Implantação
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: data-service
  template:
    metadata:
      labels:
        app: data-service
    spec:
      containers:
      - name: data-service
        image: data-service:latest
        ports:
        - containerPort: 3000
```

#### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: data-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: data-service
```

## 7. Pipeline de CI/CD

### Diagrama da Pipeline
![Pipeline de CI/CD](./diagrams/cicd-diagram.png)

### Etapas
1. **Testes Unitários:** Validar funcionalidades principais.
2. **Testes de Integração:** Validar interações entre a API e o banco de dados.
3. **Construção do Docker:** Criar e publicar a imagem do contêiner.
4. **Implantação no Kubernetes:** Implantar a imagem mais recente.

## 8. Estratégia de Testes

### Testes Unitários
- Foco: Validação de entrada, interação com o banco de dados.
- Ferramentas: Jest, mocha.
- Focar menos na cobertura geral e mais nos caminhos críticos.
- Garantir que um bom número de cenários de validação de dados seja coberto. A consistência dos dados é extremamente importante para este serviço.

### Testes de Integração
- Validar a camada da API e as interações com o banco de dados.
- Ter um script para popular o banco de dados com dados de teste e então chamar os endpoints da API, garantindo que os dados sejam retornados como esperado.
- Garantir que o banco de dados esteja em um estado limpo antes e depois dos testes.

### Testes de Desempenho
- Simular alta concorrência usando ferramentas como JMeter.

### Testes Manuais
- Para um conjunto mínimo de endpoints conforme proposto nos requisitos, testes manuais podem ser feitos para adicionar uma camada extra de validação.

### Começando
**Visão Geral do Projeto:** Microserviço para ingestão e recuperação de dados com foco em escalabilidade.

**Configuração:**
1. Clone o repositório.
2. Execute `docker-compose up`.
3. Acesse a API em `http://localhost:3000`.


