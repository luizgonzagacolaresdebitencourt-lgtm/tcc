# Projeto técnico — Estufa Clara

**Produto:** sistema web de monitoramento de estufa inteligente  
**Design de referência:** Conceito 01 · Essencial, do conjunto em [`prototipos/index.html`](./prototipos/index.html)  
**Documento relacionado:** [`PRD.md`](./PRD.md)  
**Status:** proposta técnica para orientar a implementação  
**Versão:** 0.1 · 13/10/2026

## 1. Objetivo e limites

Este documento converte o PRD e o design Essencial em uma proposta implementável: arquitetura, módulos, modelo de dados, contratos de API, fluxos, segurança, estratégia de testes e etapas de entrega.

O MVP monitora **uma estufa**, para tomate e pimentão, mostrando temperatura e umidade relativa do ar. A primeira demonstração usa leituras simuladas; a aquisição por sensores físicos vem depois de identificar equipamentos e protocolos. O sistema não controla atuadores e não oferece recomendações agronômicas.

As tecnologias abaixo são uma recomendação inicial, não uma decisão registrada no PRD. O repositório atualmente contém documentação e protótipos HTML, sem uma aplicação ou stack implantada. A implementação deve validar esta proposta com as restrições do TCC, a experiência da equipe e o ambiente de hospedagem.

## 2. Princípios técnicos e de produto

1. **Não confundir simulação com telemetria:** todo registro carrega sua origem; a interface mantém o selo de simulação quando aplicável.
2. **Uma fonte de verdade para cada leitura:** a leitura observada, limites aplicados e alertas são registrados com horários explícitos.
3. **Sem controle físico:** não criar endpoints, credenciais ou interfaces para acionar válvulas, ventiladores, bombas ou outros atuadores no MVP.
4. **Falhas visíveis:** dado ausente ou antigo não deve ser apresentado como se fosse atual ou normal.
5. **Integração desacoplada:** o painel e o domínio não devem depender de um fabricante específico de sensores.
6. **Acessibilidade e clareza:** valor, unidade, limite, estado e horário devem ser compreensíveis sem depender apenas de cor ou interação com mouse.

## 3. Arquitetura proposta

### 3.1 Stack recomendada

| Camada | Proposta | Motivo |
|---|---|---|
| Interface | React + TypeScript + Vite | Componentes reutilizáveis, tipagem e desenvolvimento local simples. |
| API e regras | Python + FastAPI | API HTTP tipada, validação de entrada e integração natural com tarefas de ingestão. |
| Persistência | PostgreSQL | Transações, integridade relacional e consultas indexadas por tempo suficientes para uma estufa e leitura horária. |
| Gráficos | SVG acessível ou biblioteca React pequena e estável | Manter legenda, unidades, filtros e alternativa tabular acessíveis. Escolher a biblioteca durante a implementação e validar seu suporte a teclado/leitor de tela. |
| Execução local | Docker Compose para API, interface e banco | Ambiente de desenvolvimento repetível e caminho simples para uma implantação pequena. |

Não há necessidade inicial de banco especializado em séries temporais, broker de mensagens ou plataforma de analytics: o volume esperado é baixo (aproximadamente 17.520 leituras por variável/ano se coletado uma vez por hora). PostgreSQL comum, índices por data e agregação na consulta devem ser suficientes. Reavaliar com dados reais e retenção confirmada.

### 3.2 Componentes lógicos

```text
Navegador (desktop / responsivo)
        │ HTTPS · sessão autenticada
        ▼
Aplicação web (React)
        │ REST/JSON · /api/v1
        ▼
API (FastAPI)
 ├── autenticação e autorização
 ├── leitura atual e histórico
 ├── configuração/versionamento de limites
 ├── avaliação de alertas
 └── ingestão e normalização de leituras
        │
        ▼
PostgreSQL
 ├── medições (source: simulated | sensor)
 ├── limites e versões
 ├── alertas e eventos
 └── usuário e auditoria essencial

MVP: gerador de dados simulados ──────────► API
Futuro: sensores ─► gateway local ─HTTPS──► API
             armazenamento local e sincronização após reconexão
```

