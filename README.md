# aluragente-challenge PetLar (@Petlarshop_bot)- Agente de IA chatbot de telegram para atendimento ao cliente via telegram
O workflow foi criado no N8N com o intuito de simular atendimento ao cliente pelo telegram para um pet shop chamado PetLar. O nome do agente é PetLar user @Petlarshop_bot. Conforme a proposta do desafio utilizei um pdf com informações desse pet shop para que o agente consultasse o documento e fizesse o RAG para responder perguntas do usuário. Foi utilizado o Claude Code AI para criar esse documento fictício de um pet shop com informações fictícias para a resolução do desafio.

O QUE O WORKFLOW FAZ:
- Recebe mensagens de usuários no Telegram.
- Processa a pergunta do usuário.
- Mantém o histórico da conversa.
- Consulta a base de conhecimento usando busca vetorial.
- Consulta uma tabela do Supabase para verificar informações pela mensagem que o usuário (cliente) manda (endereço, horários de funcionamento e dias, produtos em estoque, telefone para contato, quais serviços oferece e etc).
- Responde no Telegram após analisar a pergunta e devolve após a consulta desses dados.
OBS: Uma das regras principais de negócio é não inventar informações, valores, endereço, nada que não esteja na base de dados, nada que não esteja no Prompt ou na tabela do Supabase. Além disso, nada de entregas em domicílio de produtos, nada de passar a chamada para um atendente, isso sai da proposta do desafio. O agente atende, ele conduz o atendimento e consulta o documento para passar essas informações para o cliente/pessoa usuária.

ARQUITETURA:

.Resposta no Telegram:
-Telegram Trigger → AI Agent → Send a text message.
-AI Agente (nós): Mistral Cloud Chat Model → Simple Memory → Supabase Vector Store + Embeddings Mistral Cloud.

.Automação para o RAG no Supabase:
-Google Drive trigger → Download File (Google Drive) → Extract from file → Supabase Vector Store.
-Supabase Vector Store (nós): Embeddings Mistral Cloud + Default Data Loader.

