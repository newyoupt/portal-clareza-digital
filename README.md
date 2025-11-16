# 🎯 Portal da Clareza Digital | New You

Sistema de Lead Generation com análise dos **5 Porquês** usando IA do Claude para identificar a raiz dos problemas de presença digital.

## 🚀 Características Principais

### ✨ Portal Público
- **Formulário inicial** - Captura nome, email e Instagram antes de começar
- **8 Desafios Digitais** pré-definidos
- **Perguntas dinâmicas geradas por IA** - Claude API gera perguntas contextuais
- **Timeline visual** com círculos numerados
- **Análise inteligente** - Causa raiz + 3 soluções + insights
- **CTA para consulta gratuita** de 20 minutos
- **Design New You** - Cores da marca + tipografia Poppins

### 🔐 Painel Admin
- **Login protegido** - Password: `1FAIpQLSf4XcCMBLyAvDxTgHv0KRP`
- **Dashboard com estatísticas** - Total leads, agendamentos, novos hoje
- **Gestão de leads** - Visualização completa
- **Gestão de agendamentos** - Com todas as respostas e análise
- **Exportação CSV**
- **Copiar informações** para email de follow-up
- **Auto-refresh** a cada 30 segundos

---

## 💰 Custos

- **Vercel Hosting**: Grátis (até 100GB bandwidth/mês)
- **Anthropic API**: ~$0.003 por análise completa
  - **Estimativa**: 100 utilizadores/mês = ~$0.30
  - **Créditos recomendados**: $5-10 para começar

---

## 📋 Pré-requisitos

