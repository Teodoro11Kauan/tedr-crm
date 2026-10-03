# TEDR CRM — versão Firebase (Auth + Firestore + Hosting, plano gratuito Spark)

Este código não foi testado contra o projeto Firebase de verdade (o ambiente aqui não tem acesso à internet do Google). Teste com calma antes de confiar os dados a ele.

## 1. Preparar o projeto no console do Firebase (console.firebase.google.com, projeto "tedr-crm")

1. **Authentication → Sign-in method:** ative "E-mail/senha".
2. **Firestore Database → Create database:** modo produção, região mais próxima (ex.: southamerica-east1).
3. **Firestore → Rules:** cole o conteúdo de `firestore.rules` (ou rode `firebase deploy --only firestore:rules`, abaixo) e publique.

O próprio site cria a sua conta: abra o link publicado, clique em "Criar conta" e informe seu e-mail e senha. Depois disso, use "Entrar" normalmente. Dentro do app, em Configurações → Usuários, você cria outros e-mails com acesso (por exemplo, um só para o n8n usar) sem perder sua sessão.

## 2. Publicar (Hosting)

1. Instale o Node.js (nodejs.org) e o Firebase CLI: `npm install -g firebase-tools`
2. Na pasta do projeto: `firebase login`
3. `firebase deploy` (publica o site e as regras)
4. O endereço final aparece no fim, algo como `https://tedr-crm.web.app`

Sem instalar nada, também dá pra abrir o `public/index.html` direto no navegador — ele já aponta para o projeto "tedr-crm" e funciona, só não fica com um link público.

## O que mudou em relação à versão anterior

- **Sem servidor próprio.** O navegador fala direto com o Firebase (Auth e Firestore). Não tem mais `server.py`, chave de API fixa nem `python server.py` para lembrar de manter ligado.
- **Login:** a própria tela tem "Entrar" e "Criar conta". Qualquer pessoa com o link pode criar uma conta — isso não expõe os SEUS dados, porque as regras do Firestore isolam cada usuário nos próprios documentos (uma conta nova só vê uma área vazia dela mesma), mas é algo a saber. Dentro de Configurações → Usuários, você cria outros e-mails com acesso sem perder a sua sessão (por exemplo, um usuário separado para o n8n).
- **Vários funis:** continuam, agora como coleções do Firestore. Editar o funil (nome, etapas, cor, ordem) também continua, na aba Configurações.
- **Webhooks — a diferença mais importante:** como não há mais servidor, o próprio navegador dispara o `POST` para o n8n, e só enquanto a aba do CRM estiver aberta em algum aparelho. Fechou a aba, os eventos não disparam mais até você abrir de novo. Isso é uma limitação real do plano gratuito sem Cloud Functions — veja abaixo.
- **API para o n8n:** não existe mais uma chave fixa. O n8n autentica como um usuário (e-mail e senha) e fala direto com a API REST do Firestore. Passo a passo e exemplos prontos estão na aba "n8n / API" das Configurações, dentro do próprio CRM (peça para eu copiar aqui se preferir).

## Se quiser webhooks de verdade (mesmo com a aba fechada)

Isso exige Cloud Functions, e o Firebase pede que o projeto esteja no plano Blaze (pago por uso) para habilitar Functions — mesmo assim, o uso continua saindo de graça dentro da cota gratuita generosa, e você só paga se passar muito dela. Se quiser, eu escrevo a função (dispara ao mudar `stage_id` num negócio) depois que você decidir se quer ativar o Blaze.

## Segurança

- As regras do Firestore restringem cada usuário aos próprios dados (`/crm/{uid}/...`). Sem estar autenticado, nada é lido nem gravado.
- A `apiKey` do Firebase que aparece no código não é secreta — ela só identifica o projeto; quem protege os dados são as regras do Firestore e o Authentication.
- Publique as regras (`firebase deploy --only firestore:rules`) antes de guardar qualquer dado real — com o banco criado em "modo teste", qualquer pessoa lê e escreve.
