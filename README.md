# App Full Stack com Next.js, Stripe e Supabase

O kit inicial tudo-em-um para aplicações SaaS de alto desempenho.

## Recursos

- Gerenciamento seguro de usuários e autenticação com [Supabase](https://supabase.io/docs/guides/auth)
- Ferramentas poderosas de acesso e gerenciamento de dados em cima do PostgreSQL com [Supabase](https://supabase.io/docs/guides/database)
- Integração com [Stripe Checkout](https://stripe.com/docs/payments/checkout) e o [portal do cliente Stripe](https://stripe.com/docs/billing/subscriptions/customer-portal)
- Sincronização automática de planos de preços e status de assinaturas via [webhooks do Stripe](https://stripe.com/docs/webhooks)

## Demonstração

- https://subscription-payments.vercel.app/

[![Screenshot da demonstração](./public/demo.png)](https://subscription-payments.vercel.app/)

## Arquitetura

![Diagrama da arquitetura](./public/architecture_diagram.png)

## Configuração passo a passo

Ao implantar este template, a sequência de passos é importante. Siga os passos abaixo na ordem para configurar tudo corretamente.

### Iniciar Implantação

#### Botão de Deploy da Vercel

[![Deploy com Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fvercel%2Fnextjs-subscription-payments&env=NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY,STRIPE_SECRET_KEY&envDescription=Enter%20your%20Stripe%20API%20keys.&envLink=https%3A%2F%2Fdashboard.stripe.com%2Fapikeys&project-name=nextjs-subscription-payments&repository-name=nextjs-subscription-payments&integration-ids=oac_VqOgBHqhEoFTPzGkPd7L0iH6&external-id=https%3A%2F%2Fgithub.com%2Fvercel%2Fnextjs-subscription-payments%2Ftree%2Fmain)

O Deployment da Vercel criará um novo repositório com este template na sua conta do GitHub e o guiará na criação de um novo projeto Supabase. A [Integração de Deploy do Supabase na Vercel](https://vercel.com/integrations/supabase) configurará as variáveis de ambiente necessárias do Supabase e executará as [migrações SQL](./supabase/migrations/20230530034630_init.sql) para configurar o esquema do banco de dados na sua conta. Você pode inspecionar as tabelas criadas no [Editor de Tabelas](https://app.supabase.com/project/_/editor) do seu projeto.

Se a configuração automática falhar, [crie uma conta no Supabase](https://app.supabase.com/projects) e um novo projeto, se necessário. No seu projeto, navegue até o [Editor SQL](https://app.supabase.com/project/_/sql) e selecione o template "Stripe Subscriptions" na seção de Início Rápido.

### Configurar Autenticação

Siga [este guia](https://supabase.com/docs/guides/auth/social-login/auth-github) para configurar um aplicativo OAuth com o GitHub e configurar o Supabase para usá-lo como provedor de autenticação.

No seu projeto Supabase, navegue até [auth > URL configuration](https://app.supabase.com/project/_/auth/url-configuration) e configure sua URL principal de produção (por exemplo, https://seu-url-deploy.vercel.app) como a URL do site.

Em seguida, nas configurações de implantação da Vercel, adicione uma nova variável de ambiente **Production** chamada `NEXT_PUBLIC_SITE_URL` e defina-a com a mesma URL. Certifique-se de desmarcar os ambientes de preview e development para garantir que as branches de preview e o desenvolvimento local funcionem corretamente.

#### [Opcional] - Configurar redirecionamentos coringas para previews de deploy (não necessário se você instalou via o Botão de Deploy)

Se você implantou este template via o botão "Deploy to Vercel" acima, pode pular esta etapa. A Integração do Supabase na Vercel terá configurado os redirecionamentos coringas para você. Você pode verificar isso indo às configurações de [auth](https://app.supabase.com/project/_/auth/url-configuration) do Supabase, e deve ver uma lista de redirecionamentos sob "Redirect URLs".

Caso contrário, para que os redirecionamentos de autenticação (confirmações de email, links mágicos, provedores OAuth) funcionem corretamente em previews de deploy, navegue até as configurações de [auth](https://app.supabase.com/project/_/auth/url-configuration) e adicione a seguinte URL coringa em "Redirect URLs": `https://*-seu-usuario.vercel.app/**`. Você pode ler mais sobre padrões de redirecionamento coringa na [documentação](https://supabase.com/docs/guides/auth#redirect-urls-and-wildcards).

Se você implantou este template via o botão "Deploy to Vercel" acima, pode pular esta etapa. A Integração do Supabase na Vercel terá executado as migrações do banco de dados para você. Você pode verificar isso indo ao [Editor de Tabelas](https://supabase.com/dashboard/project/_/editor) do seu projeto Supabase, e confirmando que há tabelas com dados de seed.

Caso contrário, navegue até o [Editor SQL](https://supabase.com/dashboard/project/_/sql/new), cole o conteúdo do [arquivo schema.sql do Supabase](./schema.sql) e clique em RUN para inicializar o banco de dados.

#### [Talvez Opcional] - Configurar variáveis de ambiente do Supabase (não necessário se você instalou via o Botão de Deploy)

Se você implantou este template via o botão "Deploy to Vercel" acima, pode pular esta etapa. A Integração do Supabase na Vercel terá configurado suas variáveis de ambiente para você. Você pode verificar isso indo às configurações do seu projeto na Vercel e clicando em 'Environment variables', onde haverá uma lista de variáveis de ambiente com o ícone do Supabase exibido ao lado delas.

Caso contrário, navegue até as [configurações da API](https://app.supabase.com/project/_/settings/api) e cole as chaves da API do projeto nas interfaces de implantação da Vercel. Copie as chaves da API do projeto e cole-as nos campos `NEXT_PUBLIC_SUPABASE_ANON_KEY` e `SUPABASE_SERVICE_ROLE_KEY`, e copie a URL do projeto e cole na Vercel como `NEXT_PUBLIC_SUPABASE_URL`.

Parabéns, isso conclui a configuração do Supabase. Quase lá!

### Configurar o Stripe

Em seguida, precisamos configurar o [Stripe](https://stripe.com/) para lidar com pagamentos de teste. Se você ainda não tem uma conta no Stripe, crie uma agora.

Para os passos a seguir, certifique-se de que o ["Modo de Teste"](https://stripe.com/docs/testing) esteja ativado.

#### Criar um Webhook

Precisamos criar um webhook na seção `Developers` do Stripe. Como mostrado no diagrama de arquitetura acima, este webhook é a peça que conecta o Stripe às suas Funções Serverless da Vercel.

1. Clique no botão "Add Endpoint" na [página de Endpoints de teste](https://dashboard.stripe.com/test/webhooks).
2. Insira a URL de implantação de produção seguida por `/api/webhooks` para a URL do endpoint. (por exemplo, `https://seu-url-deploy.vercel.app/api/webhooks`)
3. Clique em `Select events` sob o título `Select events to listen to`.
4. Clique em `Select all events` na seção `Select events to send`.
5. Copie `Signing secret` pois precisaremos dele na próxima etapa (por exemplo, `whsec_xxx`) (/!\ tome cuidado para não copiar o id do webhook we_xxxx).
6. Além das variáveis `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` e `STRIPE_SECRET_KEY` que configuramos anteriormente durante a implantação, precisamos adicionar o segredo do webhook como variável de ambiente `STRIPE_WEBHOOK_SECRET`.

#### Reimplantar com novas variáveis de ambiente

Para que as novas variáveis de ambiente entrem em vigor e tudo funcione corretamente, precisamos reimplantar nosso app na Vercel. No seu Dashboard da Vercel, navegue até deploys, clique no botão de menu de transbordo e selecione "Redeploy" (NÃO habilite a opção "Use existing Build Cache"). Uma vez que a Vercel tenha reconstruído e reimplantado seu app, você está pronto para configurar seus produtos e preços.

#### Criar informações de produto e preços

O webhook do seu aplicativo escuta por atualizações de produtos no Stripe e as propaga automaticamente para o banco de dados do Supabase. Então, com seu listener de webhook em execução, você pode criar suas informações de produto e preços no [Dashboard do Stripe](https://dashboard.stripe.com/test/products).

O Stripe Checkout atualmente suporta preços que cobram um valor predefinido em um intervalo específico. Planos mais complexos (por exemplo, diferentes níveis de preços ou assentos) ainda não são suportados.

Por exemplo, você pode criar modelos de negócios com diferentes níveis de preços, como:

- Produto 1: Hobby
    - Preço

1: $5/mês
- Preço 2: $50/ano
- Produto 2: Profissional
    - Preço 1: $15/mês
    - Preço 2: $150/ano

Os webhooks criados na etapa anterior preencherão automaticamente os preços no banco de dados do Supabase, que nosso aplicativo consome para renderizar a interface do usuário. Ao visitar [localhost:3000](http://localhost:3000) (ou o URL do seu site implantado), você verá a interface de login e as tabelas de preços renderizadas com os dados do produto que você acabou de criar.

### Próximos passos

Parabéns! Se você seguiu todos os passos corretamente, seu starter de assinatura deve estar agora implantado e pronto para uso! Agora você pode permitir que os usuários façam login, inscrevam-se para assinaturas e gerenciem suas assinaturas.

## Desenvolver localmente

As conexões do Supabase [Auth](https://supabase.com/docs/guides/auth) e [Database](https://supabase.com/docs/guides/database) dependem das variáveis de ambiente configuradas no seu ambiente local. Você pode criar um arquivo `.env.local` com base no exemplo fornecido no repositório. Para rodar o app localmente, você precisará definir as seguintes variáveis de ambiente do seu Dashboard do Supabase e do Stripe.

Para o Supabase:

```bash
NEXT_PUBLIC_SUPABASE_URL=https://SEU_PROJETO_ID.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=sua-chave-anon
SUPABASE_SERVICE_ROLE_KEY=sua-chave-de-função-de-serviço
```

Para o Stripe:

```bash
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_sua-chave-publicável
STRIPE_SECRET_KEY=sk_test_sua-chave-secreta
STRIPE_WEBHOOK_SECRET=whsec_xxx
```

Instale as dependências e inicie a aplicação Next.js:

```bash
npm install
npm run dev
```

Sua aplicação deve estar rodando em [localhost:3000](http://localhost:3000).
