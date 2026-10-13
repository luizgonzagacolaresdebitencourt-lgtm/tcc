# PRD — Sistema Web de Gerenciamento de Estufa Inteligente

**Status:** rascunho inicial  
**Versão:** 0.1  
**Data:** 13/10/2026

## 1. Visão do produto

Desenvolver uma aplicação web para acompanhar as condições ambientais de uma estufa de cultivo de tomate e pimentão. A primeira versão deve permitir consultar temperatura e umidade do ar, visualizar o histórico e receber alertas quando os limites configurados pelo operador forem ultrapassados.

O MVP é de **monitoramento e alerta**. Não deve comandar válvulas, ventilação, bombas ou qualquer outro equipamento, nem apresentar recomendações agronômicas.

## 2. Contexto e problema

O operador precisa conhecer as condições da estufa e identificar períodos em que temperatura ou umidade do ar saem das faixas que ele definiu. O sistema deverá reunir leituras atuais e anteriores em um único lugar, acessível pela internet.

Como o hardware ainda não foi identificado, o protótipo poderá utilizar dados simulados. A integração com sensores físicos será uma etapa posterior, precedida por levantamento de marcas, modelos e meios de comunicação disponíveis.

## 3. Objetivos e indicadores de sucesso

### Objetivos do MVP

1. Exibir as leituras mais recentes de temperatura e umidade do ar.
2. Permitir configurar limites inferiores e superiores para cada variável.
3. Sinalizar no sistema web quando uma leitura ultrapassar um limite.
4. Exibir gráficos e permitir consultar até um ano de histórico.
5. Permitir acesso remoto autenticado pela internet.
6. Diferenciar claramente dados simulados de dados provenientes de sensores reais.

### Indicadores de sucesso

- O operador consegue identificar, no painel, a leitura mais recente e seu horário.
- Uma leitura simulada fora do limite configurado produz um alerta visível no sistema.
- O operador consegue consultar leituras históricas e visualizar tendências em gráficos.
- Os dados simulados são identificados como simulados e não são confundidos com medições reais.

Metas quantitativas de disponibilidade, desempenho e adoção ainda precisam ser definidas.

## 4. Usuário e contexto de uso

### Usuário inicial

- Um único usuário, que acumula os papéis de administrador e operador.
- Acesso por navegador web, de qualquer lugar pela internet.
- Uso em celular ou computador ainda não definido.

### Ambiente de operação

- Uma única estufa.
- Cultivos considerados: tomate e pimentão.
- Cultivo em solo ou substrato: não definido.
- Acesso à internet esperado para consulta remota. Na futura integração física, a coleta deverá continuar localmente durante uma interrupção e sincronizar os dados quando a conexão voltar.

## 5. Escopo

### Incluído no MVP

- Painel com temperatura e umidade do ar.
- Leituras atualizadas aproximadamente uma vez por hora.
- Indicação do horário da última leitura e do tipo de dado (simulado ou real).
- Configuração pelo operador dos limites mínimo e máximo de cada variável.
- Alertas visíveis na aplicação web quando uma leitura ultrapassar os limites.
- Gráficos e consulta de histórico com retenção de até um ano.
- Acesso remoto protegido por autenticação.
- Fonte simulada de dados para demonstração e validação do fluxo.

### Fora do MVP

- Acionamento ou controle remoto/automático de equipamentos.
- Recomendações de manejo ou faixas agronômicas predefinidas.
- Sensores de umidade do solo/substrato, luminosidade, CO₂, pH, condutividade elétrica ou medição de irrigação.
- Notificações por e-mail, SMS ou WhatsApp.
- Vários usuários, perfis de acesso ou múltiplas estufas.
- Previsão de doenças, visão computacional, inteligência artificial ou otimização de energia.
- Integração com ERP, aplicações de terceiros, estações meteorológicas ou controladores até que a necessidade seja confirmada.

## 6. Requisitos funcionais

| ID | Requisito | Prioridade |
|---|---|---|
| RF-01 | O sistema deve apresentar a leitura mais recente de temperatura do ar e sua data/hora de coleta. | MVP |
| RF-02 | O sistema deve apresentar a leitura mais recente de umidade relativa do ar e sua data/hora de coleta. | MVP |
| RF-03 | O sistema deve indicar se os dados apresentados são simulados ou provenientes de sensores físicos. | MVP |
| RF-04 | O operador deve poder configurar limites mínimo e máximo para temperatura e umidade do ar. | MVP |
| RF-05 | Ao receber uma leitura fora dos limites configurados, o sistema deve exibir um alerta dentro da aplicação, identificando a variável, o valor observado, o limite ultrapassado e o horário da leitura. | MVP |
| RF-06 | O sistema deve permitir consultar medições anteriores e exibi-las em gráficos por período, com até um ano de histórico disponível. | MVP |
| RF-07 | O sistema deve atualizar os dados aproximadamente a cada hora. Um atraso de até uma hora para a detecção/visualização do alerta é aceitável no MVP. | MVP |
| RF-08 | O sistema deve permitir acesso pela internet mediante autenticação do usuário. | MVP |
| RF-09 | Em uma etapa posterior com sensores reais, a coleta local deve continuar durante uma falha de internet e sincronizar os registros pendentes após a reconexão. | Pós-MVP |

