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

Steering file auxiliar com `inclusion: manual` — você ativa via `#` no chat do Kiro quando precisar. Contém:

- **Checklist pré-conversão** — o que fazer antes de escrever qualquer código
- **Regras de mapeamento de componentes** — lista de elementos comuns de frontend (botões, modais, inputs, tabs, etc.) que a IA deve procurar no projeto antes de criar novos
- **Estratégia de divisão** — quando criar um novo arquivo de componente, hierarquia de componentes, onde colocar cada arquivo
- **Regras de design responsivo** — padrões permitidos, padrões proibidos, tamanhos fixos permitidos
- **Regras de estilização** — instruções para seguir as regras de styling do projeto
- **Diretrizes de dados mockados** — estrutura, regras, boas práticas
- **Checklist de qualidade** — verificação final para cada componente gerado
- **Resumo do processo** — fluxo completo em 7 passos

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

Basta enviar uma mensagem como:

```
Converta este design do Figma em componentes:
https://www.figma.com/design/XXXXX/MeuProjeto?node-id=123-456
```

A IA vai:
1. Buscar os dados do Figma via MCP
2. Analisar o design
3. Escanear seus componentes existentes
4. Ler suas regras de steering
5. Gerar os componentes seguindo todas as regras

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

---

## ✅ Boas Práticas

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

---

## 🔧 Troubleshooting

| Problema | Solução |
|----------|---------|
| MCP retorna dados vazios | Verifique `fileKey` e `nodeId`. Tente sem `nodeId` primeiro. |
| Erro de autenticação Figma | Verifique se o token tem permissão de leitura no arquivo. |
| Fontes customizadas no design | A IA usa a fonte padrão do projeto (verifique Tailwind config). |
| Cores fora da paleta | A IA usa valores arbitrários do Tailwind: `text-[#HEXCODE]`. |
| Componente ficou muito grande | Divida em sub-componentes. Extraia lógica para hooks/utils. |

---

## 📄 Licença

Este power é de uso livre. Sinta-se à vontade para adaptar, modificar e distribuir.

---

## 👤 Autor

[Nathan Lunelli](https://www.linkedin.com/in/nathan-lunelli-23bb30290)
