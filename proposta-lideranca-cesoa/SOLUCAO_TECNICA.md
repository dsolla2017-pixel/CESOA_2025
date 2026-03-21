# Desafio Técnico PSI-CTI - CAIXA
## Processamento Assíncrono de Contratos de Crédito (Método PRICE)

**Autor:** Gestor de TI, Especialista em Data Driven e IA Senior (Microsoft, IBM, CIA)
**Objetivo:** Solução arquitetural ponta a ponta orientada a serviços para o desafio PSI-CTI (Coordenador de TI - CESOA), focada em inovação, qualidade total e reposicionamento ao cliente.

---

### 1. Visão Estratégica e Legado Institucional

A solução proposta transcende a simples entrega de código. Ela estabelece uma arquitetura de referência para a CAIXA, alinhada à Estratégia 2030, focada na transformação digital sustentável e na melhoria contínua da experiência do cliente.

O reposicionamento ao cliente exige processos ágeis, resilientes e transparentes. A arquitetura de dados e a estrutura de dados implementadas garantem que cada interação do cidadão seja processada com precisão matemática (cálculo PRICE diário) e total rastreabilidade (observabilidade via Event Hub). O legado institucional desta entrega reside na padronização "digital first", reduzindo riscos operacionais e capacitando as áreas de negócio com dados confiáveis para tomada de decisão (Data Driven).

A liderança transformadora atua como facilitadora, integrando as diretrizes da CESOA, GEPCA, SUART, DESOL e VITEC. A cultura ágil (Scrum/SAFe) é incorporada "by design", com ciclos de entrega incrementais, qualidade embutida (testes automatizados) e governança de IA preparada para o futuro.

### 2. Arquitetura da Solução

A arquitetura adota o padrão **Clean Architecture** combinado com **Event-Driven Architecture (EDA)**, garantindo baixo acoplamento, alta coesão e escalabilidade.

| Componente | Tecnologia | Responsabilidade | Inovação e Valor |
| :--- | :--- | :--- | :--- |
| **Worker Service** | .NET 8 (BackgroundService) | Consumo assíncrono da fila (Azure Service Bus), orquestração e controle de idempotência. | Resiliência nativa, processamento "Compete Consumer" escalável, tolerância a falhas transientes. |
| **API PRICE** | .NET 8 (Minimal API/MVC) | Motor de cálculo financeiro. Executa a conversão de taxas e gera a evolução diária do contrato. | Desacoplamento do motor de cálculo. Permite evolução futura para outros sistemas de amortização (SAC, SACRE). |
| **Persistência** | SQL Server + EF Core 8 | Armazenamento seguro da evolução do contrato (tabela `EvolucaoContrato`). | Bulk insert com transações implícitas para alta performance. Índices otimizados para consultas de auditoria. |
| **Mensageria/Logs** | Azure Event Hub | Recepção de telemetria de alto volume (início, sucesso, erro, métricas). | Habilita monitoramento em tempo real (Databricks/Azure Monitor). Desacopla produtores de consumidores de log. |

### 3. Instruções para Execução Local

A solução foi projetada para ser executada localmente de forma simples, utilizando Docker para emular o ambiente de produção.

**Pré-requisitos:**
* Docker e Docker Compose instalados.
* .NET 8 SDK (opcional, apenas para execução fora do Docker).
* Visual Studio 2022 ou VS Code.

**Passos para execução via Docker (Recomendado):**

1. Abra o terminal na raiz do projeto (onde está o arquivo `docker-compose.yml`).
2. Execute o comando:
   `docker-compose up -d --build`
3. O ambiente iniciará os seguintes serviços:
   * **API PRICE:** Disponível em `http://localhost:5100` (Swagger em `http://localhost:5100/swagger`)
   * **SQL Server:** Disponível na porta `1433` (Credenciais no `docker-compose.yml`)
   * **Worker Service:** Rodando em background, aguardando mensagens.

**Validação e Testes:**

1. Acesse o Swagger da API (`http://localhost:5100/swagger`).
2. Utilize o endpoint `POST /api/price/calcular` com o seguinte payload de teste:
   ```json
   {
     "valorEmprestimo": 10000,
     "taxaJurosMensal": 1.8,
     "prazoMeses": 30
   }
   ```
3. A API retornará os 900 registros diários.
4. Para testar o fluxo assíncrono completo, envie uma mensagem para a fila configurada no Azure Service Bus (necessário provisionar o recurso no Azure e atualizar as connection strings no `appsettings.json` do Worker).
5. O Worker consumirá a mensagem, chamará a API, persistirá no banco de dados e enviará o log para o Event Hub.

### 4. Boas Práticas e Qualidade Total (Built-in Quality)

* **Idempotência:** O Worker verifica no banco de dados se o `MessageId` (ou `ContratoId`) já foi processado antes de iniciar o cálculo. Isso previne duplicidade em caso de reprocessamento pela fila.
* **Testes Automatizados:** O projeto `CreditoPrice.Tests` contém testes unitários (xUnit) que validam a precisão matemática do motor de cálculo PRICE, garantindo que o saldo final seja zero e a prestação seja fixa.
* **Observabilidade:** O uso do Event Hub para telemetria estruturada permite rastreabilidade completa de cada contrato processado, essencial para a governança e auditoria da CAIXA.
* **Clean Code:** Código comentado, tipagem forte (Enums), validação de domínio em Value Objects (Fail Fast) e separação clara de responsabilidades (Interfaces/DI).

### 5. Governança de IA e Próximos Passos

A arquitetura de dados estabelecida nesta entrega prepara o terreno para a integração de Inteligência Artificial na CAIXA. Os dados limpos, estruturados e anonimizados gerados pelo processamento de crédito (armazenados no Data Lake/Databricks via Event Hub) podem alimentar modelos preditivos de inadimplência, análise de perfil de risco e hiperpersonalização de ofertas de crédito. A conformidade com a OR-220 (Diretrizes para Analytics e IA) é garantida pela rastreabilidade e transparência de cada decisão automatizada.

Esta proposta reafirma o compromisso com a excelência técnica e a liderança estratégica, pilares fundamentais para o papel de Coordenador de TI na CESOA.