### Critérios de aceite essenciais

- Dada uma leitura dentro dos limites configurados, o painel apresenta o valor e o horário sem indicar violação.
- Dada uma leitura abaixo do mínimo ou acima do máximo configurado, o sistema apresenta alerta correspondente no ambiente web.
- O operador consegue alterar limites e consultar leituras históricas em gráficos.
- Leituras simuladas são identificadas de forma inequívoca como demonstração.
- No MVP, nenhuma ação de controle é enviada a equipamentos físicos.

## 7. Requisitos não funcionais

- **Acesso e segurança:** o acesso remoto exige autenticação; credenciais e sessões devem ser protegidas. Não expor a aplicação ou dispositivos de campo sem controles de acesso.
- **Integridade dos dados:** cada medição deve estar associada à variável e ao horário de coleta; o sistema deve distinguir ausência de dado de um valor medido.
- **Retenção:** manter o histórico consultável por até um ano. Política de descarte, exportação e cópia de segurança precisa ser definida na implementação.
- **Resiliência futura:** a integração física deve prever armazenamento local temporário e sincronização após reconexão, evitando duplicidade de registros.
- **Usabilidade:** o painel deve deixar claros o valor atual, o horário, a unidade e o estado de alerta.
- **Privacidade:** coletar apenas os dados pessoais necessários para autenticação e operação. A aplicabilidade das obrigações legais dependerá do contexto de implantação.

## 8. Dados e regras

- **Temperatura do ar:** unidade de exibição ainda a confirmar; °C é a opção esperada para validar.
- **Umidade relativa do ar:** expressa em porcentagem (%).
- **Limites:** definidos pelo operador; este documento não prescreve limites agronômicos para tomate ou pimentão.
- **Frequência:** leitura/atualização aproximadamente horária no MVP.
- **Histórico:** até 12 meses de consulta.
- **Alertas:** somente dentro do sistema web nesta fase. Regras para repetição, encerramento e reconhecimento do alerta ainda precisam ser detalhadas.

## 9. Hipóteses, dependências e riscos

- A primeira demonstração usará dados simulados; esses dados não devem ser apresentados como provenientes de sensores reais.
- Marca, modelo, disponibilidade de interface e protocolo dos equipamentos existentes são desconhecidos. A integração não pode ser especificada com segurança antes desse levantamento.
- O armazenamento e a sincronização locais dependem de haver um equipamento local ou gateway compatível na futura instalação.
- A atualização horária foi considerada suficiente e o atraso potencial de até uma hora para alertas foi aceito para o MVP. Isso não atende a situações que exijam resposta imediata.
- A adequação das faixas configuradas pelo usuário é responsabilidade operacional/agronômica e não será validada pelo software no MVP.
- A disponibilidade de internet é necessária para acesso remoto; o comportamento da interface durante indisponibilidade e a frequência de sincronização posterior devem ser definidos na etapa de integração.

## 10. Questões em aberto

1. O cultivo será feito em solo ou substrato?
2. Quais são as marcas, modelos e interfaces dos sensores/equipamentos existentes?
3. O sistema deve funcionar bem em celular, além do computador?
4. Qual unidade de temperatura deve ser usada na interface?
5. Como alertas devem se comportar quando a leitura volta à faixa: encerrar automaticamente, permanecer no histórico ou exigir reconhecimento?
6. O operador precisa exportar os dados (por exemplo, CSV) ou receber relatórios?
7. Quais metas de disponibilidade, desempenho e recuperação são necessárias?
8. Quais são as exigências de hospedagem, orçamento e manutenção?

## 11. Referências e limites da pesquisa

- [Priva](https://www.priva.com/) e [Ridder](https://www.ridder.com/): materiais de fornecedores sobre tecnologia e controle climático para estufas. São referências setoriais, não prova de que todas as funções sejam universais.
- [OPC UA](https://opcfoundation.org/about/opc-technologies/opc-ua/), [MQTT](https://mqtt.org/) e [Modbus](https://modbus.org/): referências de protocolos que podem apoiar integrações, sujeitas à compatibilidade do hardware selecionado.
- [Regulamento Geral sobre a Proteção de Dados (RGPD)](https://eur-lex.europa.eu/eli/reg/2016/679/oj), [Diretiva NIS2](https://eur-lex.europa.eu/eli/dir/2022/2555/oj) e [CNPD](https://www.cnpd.pt/): referências para avaliação de privacidade e cibersegurança conforme o contexto da organização e da implantação.
- [Controlled Environment Agriculture Center — University of Arizona](https://ceac.arizona.edu/): contexto institucional sobre agricultura em ambiente controlado.

As funcionalidades priorizadas neste documento refletem as decisões do usuário para o produto; as referências contextualizam o domínio, mas não substituem validação com o operador, orientação agronômica ou levantamento técnico dos equipamentos.