O gerador simulado e o futuro adaptador de hardware devem implementar a mesma interface interna de ingestão. Isso permite trocar a fonte de dados sem redesenhar o painel nem misturar o simulador com os componentes de apresentação.

### 3.3 Implantação

**Desenvolvimento:** interface, API e PostgreSQL em Docker Compose; dados demonstrativos carregados por comando ou tarefa de desenvolvimento.

**Demonstração remota:** hospedar a aplicação e API em ambiente HTTPS; manter o banco em rede privada, com backup automatizado e acesso administrativo restrito. A URL pública deve exigir autenticação. Nunca expor diretamente o PostgreSQL ou controladores de campo à internet.

**Integração física futura:** gateway instalado na rede da estufa coleta sensores, conserva amostras durante indisponibilidade da internet e retransmite ao serviço remoto. A interface web pode ficar indisponível durante uma queda, mas a aquisição local não deve parar. O protocolo e o hardware do gateway serão definidos após inventário; MQTT, Modbus ou OPC UA são possibilidades, não requisitos fixados.

## 4. Mapeamento do conceito Essencial

O conceito organiza a página de cima para baixo:

1. **Identidade e contexto:** título da estufa, cultivos, última leitura e indicação de dados simulados.
2. **Leituras atuais:** dois cartões, um para temperatura e outro para umidade; cada cartão contém valor, unidade, faixa definida e estado textual.
3. **Alerta ativo:** variável, leitura, limite violado e horário; ocultar a região ou exibir estado “Nenhum alerta ativo” quando não houver eventos abertos.
4. **Histórico:** gráfico com períodos de 24 h, 7 dias, 30 dias, 90 dias e 1 ano.
5. **Limites e frequência:** acesso à configuração e explicação da atualização horária e do possível atraso.
6. **Proveniência e segurança:** o selo “Dados simulados” permanece visível sempre que se mostra conteúdo gerado.

### 4.1 Componentes de interface

- `GreenhousePage`: estrutura da página, cabeçalho e layout responsivo.
- `SimulationBadge`: origem da série exibida, em texto além do marcador visual.
- `CurrentMetricCard`: nome da variável, valor formatado, unidade, horário, faixa e estado.
- `AlertBanner`: resumo de evento aberto, sem instruções de manejo nem comandos de equipamento.
- `HistoryChart`: seleção de período, escalas/unidades, legenda e tabela alternativa.
- `ThresholdSettingsDialog`: formulário para os limites das duas variáveis, validação e confirmação.
- `ConnectionFreshness`: horário da amostra mais recente e estado de atraso ou indisponibilidade.
- `Toast`/mensagem de estado: confirma salvamento ou apresenta falha; erro nunca pode parecer sucesso.

O protótipo é uma referência visual e de interação, não uma implementação literal. Em especial, o gráfico demonstrativo deve ser substituído por dados reais provenientes da API. Temperatura e umidade têm unidades e escalas diferentes; os eixos precisam ser identificados e validados para não sugerir comparabilidade numérica enganosa.

### 4.2 Estados de tela que a implementação deve cobrir

- **Carregando:** esqueleto discreto ou estado de carregamento anunciado sem deslocar foco.
- **Sem leituras:** explicar que nenhuma leitura foi recebida; não renderizar números fictícios como leitura real.
- **Simulação ativa:** selo persistente e valores claramente demonstrativos.
- **Tudo dentro da faixa:** estado textual “Dentro da faixa” e nenhuma região de alerta ativa.
- **Limite ultrapassado:** estado textual, ícone e informação da variável, valor, limite e hora.
- **Leitura desatualizada:** exibir a última hora conhecida e informar que ela pode não refletir o estado atual. Como ponto de partida, tratar como desatualizada após duas janelas esperadas (duas horas), configurável no projeto e sujeito à confirmação operacional.
- **Erro de rede/API:** manter a última leitura com indicação explícita de que está desatualizada; oferecer tentar novamente.
- **Falha de validação:** erro próximo ao formulário e indicação acessível dos campos inválidos.

