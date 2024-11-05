# Sprint_net-4


CurriculumEnhancerAPI
CurriculumEnhancerAPI é uma API RESTful desenvolvida em .NET Core para analisar currículos e fornecer sugestões de melhoria. Além disso, ela possui funcionalidades de análise de sentimento para identificar o tom emocional de um texto.

Funcionalidades
Análise de Currículo: Oferece sugestões para melhorar a qualidade do conteúdo de currículos.
Análise de Sentimento: Identifica o tom do texto enviado, utilizando modelos de Machine Learning com ML.NET.
Resiliência de Conexão: Implementa política de retry com Polly para chamadas de API externas, garantindo maior estabilidade.
Tecnologias Utilizadas
ASP.NET Core: Para a construção da API.
ML.NET: Para o treinamento e execução do modelo de análise de sentimento.
Polly: Para políticas de retry e resiliência na chamada de serviços externos.
Swagger: Para documentação interativa da API.
Injeção de Dependência: Utilizada para facilitar a testabilidade e modularidade do código.

 # Estrutura do Projeto
 CurriculumEnhancerAPI/
│
├── Controllers/
│   └── CurriculumController.cs      # Controlador principal para os endpoints de análise
│
├── Models/
│   ├── CurriculumApiSettings.cs     # Configurações da API para integração externa
│   ├── SentimentData.cs             # Modelos de dados para análise de sentimento
│
├── Services/
│   ├── ICurriculumService.cs        # Interface do serviço de análise de currículo
│   ├── ISentimentService.cs         # Interface do serviço de análise de sentimento
│   ├── CurriculumService.cs         # Implementação da análise de currículo
│   └── SentimentService.cs          # Implementação da análise de sentimento
│
└── Program.cs                       # Configuração do host e DI
└── appsettings.json                 # Configuração de ambientes e URL da API externa

# Pré-requisitos

.NET 8 SDK ou superior: Instale o SDK

ML.NET: Instalado via NuGet no projeto.

Polly: Instalado via NuGet no projeto para gerenciar resiliência.

JSON Server (opcional): Simular uma API local para testes

Instalação dos Pacotes Necessários 

Execute os seguintes comandos no Package Manager Console para instalar as dependências:

dotnet add package Polly
dotnet add package Polly.Extensions.Http
dotnet add package Microsoft.ML
dotnet add package Microsoft.ML.FastTree

Configuração

No arquivo appsettings.json, configure a URL base para a API externa (ou uma API simulada para testes locais):

{
  "CurriculumAnalysisApi": {
    "BaseUrl": "http://localhost:5000/analyze"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}

Endpoints da API

1. Análise de Currículo
2. 
Endpoint: POST /api/Curriculum/analyze

Descrição: Recebe um currículo em formato de texto e retorna sugestões de melhorias.

Exemplo de Requisição:

curl -X 'POST' \
  'https://localhost:7125/api/Curriculum/analyze' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
        "text": "Minha experiência inclui desenvolvimento de software..."
      }'
Exemplo de Resposta:

{
  "suggestions": [
    "Use palavras mais objetivas",
    "Evite repetição"
  ]
}

2. Análise de Sentimento
Endpoint: POST /api/Curriculum/sentiment

Descrição: Analisa o tom emocional do texto fornecido, utilizando um modelo de ML para determinar o sentimento positivo ou negativo.

Exemplo de Requisição:

curl -X 'POST' \
  'https://localhost:7125/api/Curriculum/sentiment' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
        "text": "Este é o melhor produto que já usei!"
      }'
Exemplo de Resposta:

{
  "prediction": true,
  "probability": 0.85,
  "score": 0.95
}

Estrutura de Classes e Serviços

CurriculumService.cs

Este serviço é responsável por realizar chamadas para a API externa que analisa o currículo. Implementa a interface ICurriculumService para facilitar testes e modularidade.

public interface ICurriculumService
{
    Task<AnalysisResponse> AnalyzeCurriculumAsync(string text);
}

SentimentService.cs

Este serviço usa ML.NET para análise de sentimento. Ele carrega um modelo pré-treinado e utiliza PredictSentiment para classificar o sentimento do texto.

public interface ISentimentService
{
    SentimentPrediction PredictSentiment(string text);
}

Configuração de HttpClient com Polly para Retries

No Program.cs, configuramos o HttpClient com política de retry usando Polly para lidar com erros temporários na comunicação com a API externa:

builder.Services.AddHttpClient<ICurriculumService, CurriculumService>()
    .AddPolicyHandler(HttpPolicyExtensions
        .HandleTransientHttpError()
        .WaitAndRetryAsync(3, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))));
