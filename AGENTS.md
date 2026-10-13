# Diretrizes do projeto

## Contexto

- Este repositório documenta o TCC do sistema web de monitoramento de uma estufa inteligente.
- Consulte [`PRD.md`](./PRD.md) para requisitos e decisões de produto e [`PROJETO-TECNICO.md`](./PROJETO-TECNICO.md) para arquitetura proposta, modelo de dados, API e plano de implementação.
- O design de referência escolhido é o conceito **01 · Essencial**, disponível em [`prototipos/index.html`](./prototipos/index.html).

## Regras de produto

- O escopo inicial é uma estufa para tomate e pimentão, com monitoramento de temperatura e umidade do ar, limites configuráveis, alertas na aplicação e histórico.
- Diferencie sempre dados simulados de medições reais. Nunca apresente valores demonstrativos como telemetria real.
- O MVP não controla atuadores e não fornece recomendações agronômicas. Não implemente comandos para válvulas, ventilação, bombas ou outros equipamentos sem revisão explícita do escopo.
- Não assuma compatibilidade com hardware, protocolos, limites agronômicos ou comportamento ainda não definido nos documentos; registre as dependências e decisões pendentes.

## Implementação e interface

- Preserve a hierarquia do conceito Essencial: leituras atuais, alerta, histórico e configuração de limites.
- Mostre valor, unidade, horário e estado em texto; não comunique alertas somente por cor.
- Mantenha acessibilidade por teclado, foco visível, rótulos associados, mensagens de erro explícitas e alternativa acessível aos gráficos.
- Não use sucesso silencioso nem substitua falhas de API/sensor por valores padrão que pareçam medições válidas.
- Siga as tecnologias e padrões já presentes no repositório. Quando uma escolha técnica ainda estiver apenas proposta, confirme-a antes de introduzir dependências ou estruturar a aplicação.

## Validação

- Execute os testes, verificações ou comandos de build existentes que cubram as alterações.
- Se a mudança for apenas documental e não houver verificações de documentação, não é necessário executar testes.
- Atualize o PRD ou o projeto técnico quando uma alteração mudar requisitos ou decisões arquiteturais.