## 5. Fluxos funcionais

### 5.1 Consultar painel

1. O navegador verifica a sessão.
2. Carrega a leitura mais recente por variável, os limites vigentes e os alertas abertos.
3. Apresenta a fonte, hora de coleta e estado de atualidade.
4. Atualiza a tela aproximadamente de hora em hora, consultando a API; não atualizar continuamente nem deslocar foco.
5. Em falha, preserva a última leitura e marca-a como desatualizada, sem converter erro em valor zero ou estado normal.

O PRD aceita até uma hora de atraso. O intervalo da tarefa de simulação e do polling deve ser coordenado de modo que uma leitura nova esteja disponível dentro dessa expectativa; registrar horário de coleta e horário de recebimento separadamente.

### 5.2 Alterar limites

1. O usuário abre o diálogo pelo botão “Configurar limites”.
2. O formulário mostra temperatura mínima/máxima em °C e umidade mínima/máxima em %.
3. Validar presença dos quatro valores, mínimo menor que máximo e umidade dentro de 0–100. Não impor uma faixa agronômica.
4. Enviar atualização com versão/ETag para evitar sobrescrever silenciosamente alteração concorrente.
5. A API persiste uma nova versão e avalia a leitura mais recente com os limites atualizados.
6. Atualizar estados e alertas; informar sucesso apenas após confirmação da API.

Os números exibidos no protótipo (por exemplo, 18–30 °C e 50–70%) são placeholders visuais, não defaults agronômicos aprovados. Antes de produção, definir o comportamento de primeira execução: solicitar que o operador configure limites ou apresentar valores iniciais explícitos e editáveis validados com orientação competente.

### 5.3 Ingerir e classificar medições

1. Receber um payload da fonte simulada ou do adaptador autenticado.
2. Validar identidade da estufa/sensor, variável, unidade, valor, timestamp e identificador idempotente.
3. Normalizar unidades e timestamp UTC; rejeitar ou colocar em quarentena amostras inválidas, sem corrigi-las silenciosamente.
4. Persistir a amostra com proveniência (`simulated` ou `sensor`).
5. Avaliar os limites vigentes e atualizar o ciclo de alerta.
6. Atualizar o painel na próxima consulta, dentro da janela horária do MVP.

### 5.4 Consultar histórico

- Períodos permitidos: últimas 24 horas, 7 dias, 30 dias, 90 dias e 1 ano.
- API limita datas máximas e devolve timestamp, valores, unidades e origem.
- Para 24 horas/7 dias, retornar amostras horárias. Para períodos extensos, agregação diária ou semanal é preferível para desempenho e legibilidade; registrar na resposta a granularidade efetivamente devolvida.
- Disponibilizar tabela acessível com horário, temperatura, umidade e origem. A legenda do gráfico não deve ser a única forma de identificar séries.
- O gráfico deve apresentar separadamente as unidades/escala de temperatura e umidade, e tooltips/consultas devem estar disponíveis sem depender exclusivamente de hover.

## 6. Modelo de dados

Todos os timestamps são armazenados em UTC e convertidos para o fuso local configurado na interface. Toda tabela de domínio inclui `created_at`/`updated_at` quando pertinente.

### `greenhouse`

| Campo | Tipo sugerido | Descrição |
|---|---|---|
| `id` | UUID, PK | Identificador estável. |
| `name` | text | Nome apresentado no cabeçalho. |
| `timezone` | text | Fuso IANA da estufa, a confirmar. |
| `created_at` | timestamptz | Criação do cadastro. |

MVP comporta uma estufa, mas manter a relação explícita facilita vincular medições e evita dados órfãos.

### `measurement`

