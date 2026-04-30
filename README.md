# 🎨 Figma to Component Power

Power para o [Kiro](https://kiro.dev) que converte designs do Figma em componentes React prontos para produção, seguindo a arquitetura, regras de steering e biblioteca de componentes do seu projeto.

---

## 📋 O que é este Power?

Este power automatiza o fluxo **Figma → Código React**. Ele utiliza o servidor MCP **Framelink Figma MCP** para buscar dados do design diretamente do Figma e, em seguida, refatora o resultado bruto em componentes limpos, responsivos e padronizados — respeitando todas as regras do seu projeto.

O diferencial é que ele **não gera código genérico**. Antes de criar qualquer componente, ele analisa o que já existe no seu projeto e reutiliza, evitando duplicação.

---

## 📁 Estrutura do Power

```
figma-to-component-power/
├── POWER.md              # Arquivo principal do power (instruções para a IA)
├── figma_conversion.md   # Steering file auxiliar (guia detalhado de conversão)
├── mcp.json              # Configuração do servidor MCP do Figma (pronto para uso)
└── README.md             # Este arquivo
```

### `POWER.md`

Arquivo principal que a IA carrega ao ativar o power. Contém:

- **Visão geral** do que o power faz e como funciona
- **Configuração do MCP** — como configurar o Framelink Figma MCP no `mcp.json`
- **5 princípios fundamentais:**
  1. Reutilizar componentes existentes (nunca recriar)
  2. Divisão inteligente de componentes (nunca monolítico)
  3. Design responsivo mobile-first (sem tamanhos fixos)
  4. Estratégia de dados mockados (quando não há API)
  5. Respeitar todas as regras de steering do projeto
- **Workflow completo** em 4 etapas: buscar dados do Figma → analisar o design → gerar componentes → revisar
- **Padrões comuns** com exemplos de código (modais, cards, páginas com tabs, listas com filtros)
- **Troubleshooting** para problemas comuns

### `figma_conversion.md`

Steering file auxiliar com `inclusion: always` — é incluído automaticamente sempre que o power é ativado. Contém:

- **Checklist pré-conversão** — o que fazer antes de escrever qualquer código (incluindo leitura do `tailwind.config` e scan de icon library)
- **Mapeamento de design tokens** — regras detalhadas para mapear cores, fontes e espaçamentos do Figma para tokens Tailwind do projeto, incluindo registro de novas cores no `tailwind.config`
- **Estratégia de ícones e assets** — como descobrir e reutilizar a icon library do projeto, quando baixar SVGs, padrão de componente de ícone
- **Regras de mapeamento de componentes** — lista de elementos comuns de frontend (botões, modais, inputs, tabs, etc.) que a IA deve procurar no projeto antes de criar novos
- **Estratégia de divisão** — quando criar um novo arquivo de componente, hierarquia de componentes, onde colocar cada arquivo
- **Regras de design responsivo** — padrões permitidos, padrões proibidos, tamanhos fixos permitidos
- **Regras de estilização** — instruções para seguir as regras de styling do projeto (sem hex arbitrário, ordem de classes Tailwind)
- **Ordenação de classes Tailwind** — ordem padrão consistente: layout → sizing → spacing → typography → colors → borders → effects → transitions → states → responsive
- **Convenção de naming de props e handlers** — `is`/`has` para booleans, `on` para event props, `handle` para handlers internos
- **Revisão de fidelidade visual** — checklist de comparação com o Figma, documentação de divergências com comentários `// NOTE:`
- **Diretrizes de dados mockados** — estrutura, regras, boas práticas
- **Checklist de qualidade completo** — verificação final categorizada (export, reuso, TypeScript, tokens, styling, interatividade, naming, dados, fidelidade visual)
- **Resumo do processo** — fluxo completo em 11 passos

### `mcp.json`

Arquivo de configuração do servidor MCP do Figma, pronto para uso. Basta substituir `YOUR_FIGMA_API_KEY_HERE` pela sua API key do Figma e o power já estará funcional.

---

## 🚀 Como Usar

### Pré-requisitos

1. [Kiro](https://kiro.dev) instalado
2. Token de acesso pessoal do Figma

### Configuração do MCP

Este power utiliza o **Framelink Figma MCP**, um servidor MCP open source para integração com o Figma.
Repositório: [https://github.com/glips/figma-context-mcp](https://github.com/glips/figma-context-mcp)

O arquivo `mcp.json` já vem incluso dentro da pasta do power, pronto para uso. A única configuração necessária é substituir o placeholder pela sua API key do Figma:

1. Abra o arquivo `mcp.json` dentro da pasta do power
2. Substitua `YOUR_FIGMA_API_KEY_HERE` pelo seu token de acesso pessoal do Figma
   - Para gerar o token: Figma → Settings → Personal Access Tokens
3. O Kiro detecta automaticamente o `mcp.json` do power e carrega o servidor MCP

### Instalação do Power

1. Copie a pasta `figma-to-component-power/` para o local de powers do seu projeto
2. Certifique-se de que o `POWER.md` e o `figma_conversion.md` estão dentro da pasta
3. Ative o power no Kiro

### Uso no Chat

Para obter o melhor resultado, use o prompt modelo abaixo como base:

> **Prompt modelo — copie e adapte:**
>
> Converta este design do Figma em componentes React para o meu projeto.
>
> Link do Figma: https://www.figma.com/design/XXXXX/MeuProjeto?node-id=123-456
>
> [arraste o print/screenshot da tela aqui]
>
> Contexto:
> - Esta é a página de [nome da página, ex: "Dashboard", "Login", "Listagem de Produtos"]
> - [Opcional: descreva brevemente o que a página faz]
> - [Opcional: mencione estados específicos se o Figma não mostra todos, ex: "Preciso também de loading com skeleton e empty state"]
>
> Siga o fluxo completo do power Figma to Component:
> 1. Busque os dados via Framelink MCP
> 2. Mapeie os design tokens (cores, fontes, espaçamentos) para o tailwind.config do projeto — registre novas cores se necessário
> 3. Escaneie os componentes existentes em src/components/ e a icon library antes de criar qualquer coisa
> 4. Leia todos os steering files em .kiro/steering/
> 5. Planeje a árvore de componentes antes de gerar código
> 6. Gere cada componente seguindo todas as regras
> 7. Faça a revisão de fidelidade visual e documente divergências com // NOTE:

#### Por que cada parte importa

- **Link do Figma** — é o input obrigatório pro MCP buscar os dados da tela.
- **Screenshot** — melhora significativamente a fidelidade. A IA consegue "ver" o layout real e não depender só dos dados estruturais do MCP, que às vezes perdem nuances visuais. Basta arrastar a imagem para o chat ou clicar no ícone de anexo.
- **Nome e contexto da página** — ajuda a IA a nomear os arquivos corretamente (ex: `DashboardPage.tsx`, `DashboardMetricCard.tsx`) e a gerar mock data com conteúdo realista pro domínio.
- **Menção a estados extras** — o Figma geralmente mostra só o "happy path". Se você precisa de loading, empty ou error, precisa pedir explicitamente, senão a IA vai gerar só o estado default e marcar os outros com `// TODO`.
- **Listar os passos** — reforça pra IA seguir o fluxo completo. Sem isso, ela pode pular etapas como o mapeamento de tokens ou a revisão de fidelidade.

#### Variações úteis

Se for um **modal** ao invés de uma página, adicione ao prompt:

> Este design é um modal de [descrição]. Use o componente Dialog/Modal existente do projeto como base.

Se quiser que a IA **não crie novos componentes base** (só use os existentes):

> Use apenas componentes que já existem em src/components/. Se algo não existir, me avise antes de criar.

Se o projeto já tem **dados reais** (API pronta):

> Não gere mock data. O hook useProducts() já retorna os dados necessários. Use-o diretamente.

💡 **Resumo:** Quanto mais contexto sobre a página e o que você espera, melhor o resultado. O power já cuida de toda a parte técnica (tokens, reuso, splitting, fidelidade) — o prompt só precisa dar o "o quê" e o "onde".

---

## 🏗️ Bases para Código de Qualidade

O power se apoia em várias bases para garantir que o código gerado seja de alta qualidade:

### Reutilização de Componentes

A IA escaneia `src/components/` (ou equivalente) antes de criar qualquer coisa. Elementos comuns que ela procura:

- Botões (com suporte a variantes)
- Modais / Dialogs
- Dropdowns / Selects / Comboboxes
- Inputs de texto, busca, data
- Checkboxes, toggles, radio buttons
- Tabs / Navegação por abas
- Paginação
- Toasts / Notificações
- Cards
- Headers, footers, sidebars
- Badges, avatars, tooltips
- Tabelas, accordions
- Skeletons / Loading states
- Empty states
- Barras de filtro
- Ícones SVG

Se existe → usa. Se precisa de extensão → estende. Só cria novo se não houver nada equivalente.

### Mapeamento de Design Tokens

A IA nunca usa valores hex arbitrários nos componentes. Antes de gerar código:

- Lê o `tailwind.config` do projeto para entender a paleta existente
- Mapeia cada cor do Figma para o token Tailwind mais próximo
- Se uma cor não tem match, **registra uma nova cor** no `tailwind.config` com nome descritivo e shade number (ex: `'midnight-900': '#1A1A2E'`)
- Mapeia font sizes e espaçamentos para as classes Tailwind mais próximas
- Resultado: zero valores arbitrários como `text-[#1A1A2E]` ou `text-[17px]` nos componentes

### Estratégia de Ícones

A IA não cola SVG inline nos componentes. Antes de criar qualquer ícone:

- Verifica se o projeto usa uma icon library (Lucide, Heroicons, Phosphor, etc.)
- Se o ícone do Figma tem equivalente na library → usa a library
- Se não tem → baixa o SVG e cria um componente de ícone dedicado seguindo o padrão do projeto
- Ícones sempre ficam na pasta de ícones do projeto, nunca inline no JSX

### Divisão Inteligente

Nunca gera um componente monolítico para uma tela inteira. Divide em:

- **Página** — orquestra layout e composição (lógica fina)
- **Componentes de feature** — blocos de UI independentes (cards, listas, modais)
- **Componentes compartilhados** — elementos genéricos reutilizáveis

Critérios para dividir:
- Responsabilidade própria
- Potencial de reutilização
- Estado local próprio
- JSX excede ~80-100 linhas

### Design Responsivo

- Mobile-first — funciona em telas de 320px
- Sem larguras fixas em containers (`w-[450px]` proibido)
- Classes responsivas do Tailwind (`grid-cols-1 md:grid-cols-2 lg:grid-cols-3`)
- Layouts com `flex` e `grid` + breakpoints

### Conformidade com Steering

Antes de gerar código, a IA lê todos os arquivos `.kiro/steering/*.md` do projeto e aplica:

- Regras de estilização (Tailwind, exceções de `style`, regras de margin)
- Padrões de páginas (estrutura, naming, composição)
- Padrões de hooks (naming, retorno, data fetching)
- Padrões de formulários (edit mode, loading states, validação)
- Padrões de DTOs (propriedades opcionais, naming)
- Padrões de utils (classes estáticas, sem dependências)
- Regras gerais (file naming, imports, TypeScript)

### TypeScript Rigoroso

- Sem `any`
- Sem type casts (`as Type`)
- Interfaces para props de componentes
- Dados mockados tipados conforme DTOs do projeto

### Ordenação de Classes Tailwind

Todas as classes Tailwind seguem uma ordem consistente para melhorar legibilidade:

layout → sizing → spacing → typography → colors → borders → effects → transitions → states → responsive

Se o projeto usa `prettier-plugin-tailwindcss`, a ordenação automática do plugin é respeitada.

### Convenção de Naming

Props e handlers seguem padrões consistentes:

- Booleans: prefixo `is`/`has` (`isOpen`, `isLoading`, `hasError`)
- Event handlers (props): prefixo `on` (`onClick`, `onClose`, `onSubmit`)
- Handlers internos: prefixo `handle` (`handleClick`, `handleClose`)

### Revisão de Fidelidade Visual

Após gerar os componentes, a IA compara o resultado com o design original do Figma:

- Verifica layout, espaçamentos, cores, tipografia, ícones e estados
- Quando diverge intencionalmente do Figma (ex: usar shadow do projeto ao invés do Figma), documenta com comentário `// NOTE:`
- Garante que nenhum detalhe visual importante foi perdido na conversão

---

## ✅ Boas Práticas

- Sempre use `export const` com arrow function para todos os componentes
- Sempre escaneie os componentes existentes antes de criar novos
- Na dúvida entre dividir ou não, divida — componentes menores são mais fáceis de manter
- Páginas devem ser finas — orquestram, não implementam
- Use HTML semântico (`section`, `nav`, `header`, `main`, `aside`)
- Adicione `aria-label` em elementos interativos sem texto visível
- Use `gap` ao invés de margins para espaçamento entre irmãos
- Prefira `grid` para layouts 2D e `flex` para layouts 1D
- Nomeie dados mockados claramente: `MOCK_USERS`, `MOCK_PRODUCTS`
- Sempre marque dados mockados com `// TODO: Replace with API data`
- Siga a convenção de naming do projeto (verifique steering)
- Nunca use valores hex arbitrários — sempre mapeie para tokens Tailwind nomeados
- Nunca cole SVG inline no JSX — use a icon library ou crie um componente de ícone
- Ordene classes Tailwind consistentemente (layout → sizing → spacing → typography → colors → borders → effects)
- Use prefixos consistentes: `is`/`has` para booleans, `on` para event props, `handle` para handlers internos
- Documente divergências visuais com comentários `// NOTE:`

---

## 🔧 Troubleshooting

| Problema | Solução |
|----------|---------|
| MCP retorna dados vazios | Verifique `fileKey` e `nodeId`. Tente sem `nodeId` primeiro. |
| Erro de autenticação Figma | Verifique se o token tem permissão de leitura no arquivo. |
| Fontes customizadas no design | A IA usa a fonte padrão do projeto (verifique Tailwind config). |
| Cores fora da paleta | A IA registra novas cores no `tailwind.config` com nome descritivo e shade number. Nunca usa hex arbitrário. |
| Componente ficou muito grande | Divida em sub-componentes. Extraia lógica para hooks/utils. |

---

## 📄 Licença

Este power é de uso livre. Sinta-se à vontade para adaptar, modificar e distribuir.

---

## 👤 Autor

[Nathan Lunelli](https://www.linkedin.com/in/nathan-lunelli-23bb30290)