### 1. Criar Conta no Vercel
1. Vai a [vercel.com](https://vercel.com)
2. Clica em "Sign Up"
3. Cria conta com GitHub (recomendado)
4. É **100% grátis**

### 2. Obter Chave API do Anthropic
1. Vai a [console.anthropic.com](https://console.anthropic.com)
2. Faz login ou cria conta
3. Vai a **"API Keys"**
4. Clica em **"Create Key"**
5. Copia a chave (começa com `sk-ant-...`)
6. **GUARDA BEM** - só aparece uma vez!

💡 **Nota:** Precisas de adicionar créditos ($5-10) na conta Anthropic para usar a API.

---

## 🚀 Deploy no Vercel

### Opção 1: Via GitHub (Recomendado) ⭐

#### Passo 1: Criar Repositório Git

```bash
# Navega para a pasta do projeto
cd portal-clareza-digital

# Inicializa git
git init
git add .
git commit -m "Initial commit: Portal da Clareza Digital"
git branch -M main
```

#### Passo 2: Criar Repositório no GitHub

1. Vai a [github.com](https://github.com) e faz login
2. Clica no **"+"** (canto superior direito) → **"New repository"**
3. Nome: `portal-clareza-digital`
4. **NÃO** inicializes com README
5. Clica em **"Create repository"**

#### Passo 3: Push para GitHub

```bash
# Conecta ao repositório GitHub (substitui USERNAME pelo teu username)
git remote add origin https://github.com/USERNAME/portal-clareza-digital.git
git push -u origin main
```

#### Passo 4: Deploy no Vercel

1. Vai a [vercel.com](https://vercel.com)
2. Clica em **"New Project"**
3. Clica em **"Import Git Repository"**
4. Seleciona `portal-clareza-digital`
5. Clica em **"Deploy"**
6. Aguarda 1-2 minutos ✅

#### Passo 5: CRÍTICO - Configurar CORS no Vercel

**IMPORTANTE:** O browser bloqueia chamadas diretas à API do Anthropic por questões de CORS. Precisas adicionar headers ao `vercel.json`:

Edita o ficheiro `vercel.json` e substitui pelo seguinte:

```json
{
  "version": 2,
  "routes": [
    {
      "src": "/admin",
      "dest": "/private/index.html"
    },
    {
      "src": "/private/(.*)",
      "dest": "/private/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/public/$1"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "Access-Control-Allow-Origin",
          "value": "*"
        },
        {
          "key": "Access-Control-Allow-Methods",
          "value": "GET, POST, OPTIONS"
        },
        {
          "key": "Access-Control-Allow-Headers",
          "value": "Content-Type"
        }
      ]
    }
  ]
}
```

**ATENÇÃO:** Mesmo com isto, o browser pode bloquear as chamadas à API do Anthropic. Vê a secção "Alternativa com Proxy" abaixo.

---

### Opção 2: Via Vercel CLI

```bash
# Instala Vercel CLI
npm install -g vercel

# Login
vercel login

# Deploy
vercel --prod
```

---

## ⚠️ PROBLEMA: Chamadas à API do Anthropic Bloqueadas pelo Browser

O browser bloqueia chamadas diretas do frontend para a API do Anthropic (CORS policy). 

### ✅ Solução: Criar Função Serverless no Vercel

#### 1. Criar pasta `api`

```bash
mkdir api
```

#### 2. Criar `api/claude.js`

```javascript
export default async function handler(req, res) {
  // Enable CORS
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }

  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': process.env.ANTHROPIC_API_KEY,
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify(req.body)
    });

    const data = await response.json();
    return res.status(200).json(data);
  } catch (error) {
    console.error('Error:', error);
    return res.status(500).json({ error: 'Internal server error' });
  }
}
```

#### 3. Atualizar `public/index.html`

Substitui todas as chamadas:
```javascript
// ANTES
const response = await fetch("https://api.anthropic.com/v1/messages", {
```

Por:
```javascript
// DEPOIS
const response = await fetch("/api/claude", {
```

E **REMOVE** o header `x-api-key` das chamadas (vai ser adicionado no servidor).

#### 4. Configurar API Key no Vercel

1. Vai ao dashboard do Vercel
2. Seleciona o projeto
3. Vai a **"Settings" → "Environment Variables"**
4. Adiciona:
   - **Name:** `ANTHROPIC_API_KEY`
   - **Value:** `sk-ant-...` (a tua chave API)
   - **Environment:** Production, Preview, Development
5. Clica em **"Save"**
6. Vai a **"Deployments"** e clica em **"Redeploy"**

#### 5. Faz novo deploy

```bash
git add .
git commit -m "Add serverless proxy for Anthropic API"
git push
```

Ou clica em **"Redeploy"** no Vercel dashboard.

---

## 🔗 URLs Após Deploy

Assumindo domínio `portal-clareza.vercel.app`:

- **Portal Público**: `https://portal-clareza.vercel.app`
- **Painel Admin**: `https://portal-clareza.vercel.app/admin`

---

## 📁 Estrutura de Ficheiros

```
portal-clareza-digital/
├── api/
│   └── claude.js              # Proxy serverless para API Anthropic
├── public/
│   └── index.html             # Portal público
├── private/
│   └── index.html             # Painel admin
├── vercel.json                # Configuração Vercel
└── README.md                  # Este ficheiro
```

---

## 🎨 Cores da Marca New You

- **Preto Principal**: `#1d1d1d`
- **Vermelho Escuro**: `#96082b`
- **Rosa/Magenta**: `#ad2569`
- **Cinza Claro**: `#ecebec`
- **Bege/Cinza**: `#b6b4af`

---

## 💾 Como Funcionam os Dados

- **Armazenamento**: `localStorage` do browser
- **Keys**: `portalLeads` (leads) e `portalBookings` (agendamentos)
- **Formato**: Array JSON

### Estrutura de um Lead:
```json
{
  "timestamp": "2024-11-14T10:30:00.000Z",
  "name": "João Silva",
  "email": "joao@exemplo.com",
  "instagram": "@joaosilva"
}
```

### Estrutura de um Agendamento:
```json
{
  "timestamp": "2024-11-14T10:35:00.000Z",
  "name": "João Silva",
  "email": "joao@exemplo.com",
  "instagram": "@joaosilva",
  "problem": "Baixa presença online",
  "whys": ["resposta1", "resposta2", "resposta3", "resposta4", "resposta5"],
  "analysis": {
    "rootCause": "...",
    "solutions": [...],
    "insights": [...]
  }
}
```

---

## ⚙️ Personalização

### Alterar Password do Admin

Edita `/private/index.html`, linha:
```javascript
const ADMIN_PASSWORD = 'NOVA_PASSWORD_AQUI';
```

### Adicionar Novos Exemplos de Problemas

Edita `/public/index.html`, objeto `examples`:
```javascript
const examples = {
    'novo-problema': 'Descrição do novo problema...',
    // ...
};
```

---

## 🐛 Troubleshooting

### Erro: "API Key inválida"
- Verifica se a chave começa com `sk-ant-`
- Confirma que configuraste a variável de ambiente `ANTHROPIC_API_KEY` no Vercel
- Verifica se tens créditos na conta Anthropic

### Erro: "CORS policy"
- Certifica-te que criaste a função serverless em `/api/claude.js`
- Verifica se estás a chamar `/api/claude` em vez de `https://api.anthropic.com`

### Leads não aparecem no Admin
- Admin e Portal devem estar no **mesmo domínio**
- Usa sempre HTTPS (não HTTP)
- Dados estão no `localStorage` - específico por browser

### Perdi os dados
1. Abre consola do developer (F12)
2. Vai a **Application → Local Storage**
3. Copia `portalLeads` e `portalBookings`
4. Guarda num ficheiro de backup

---

## 📈 Próximos Passos (Melhorias Futuras)

- [ ] Integração com Mailchimp/ConvertKit
- [ ] Sincronização com Google Sheets
- [ ] Base de dados real (Firebase/Supabase)
- [ ] Calendário Calendly para agendamento automático
- [ ] Notificações por email
- [ ] Analytics e tracking

---

## 📧 Suporte

Para questões ou suporte, contacta a equipa New You.

---

**Powered by New You** 💜

© 2024 Todos os direitos reservados