| Campo | Tipo sugerido | Descrição |
|---|---|---|
| `id` | UUID, PK | Identificador interno. |
| `greenhouse_id` | UUID, FK | Estufa de origem. |
| `metric` | enum/text controlado | `air_temperature` ou `relative_humidity`. |
| `value` | numeric | Valor normalizado. |
| `unit` | enum/text | `Cel` ou `%RH` (definir formato final). |
| `measured_at` | timestamptz | Instante da coleta informado pela fonte. |
| `received_at` | timestamptz | Instante da recepção pelo backend. |
| `source` | enum | `simulated` ou `sensor`. |
| `source_id` | text nullable | Identificador do sensor/gateway, quando conhecido. |
| `idempotency_key` | text | Identificador único da amostra na origem. |
| `quality` | enum | `valid`, `suspect` ou `invalid`; política de qualidade será refinada na integração. |

Restrições/índices: unicidade por `(source, source_id, idempotency_key)` quando `source_id` existir; índice `(greenhouse_id, metric, measured_at DESC)`; rejeitar timestamps excessivamente futuros e payload duplicado. Política para relógios incorretos no gateway a definir.

### `threshold_set` e `threshold_version`

Guardar limites editáveis por variável com `min_value`, `max_value`, `unit`, `valid_from`, autor e versão. Uma nova edição cria uma versão em vez de apagar o histórico anterior. Assim é possível explicar qual limite causou um alerta histórico.

### `alert`

Guardar `id`, estufa, métrica, estado (`active`, `resolved`), primeira amostra que disparou, leitura e limite que dispararam, horários de abertura/encerramento, versão de limite aplicada e origem da leitura. Um alerta representa um episódio contínuo fora da faixa; amostras subsequentes do mesmo episódio atualizam `last_observed_at`/valor, evitando criar um registro por hora.

### `user` e `audit_event`

Um usuário administrador no MVP; armazenar hash de senha por algoritmo apropriado, nunca senha em texto claro. Registrar alterações de limites e operações administrativas mínimas com usuário, instante e valores anterior/novo. Não registrar credenciais ou tokens nos logs.

## 7. Regras de alerta

1. Uma amostra válida viola a faixa quando `value < min_value` ou `value > max_value`. Igualdade ao limite conta como dentro da faixa, salvo decisão futura em contrário.
2. Ao sair da faixa, abrir um alerta se não houver episódio ativo para a variável/estufa; associar a amostra e a versão de limites.
3. Durante o mesmo episódio, atualizar a observação mais recente sem criar alertas duplicados.
4. Ao voltar à faixa, resolver o alerta ativo e registrar a amostra que encerrou o episódio.
5. Na mudança dos limites, reavaliar a leitura mais recente. Se isso abrir ou resolver alerta sem nova medição, registrar que a causa foi alteração de configuração.
6. Alerta resolvido permanece no histórico até a política de retenção definida; não é um comando para corrigir a estufa.
7. A tela de estado vazio exibe “Nenhum alerta ativo” em vez de sumir com o contexto da seção.
8. Não gerar alertas por amostras duplicadas, inválidas ou exclusivamente simuladas em ambiente de produção. O modo de simulação deve ser separado/configurado por ambiente e marcado na interface.

Este ciclo é uma proposta para resolver a regra de repetição/encerramento que o PRD deixou em aberto; confirmar com o usuário antes de produção.

## 8. Contrato da API

Prefixo `/api/v1`; JSON UTF-8; HTTPS em todos os ambientes remotos; autenticação obrigatória exceto endpoints explicitamente públicos de saúde sem informação sensível. Erros seguem um formato consistente, com `code`, mensagem segura e identificador de correlação.

### `GET /api/v1/dashboard`

Retorna snapshot para o primeiro carregamento:

```json
{
  "greenhouse": {"id": "uuid", "name": "Estufa principal", "timezone": "America/Sao_Paulo"},
  "generated_at": "2026-10-13T12:00:00Z",
  "data_source": "simulated",
  "measurements": {
    "air_temperature": {
      "value": 27.4, "unit": "Cel",
      "measured_at": "2026-10-13T12:00:00Z",
      "received_at": "2026-10-13T12:00:01Z",
      "freshness": "current"
    },
    "relative_humidity": {
      "value": 71, "unit": "%RH",
      "measured_at": "2026-10-13T12:00:00Z",
      "received_at": "2026-10-13T12:00:01Z",
      "freshness": "current"
    }
  },
  "thresholds": {
    "version": 3,
    "air_temperature": {"min": 18, "max": 30, "unit": "Cel"},
    "relative_humidity": {"min": 50, "max": 70, "unit": "%RH"}
  },
  "active_alerts": []
}
```

