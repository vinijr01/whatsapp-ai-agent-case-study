# Leitura técnica da arquitetura

## Caminho principal

`Webhook -> normalização -> filtros -> roteamento por tipo -> mensagem comum ->
verificação de evento próprio/pausa -> buffer Redis -> agente -> divisão da
resposta -> seleção de texto ou mídia -> Evolution API`.

O texto pode chegar como mensagem comum ou texto estendido. No caminho de áudio,
a mídia é obtida pela Evolution API, convertida para arquivo e transcrita por uma
requisição à Groq com o modelo `whisper-large-v3` e idioma `pt`.

O export também inclui caminhos de entrada para análise de imagem e extração de
PDF. Eles não foram destacados como funcionalidades validadas neste case: o
prompt local contém instruções de encaminhamento para algumas dessas entradas,
e sua interação com os caminhos técnicos precisa de teste específico.

## Orquestração e ferramentas

O agente principal está ligado a um modelo `gpt-4o-mini`, a uma memória
PostgreSQL com janela de contexto 20 e às ferramentas de calendário, catálogo,
financeiro e atendimento humano. O agente financeiro possui uma ferramenta de
cálculo. As operações diretas de calendário estão ligadas ao agente especializado.

Um servidor MCP de calendário e suas ferramentas existem em um ramo separado.
O cliente MCP presente no export não está conectado ao orquestrador. Portanto,
ele não foi representado como caminho principal de execução.

## Buffer e pausa

O caminho ligado à entrada usa a sequência de lista Redis, espera, leitura da
lista, comparação da última mensagem, exclusão do buffer e formação do texto.
Há um segundo conjunto de buffer desconectado da entrada, excluído do diagrama
simplificado. A espera principal registra quantidade 12, sem unidade explícita.

O caminho `fromMe` define uma chave de pausa com TTL de 300 segundos e registra a
mensagem na memória. O caminho de mensagens recebidas consulta essa chave e pode
registrar o conteúdo sem chamar novamente o agente.

O export não demonstra por si só que a pausa discrimina mensagens manuais de
mensagens da própria automação; isso depende dos eventos da integração.

## Subworkflow humano

`Chamada do workflow -> normalização -> consulta do histórico PostgreSQL ->
resumo com modelo OpenAI -> notificação à responsável pela Evolution API`.

O aviso contém o resumo e uma referência à conversa. Dados de contato,
servidores e autenticação foram omitidos do material público.
Os nós Redis Chat Memory e Gemini presentes nesse subworkflow não estão ligados
ao caminho executável analisado.

## O que os diagramas representam

- `arquitetura.png/.svg`: visão simplificada das responsabilidades.
- `atendimento-humano.png/.svg`: detalhamento do subworkflow e distinção da pausa.
- `workflow-estrutura.png/.svg`: mapa reconstruído das posições e conexões do
  export principal, sem credenciais, parâmetros ou conteúdo de notas.
- Arquivos `.mmd`: versões editáveis em Mermaid.

O mapa reconstruído não é uma captura da interface n8n nem uma execução.

## Referência para conferir a espera

A documentação do n8n separa quantidade e unidade da espera. No código oficial
consultado, a unidade padrão é `hours`; a versão instalada precisa ser
confirmada antes de executar novamente o export sem unidade explícita.

- [Documentação do nó Wait](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.wait/)
- [Implementação oficial do Wait](https://github.com/n8n-io/n8n/blob/master/packages/nodes-base/nodes/Wait/Wait.node.ts)