Utilizando o Supabase como Vector store (banco de dados) do agente.
Supabase foi utilizado para inserir o documento para o agente IA utilizar na base de conhecimento dele. A IA consulta o documento e responde o usuário.
Criar conta no Supabase (caso não tenha), criar uma tabela em "SQL Editor" e você vai precisar do código para criar a tabela que você consegue dentro do N8N procurando pelo nó do supabase vector store e na opção "add documents to vector store" selecione docs, vai te levar para uma página de instruções do supabase do N8N que você pode ler se for do seu interesse mas para especificamente pegar o código selecione "quickstart for setting up your vector store". É o primeiro código que aparece, você pode simplesmente copiar e colar o código no "SQL Editor" e selecionar o RUN para o código ser lido. Pode conferir a tabela criada em "Table editor" usando a barra lateral geralmente vem escrito "documents" que é o nome padrão.
Inserindo o documento no Google Drive e fazendo automação para quando o documento for inserido no Google Drive ele ser inserido no Supabase. Selecionei o gatilho do Google Drive de mudanças em um documento específico e depois selecionei o nome da pasta nova que estou usando para colocar o documento e o gatilho será ativado toda vez que houver uma alteração na pasta que eu escolhi nos parâmetros. Você vai precisar da api do Google Drive/Google cloud caso não tenha conectado ao N8N com suas credenciais. Você também deve mudar os parâmetros dentro do nó para changes involving a specific folder no trigger e depois watch file created.
Traduzindo: o gatilho será ativado sempre que houver uma mudança na pasta que você escolheu da sua conta e essa mudança deve ser um arquivo criado, quando for criado será enviado para o Supabase.
Essa automação foi feita para tornar essa parte do RAG mais política.
Como estou utilizando um documento em pdf para fazer o RAG, utilizarei markdown para esse pdf performar melhor na base de conhecimento do agente. Pedi ao chatgpt para fazer o markdown do documento.
Faça download do documento com markdown, depois abra na pasta nova do Google Drive que criamos nos passos anteriores, você vai executar o gatilho após colocar o documento lá, depois você seleciona um nó do Google Drive de Download de arquivo ou "download file" para você fazer download desse documento dentro do N8N, você vai conectar esse nó com o gatilho do Google Drive, nos parâmetros em "file" selecione "by id", em seguida procure o id no output do documento, está nos códigos do output do documento que colocamos no Google Drive e que nós executamos no gatilho.
Em "by id" você precisa encontrar a variável id e arrastar ele até esse campo, nós queremos baixar o documento dentro do Google Drive pelo id dele. Ao encontrar o id execute o workflow para verificar se está tudo certo.
O que nós fizemos foi pegar esse arquivo e baixar para o N8N conseguir manipular. O próximo passo é pegar esse arquivo com informações e extrair, transformar em texto para enviar para o Supabase. 
Ao lado do nosso nó de "download file" do Google Drive selecione o nó "extract from file" e "extract from pdf" já que estamos utilizando um arquivo em pdf e execute o workflow, pelo certo o nosso pdf vai estar lá e tem uma variável no nosso output nesse nó chamado "text" que é onde terá o texto do nosso pdf em markdown.
Agora você pode conectar esse módulo extract from file no Supabase e você vai precisar de embeddings, vamos utilizar embedding da Open AI que também precisa de chave de API nas credenciais do N8N caso você não tenha e é bem simples de criar uma e colocar no N8N.
No outro nó "document" selecione "Default Data loader", clique no nó e selecione o modo "load specific data", você vai agora no output e achar a variável "text" e arrastar para o "data" para subir o texto presente no nosso pdf com markdown.
No nosso Supabase vector store, você vai criar suas credenciais caso não tenha para conectar na sua conta, você também vai precisar de uma API mas não precisa criar, se criou a conta a chave de API e o Host para colocar nas credenciais do N8N já está na sua conta. Após colocar suas credenciais selecione a tabela "documents" que criamos no supabase para esse projeto.
OBS: Executei o workflow e deu problemas nos embeddings da OpenAI, acusou falta de créditos e como eu não pretendo investir nisso agora, troquei pelos embeddings Mistral Cloud que também servem bem, mesmo esquema você vai precisar colocar as suas credenciais caso não tenha e vai precisar criar uma conta na Mistral Cloud para fazer suas credenciais e criar uma chave de API.
Agora vamos para o nosso Agente de IA, irei usar o gatilho de chat comum "When message received" para fazer os primeiros testes antes de conectar ao telegram, pretendo testar como o agente responde antes de fazer o chatbot. Em seguida, vamos conectar o gatilho de chat no AI agent, o modelo de chat da Mistral Cloud, modelo de memória simples, a tool vai ser o nosso Supabase Vector Store (lembre-se de selecionar a tabela documents) e o nosso embeddings da Mistral Cloud. 
Fazendo perguntas ao agente no chat do N8N sobre horários de funcionamento e endereço obtive as respostas corretas, agora vou conectar ele ao telegram.
Conectando ao telegram, excluí o nó de gatilho e procurei pelo telegram no gatilho "on message", depois conectei no agente e no outro nó do lado direito do agente coloquei um nó do telegram "send message" que vai ser o da resposta do chatbot. Você vai precisar das suas credenciais nesse gatilho também, selecione o gatilho do telegram e ao colocar suas credenciais novas você vai precisar do "open docs", lá você vai se deparar com o passo a passo e nele tem o BotFather que é um Bot o telegram para criar Bots, abra o BotFather e digite o comando /newbot, dê um nome para seu Bot e depois um user para ele que deve terminar com _bot. Após isso, o BotFather vai te providenciar a API para conectar ao N8N. 
Volte ao BotFather e na mesma mensagem que ele te passa sua API para o N8N você vai ter o user do seu Bot para clicar nele e inicar com o comando /start, você também pode manda umas mensagens para ele para testar e confira no seu N8N se as mensagens estão aparecendo no nó de gatilho. O próximo passo foi voltar no meu nó agente de ia e terminar de configurar, em "source for prompt" troquei por "define bellow" e em "prompt" arrastei a variável "text" para o campo "prompt". Lembrando que essas variáveis apareceram porque eu iniciei o meu Bot e testei ele mandando mensagens.
No nó do telegram "send message" para o agente mandar mensagem, configurei arrastando a variável "chat id" arrastando do input do gatilho e depois o text é a variável do texto que o agente responde, voltei ao nó "send message" e lá apareceu a variável "output" que era a mensagem da IA e arrastei até text.
Também notei que não tinha configurado o prompt dentro do Agente de IA para ele fazer seu papel encorporando o chatbot, fiz meu prompt no chatgpt para isso e troquei a parte de prompt para "define bellow", depois no campo de prompt abaixo arrastei a variável "text" que estava no input, também na parte options mais abaixo coloquei no "system message" que foi onde inseri o prompt para o agente incorporar um atendente no telegram, errei nessa parte e o meu agente antes não estava ativando o banco de dados do Supabase Vector Store para ler o pdf de pet shop e fazer o RAG. 