Os exemplos são de formato; valores são demonstrativos e não devem virar recomendação agronômica.

### `GET /api/v1/measurements`

Parâmetros: `metric`, `from`, `to`, `interval` (`raw`, `hour`, `day`, `week`), paginação limitada. Validar ordem das datas, máximo de um ano e tamanho de resposta. Retornar `items`, granularidade e metadados de paginação.

### `GET /api/v1/thresholds`

Devolve limites vigentes e versão.

### `PUT /api/v1/thresholds`

Recebe os limites completos e versão esperada. Requer `If-Match`/ETag ou campo `expected_version`; conflito devolve `409 Conflict`. Validar unidade, finitude dos números, mínimo < máximo e domínio 0–100 para umidade.

### `GET /api/v1/alerts`

Filtros: `status=active|resolved`, período e paginação. Ordenação por abertura descendente. Não inferir resolução no cliente.

### Ingestão de amostras

Preferir endpoint separado e credencial máquina-a-máquina, por exemplo `POST /api/v1/ingestion/readings`, que valida chave idempotente e autorização específica da fonte. No desenvolvimento, dados simulados podem ser inseridos por tarefa autenticada/seed, sem endpoint público para forjar telemetria. A forma de autenticação do gateway será escolhida após levantamento de hardware.

### Erros HTTP esperados

- `400`: filtro ou payload malformado.
- `401`/`403`: sessão ausente ou operação não autorizada.
- `404`: recurso inexistente.
- `409`: versão de limites desatualizada ou chave idempotente incompatível.
- `422`: valores/unidades semanticamente inválidos.
- `429`: limite de requisições excedido, se necessário na implantação.
- `5xx`: erro interno com identificador de suporte, sem detalhe sensível na resposta.

## 9. Estratégia de simulação e integração futura

### MVP demonstrativo

- Implementar `ReadingProvider`/serviço equivalente com dados repetíveis e relógio controlável nos testes.
- Criar uma semente reproduzível de pelo menos 24 horas e uma amostra que demonstre estado dentro e fora do limite.
- Marcar todos os registros como `simulated` no banco e na API.
- Separar o modo de demonstração do ambiente de produção por configuração explícita; o servidor não deve recorrer silenciosamente ao simulador quando sensor/API falhar.
- O protótipo visual atual usa um exemplo fixo; integrá-lo deve substituir dados de marcação por resposta da API, sem manter leituras duplicadas no frontend.

### Futura integração física

1. Inventariar fabricante, modelo, tipo de sensor, intervalo recomendado, unidade, calibração, interface e documentação.
2. Confirmar se os sensores medem as variáveis requeridas e se os valores podem ser lidos sem comandos de escrita.
3. Criar adaptador somente de leitura no gateway e normalizar unidades/timestamps.
4. Persistir fila local durável antes de enviar, com identificadores idempotentes.
5. Enviar lotes com backoff exponencial limitado e jitter; após receber confirmação, remover somente os itens aceitos.
6. Registrar estado de conexão e tamanho da fila, sem expor credenciais no painel.
7. Testar desligamento/retomada, duplicidade, perda de rede e dados fora de ordem.
8. Não permitir escrita em registradores ou publicação de comandos ao equipamento no MVP; idealmente, credencial e política de rede só autorizam leitura.

## 10. Segurança, privacidade e operação

- Usar HTTPS e cookies de sessão `HttpOnly`, `Secure` e `SameSite`; adotar proteção CSRF se a autenticação for por cookie.
- Armazenar senhas com algoritmo dedicado de hashing, como Argon2id ou bcrypt, via biblioteca mantida; não inventar criptografia própria.
- Aplicar rate limiting em login e proteger recuperação/alteração de credenciais.
- Provisionar o único usuário admin por comando seguro; não entregar senha padrão imutável no código.
- Validar autenticação/autorização também no backend; ocultar botão na interface não é controle de acesso.
- Segmentar rede do gateway e dos dispositivos; impedir conexão de entrada da internet para sensores/controladores.
- Guardar segredos em variáveis/secret manager fora do repositório; rotacionar credenciais da integração.
- Manter logs com timestamp, severidade, request ID e eventos operacionais necessários, sem tokens, senhas ou dados pessoais desnecessários.
- Backup automatizado e teste documentado de restauração do PostgreSQL. Definir RPO/RTO com o responsável pela operação.
- Atualizar dependências e imagens base, restringir portas e aplicar patches no host/serviços.
- Minimizar dados pessoais. Avaliar aplicabilidade do RGPD e requisitos legais conforme hospedagem e contexto organizacional; este documento não constitui parecer jurídico.

## 11. Acessibilidade e qualidade visual

Critério-alvo recomendado: WCAG 2.2 nível AA.

- Estrutura semântica com cabeçalhos hierárquicos, regiões nomeadas e botão de pular conteúdo.
- Todos os controles operáveis por teclado; foco claramente visível e não encoberto; nenhum foco movido ao atualizar leituras.
- Dialog de limites com foco inicial previsível, fechamento por botão/Escape, retorno do foco ao acionador, labels associados, mensagem de erro relacionada aos campos e submissão clara.
- Estado do alerta comunicado por texto e ícone, não apenas vermelho/verde; mudanças de estado não devem sequestrar foco nem causar anúncios excessivos.
- Gráfico com título, descrição, legenda/unidades e uma representação de tabela consultável por teclado/leitor de tela.
- Contraste AA para texto e controles, tamanho de alvo apropriado, layout sem perda de informação em largura móvel e zoom.
- Reduzir ou remover animação quando `prefers-reduced-motion` está ativo.
- Testar em 320 CSS px, ampliação, navegação só por teclado e leitor de tela. Fazer verificação automatizada e revisão manual; automação sozinha não comprova conformidade.

## 12. Desempenho, retenção e observabilidade

### Metas preliminares a validar

- API de snapshot: p95 até 1 segundo sob carga de uma estufa e uma sessão.
- Consulta de gráfico anual agregada: p95 até 2 segundos.
- Painel indica dados desatualizados após duas janelas horárias sem amostra (proposta).
- Disponibilidade mensal e metas de recuperação ainda precisam ser acordadas; não prometer SLA sem infraestrutura contratada.

### Retenção

- Manter dados consultáveis por até 12 meses, conforme PRD.
- Definir política para descartar amostras/alertas mais antigos e respeitar a retenção operacional aplicável.
- Índices temporais e agregação sob demanda; se volume futuro exigir, acrescentar partições e agregados diários sem alterar o contrato do painel.
- Não apagar um registro bruto ao produzir uma agregação.

### Métricas e alertas de infraestrutura

- Registrar taxa de ingestão por origem, atraso da última amostra, falhas de validação, duplicatas, erros da API e tamanho da fila do gateway quando houver.
- Monitorar espaço em disco, idade do último backup e resultado da última restauração automatizada.
- Alertas operacionais da infraestrutura devem ser distintos dos alertas ambientais e não confundir o operador.

## 13. Estratégia de testes

### Unitários

- Validação de limites e unidades.
- Transições de alerta: faixa normal → violação → persistência do episódio → retorno à faixa → resolução.
- Formatação/localização de valores, timestamps e estado de dado antigo.
- Rejeição de amostra inválida, duplicada e de origem indevida.

### Integração

- Rotas autenticadas, persistência, ordenação temporal e paginação.
- Atualização concorrente de limite/ETag.
- Avaliação de limites ao ingerir uma leitura e ao salvar nova versão de limites.
- Retenção histórica e agregação de períodos.
- Identificação da origem simulada e falhas de API/banco.

### Interface/E2E

- Primeiro carregamento, indicadores dentro/fora da faixa e ausência de dados.
- Ajustar limites, validar valores invertidos e observar atualização do cartão e alerta.
- Períodos históricos, tabela alternativa e mensagem “nenhum alerta ativo”.
- Sessão expirada e falha de rede não devem parecer sucesso nem substituir os dados por zeros.
- Navegação por teclado, foco/diálogo, leitor de tela, responsividade e modo de movimento reduzido.