APIs E CREDENCIAIS NECESSÁRIAS

- Google Drive/Google cloud (serve para Google Drive, Google Sheets, Google Agenda, aplicações do Google).
- Supabase Vector Store.
- Telegram.
- Mistral Cloud.


STACK UTILIZADA

- Claude Code para produzir o pdf fictício
- Chatgpt para converter o pdf em markdown e para produzir o prompt do agente.
- Google Drive para ser mais político quanto ao pdf e o processo e RAG.
- Supabase como banco de dados.
- Mistral cloud para embeddings, modelo de chat.
- Default Data Loader para deixar o texto mais mastigado.
- Extract from file para transformar o pdf em texto.
- Telegram para ser o nosso chat e para fazer o bot com BothFather.


COMO IMPORTAR O PROJETO PARA RODAR LOCAL OU CLOUD

- Acesse seu n8n (local ou cloud)
- Vá em Settings → Import from File (ou arraste o arquivo direto no editor)
- Selecione o arquivo JSON com o workflow.



EXEMPLOS DE PERGUNTAS E RESPOSTAS 

- Pergunta: Quanto custa a areia para gatos ? Resposta: A areia higiênica está por R$ 30,00.
- Pergunta: Vocês realizam atendimento de emergência ? Resposta: A PetLar não realiza atendimento de emergência ou plantão.
- Pergunta: Também quero comprar coleira para gato. Resposta: A coleira está por R$20,00.

  <img width="720" height="1381" alt="Screenshot_20260811-202810_Telegram" src="https://github.com/user-attachments/assets/8ac6b42e-c6ba-4d6b-b4fe-2dcd5138c7dd" />
<img width="720" height="1379" alt="Screenshot_20260811-203010_Telegram" src="https://github.com/user-attachments/assets/72c955b1-23a5-4821-a388-c629b7bb574e" />
<img width="720" height="1381" alt="Screenshot_20260811-202958_Telegram" src="https://github.com/user-attachments/assets/67dbf7ad-8f69-4037-be2a-796980596a7f" />
<img width="720" height="1378" alt="Screenshot_20260811-202946_Telegram" src="https://github.com/user-attachments/assets/e04245a7-d55e-4d62-8e29-0e98029e9bcf" />

EXEMPLOS DO QUE NÃO PERGUNTAR

- Nenhuma pergunta que não seja relacionada aos produtos, serviços, dados de contato, endereço e horário de funcionamento o pet shop PetLar.

<img width="720" height="1384" alt="Screenshot_20260811-204714_Telegram" src="https://github.com/user-attachments/assets/23996018-b7c6-457b-bc20-7288557666c0" />

POR DENTRO DO WORKFLOW DO N8N


<img width="1365" height="719" alt="Captura de tela 2026-08-12 155004" src="https://github.com/user-attachments/assets/ddcb0d2f-d43a-46f0-b413-6501f62b495c" />
<img width="1365" height="606" alt="Captura de tela 2026-08-12 155123" src="https://github.com/user-attachments/assets/318877d4-e8be-4fa6-926f-ccfc66be04f1" />
<img width="1344" height="686" alt="Captura de tela 2026-08-12 155410" src="https://github.com/user-attachments/assets/e66b1800-ee2a-40b2-a5ce-88f270426ccb" />