### Hardware (etapa posterior)

- Testes do adaptador com simulador do protocolo e hardware real em ambiente controlado.
- Queda prolongada da internet, reinício do gateway, sincronização em lote, dados fora de ordem e duplicidade.
- Confirmar que a integração MVP não escreve em registradores nem transmite comando aos atuadores.

## 14. Plano incremental de implementação

| Etapa | Entrega | Critério para avançar |
|---|---|---|
| 0. Decisões e fundação | Confirmar stack, unidades, fuso, defaults/primeira configuração de limites, hospedagem e conta inicial. Preparar repositório e ambiente. | Decisões anotadas e aplicação executável em desenvolvimento. |
| 1. Esqueleto navegável | Implementar o design Essencial em componentes responsivos, com estados carregando, sem dados, falha e amostra simulada identificada. | Painel visual reproduz a hierarquia escolhida e passa navegação básica por teclado. |
| 2. Persistência e API | Esquema, migrações, autenticação e endpoints de dashboard, medições e limites. | Dados persistem e rotas aplicam autorização e validação. |
| 3. Simulação e alertas | Fonte simulada repetível, motor de avaliação e histórico de episódios de alerta. | Critérios de aceite do PRD passam em testes automatizados. |
| 4. Histórico e acessibilidade | Consultas por período, agregação, gráfico, tabela equivalente e revisão responsiva/WCAG. | Histórico anual responde dentro das metas preliminares e alternativa tabular funciona. |
| 5. Demonstração e operação | Implantação HTTPS, backup, logs, documentação de execução/restauração e avaliação do TCC. | Demonstração remota autenticada; recuperação do banco testada. |
| 6. Levantamento e integração física | Inventário de sensores, adaptador somente leitura, gateway com fila local e sincronização. | Hardware compatível identificado; teste de perda de internet e reenvio sem duplicidade aprovado. |

As etapas 0–5 compõem o produto demonstrável com dados simulados. A etapa 6 é posterior e depende de informações do hardware ainda não conhecidas.

## 15. Decisões pendentes antes de produção

1. Confirmar stack e ambiente de implantação aceitos para o TCC.
2. Confirmar unidade de temperatura, timezone da estufa e apresentação de horário.
3. Definir se a configuração inicial exige o operador informar limites ou usa valores demonstrativos explicitamente identificados.
4. Validar ciclo de abertura/encerramento do alerta e comportamento após alteração de limites.
5. Definir política de retenção precisa, exportação, backups, RPO e RTO.
6. Confirmar se a interface deve ser priorizada para computador, celular ou ambos.
7. Identificar marca/modelo, protocolo, intervalo e precisão dos sensores existentes.
8. Definir gateway/armazenamento local compatível com o requisito pós-MVP de sincronização sem internet.
9. Validar a expectativa de alerta horário; caso a operação demande resposta rápida, rever frequência e arquitetura antes de conectar sensores.
10. Verificar obrigações de privacidade, hospedagem e cibersegurança aplicáveis à implantação real.

## 16. Critérios de conclusão técnica do MVP

- Usuário autenticado acessa o painel remoto via HTTPS.
- Painel distingue dados simulados, apresenta leitura, unidade, horário, faixa e estado.
- Usuário consegue configurar limites sem inserir uma faixa mínima maior/igual à máxima; umidade permanece entre 0% e 100%.
- Alertas são abertos, atualizados e encerrados conforme a regra documentada, sem duplicação por cada amostra horária.
- Histórico consultável por até 1 ano, com granularidade identificada e alternativa tabular acessível.
- Estados de sem dados, amostra antiga e indisponibilidade da API são explícitos.
- Nenhum endpoint ou componente permite comando a equipamento físico.
- Testes automatizados das regras críticas, backup/restauração e verificação manual de teclado/responsividade foram realizados.
- Instruções de instalação, configuração segura e demonstração estão disponíveis junto ao projeto.
