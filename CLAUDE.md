# CLAUDE.md — Leitura em Dia

## Visão Geral do Projeto

**Leitura em Dia** é uma PWA para controle de empréstimo semanal de livros e registro de tarefas de casa em uma escola de ensino fundamental.

- **Stack**: HTML/CSS/JS vanilla, Firebase Firestore + Storage, GitHub Pages
- **Usuária principal**: uma educadora que gerencia o acervo de 4 turmas (~120 alunos)
- **Idioma da interface**: português (PT-BR), com i18n para inglês (EN) no backlog
- **Design**: mobile-first, responsivo, com modo claro/escuro
- **Arquitetura**: SPA (Single Page Application) — todas as telas em um único `index.html`

## Referências do Projeto

- **Backlog completo (Trello)**: https://trello.com/b/UW9SuxjC/projeto-leitura-em-dia
- **Brainstorm e decisões (Google Doc)**: documento "Leitura em Dia - Brainstorm e Dúvidas" no Google Drive do dono do projeto
- **POC Visual (Artifact)**: https://claude.ai/artifact/2ntzBQtBvvjRG97MRtqck9 — protótipo HTML com 6 telas navegáveis que define a direção visual do app

> **IMPORTANTE**: A POC visual é a referência principal de design. O app final deve manter o mesmo nível de qualidade visual, identidade e acabamento da POC. Antes de implementar qualquer tela, consulte a POC para entender o padrão visual esperado.

---

## Design System

### Tipografia

| Uso            | Fonte    | Import                                                           |
| -------------- | -------- | ---------------------------------------------------------------- |
| Display/Títulos| **Outfit**   | `fonts.googleapis.com/css2?family=Outfit:wght@400;500;600;700`  |
| Corpo/UI       | **DM Sans** | `fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700` |

- Títulos de seção: Outfit 600–700, tamanho proporcional à hierarquia
- Corpo de texto: DM Sans 400, line-height confortável (~1.5)
- Labels e chips: DM Sans 500, letter-spacing +0.02em em uppercase

### Paleta de Cores

O sistema usa CSS custom properties para suportar modo claro e escuro.

```css
:root {
  /* Accent — Emerald */
  --accent: #059669;
  --accent-light: #D1FAE5;
  --accent-hover: #047857;

  /* Superfícies */
  --bg-primary: #FFFFFF;
  --bg-secondary: #F9FAFB;
  --bg-card: #FFFFFF;

  /* Texto */
  --text-primary: #111827;
  --text-secondary: #6B7280;
  --text-on-accent: #FFFFFF;

  /* Bordas e divisores */
  --border: #E5E7EB;
  --border-focus: #059669;

  /* Status */
  --success: #059669;
  --warning: #F59E0B;
  --error: #EF4444;
  --info: #3B82F6;
}

/* Modo escuro */
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --accent: #34D399;
    --accent-light: #064E3B;
    --accent-hover: #6EE7B7;

    --bg-primary: #111827;
    --bg-secondary: #1F2937;
    --bg-card: #1F2937;

    --text-primary: #F9FAFB;
    --text-secondary: #9CA3AF;
    --text-on-accent: #111827;

    --border: #374151;
    --border-focus: #34D399;

    color-scheme: dark;
  }
}

:root[data-theme="dark"] {
  /* Mesmos valores do bloco acima */
  --accent: #34D399;
  --accent-light: #064E3B;
  --accent-hover: #6EE7B7;
  --bg-primary: #111827;
  --bg-secondary: #1F2937;
  --bg-card: #1F2937;
  --text-primary: #F9FAFB;
  --text-secondary: #9CA3AF;
  --text-on-accent: #111827;
  --border: #374151;
  --border-focus: #34D399;
  color-scheme: dark;
}
```

### Componentes UI

- **Cards**: bordas arredondadas (`border-radius: 12px`), sombra sutil, fundo `--bg-card`
- **Botões primários**: fundo `--accent`, texto `--text-on-accent`, border-radius 8px, hover com `--accent-hover`
- **Inputs**: borda `--border`, foco com `--border-focus` e sombra emerald sutil
- **Chips de filtro (turmas)**: horizontais, scrolláveis, com estado ativo em `--accent`
- **Bottom navigation**: 5 itens com ícones (Início, Empréstimo, Conferência, Cadastros, Tarefa de Casa)
- **Empty states**: ilustrações ou ícones com mensagem amigável, nunca tela em branco
- **Feedback visual**: toasts para sucesso/erro, loading spinners durante operações Firebase

### Direção Visual

O visual deve ser **moderno e acolhedor**, indo além do ambiente escolar tradicional:
- Cantos arredondados generosos
- Espaçamento confortável (padding 16–24px em cards)
- Micro-animações sutis (transições de 200–300ms)
- Ícones limpos (estilo outline, não preenchido)
- Sem elementos infantis ou cliparts — o público do app é a professora, não as crianças
- Inspiração: apps modernos de produtividade pessoal (Notion, Linear, Todoist)

---

## Estrutura de Dados (Firestore)

### Collections

```
turmas/
  {turmaId}
    nome: "3º Ano A"
    serie: "3"
    ano_letivo: 2025

alunos/
  {alunoId}
    nome: "Maria Silva"
    turma_id: "ref_turma"
    ativo: true

livros/
  {livroId}
    titulo: "O Pequeno Príncipe"
    numero: 42
    turma_id: "ref_turma"  // livros são por turma/série
    disponivel: true

emprestimos/
  {emprestimoId}
    aluno_id: "ref_aluno"
    livro_id: "ref_livro"
    data_emprestimo: timestamp
    data_devolucao: timestamp | null
    devolvido: false

tarefas_casa/
  {tarefaId}
    serie: "3"  // "3" ou "5" — agrupa as 2 turmas da série
    data: timestamp
    imagem_url: "gs://..."
    semana: 12  // número da semana no ano
```

### Regras de negócio

- Um livro só pode estar emprestado para um aluno por vez
- A devolução é coletiva: a professora confere a cesta e marca os devolvidos
- Livros são separados por série (3º ano e 5º ano têm acervos diferentes)
- Tarefa de casa é por série (não por turma) — o mesmo dever serve para as 2 turmas da mesma série
- Apenas ~2 registros de tarefa por semana (1 por série × sexta-feira)
- Imagens de tarefa: comprimidas client-side para ~500KB antes do upload ao Firebase Storage

---

## Telas do App

Baseadas na POC visual. Cada tela tem navegação via bottom bar.

### 1. Início (Home)
- Resumo geral: total de livros emprestados, pendentes de devolução, alunos ativos
- Chips de filtro por turma (horizontais, scrolláveis)
- Cards de atividade recente (últimos empréstimos/devoluções)
- Estatísticas rápidas com números grandes e labels

### 2. Empréstimo
- Selecionar turma → selecionar aluno → selecionar livro (ou digitar número)
- Botão de QR code (futuro, desabilitado inicialmente com tooltip explicativo)
- Confirmação visual após registro
- Lista de empréstimos ativos da turma selecionada

### 3. Conferência da Cesta
- Selecionar turma
- Lista de todos os livros emprestados da turma com checkbox para marcar devolução
- Destaque visual dos livros pendentes com nome do aluno
- Botão "Confirmar devoluções" que dá baixa em lote

### 4. Cadastros
- Sub-telas: Turmas, Alunos, Livros
- CRUD simples com formulários limpos
- Busca/filtro em cada lista

### 5. Tarefa de Casa
- Registrar: selecionar série, data, tirar/selecionar foto
- Consultar: timeline por série, buscar por data ou semana
- Visualizar: imagem em tela cheia com zoom

### 6. Dashboard (melhoria futura)
- Gráficos com Chart.js 4.4.1
- Por aluno: quantos e quais livros leu, alerta de repetição
- Por turma: livros mais procurados, livros que nunca saíram
- Por livro: popularidade, gênero/tema (se cadastrado)

### 7. Perfil do Aluno (detalhe)
- Histórico de leituras do aluno
- Total de livros lidos
- Alertas de livros repetidos

---

## Estrutura de Arquivos (Alvo)

```
leitura-em-dia/
├── index.html              # App completo (SPA single-file)
├── manifest.json           # PWA manifest
├── sw.js                   # Service Worker (cache offline)
├── firebase-config.js      # Configuração Firebase (sem chaves secretas em produção)
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
├── CLAUDE.md               # Este arquivo
├── README.md               # Documentação pública
└── .claude/
    └── agents/             # Definições dos agentes (futuro)
```

---

## Convenções de Código

- **Arquitetura SPA**: todas as telas em um único HTML com navegação via JS; cada tela é uma `<section>` com `id="screen-nome"`
- **CSS**: custom properties para theming; classes BEM-like (`.card`, `.card__title`, `.card--active`); sem frameworks
- **JS**: vanilla ES6+; funções organizadas por domínio (ui, firebase, utils); sem frameworks
- **Firebase**: SDK via CDN (`firebase/10.x`); Firestore para dados, Storage para imagens
- **Compressão de imagens**: canvas API client-side, target ~500KB/foto, formato JPEG quality 0.7
- **Commits**: em português, imperativo ("Adiciona tela de cadastro", "Corrige filtro de turma")
- **Branch principal**: `main`
- **Sem dependências pesadas**: o app é leve, roda em qualquer celular com navegador moderno

---

## Regras Gerais para Todos os Agentes

1. **Mobile-first**: desenvolver primeiro para 400px, depois adaptar para desktop
2. **Design system**: usar SEMPRE variáveis CSS (`var(--accent)`, nunca `#059669` direto)
3. **Acessibilidade**: labels em todo input, contraste WCAG AA, estados de foco visíveis
4. **Bilíngue-ready**: todas as strings de UI em um objeto `i18n` (mesmo que só PT por enquanto)
5. **Dois temas**: toda alteração visual DEVE funcionar nos modos claro e escuro
6. **Sem surpresas**: não introduzir dependências ou mudar estrutura de dados sem documentar
7. **Firebase seguro**: regras de segurança adequadas, nunca commitar chaves em texto aberto
8. **Referência visual**: consultar a POC antes de implementar qualquer tela nova
9. **Código limpo**: comentários em português, sem console.log de debug no código final
10. **Commits atômicos**: uma feature por commit, mensagem descritiva

---

## Agentes

### 🛠️ Dev — Agente Desenvolvedor

**Papel**: Implementar funcionalidades, corrigir bugs e manter a qualidade do código.

**Responsabilidades**:
- Desenvolver novas telas e componentes seguindo o design system acima
- Implementar integração com Firebase (Firestore + Storage)
- Garantir responsividade (mobile-first, funcional de 360px a 1440px)
- Manter a estrutura SPA com navegação consistente via bottom bar
- Implementar PWA (manifest, service worker, offline básico)
- Otimizar performance (lazy loading, compressão client-side de imagens)

**Ao receber uma tarefa**:
1. Ler o CLAUDE.md para contexto do design system e estrutura de dados
2. Consultar a POC visual para entender o padrão visual esperado
3. Implementar seguindo as convenções de código
4. Testar nos modos claro e escuro
5. Testar em viewport mobile (400px) e desktop (1200px)
6. Fazer commit atômico com mensagem descritiva em português

**Não deve**:
- Alterar a estrutura de dados do Firestore sem documentar a mudança neste CLAUDE.md
- Remover funcionalidades existentes sem aprovação
- Adicionar bibliotecas externas sem justificar a necessidade
- Fazer commits com múltiplas features misturadas
- Usar cores, fontes ou espaçamentos fora do design system

---

### 🔍 QA — Agente de Qualidade

**Papel**: Revisar código, garantir padrões de qualidade e identificar problemas antes que cheguem à usuária.

**Responsabilidades**:
- Revisar código verificando aderência ao design system e convenções do CLAUDE.md
- Validar acessibilidade (labels, contraste WCAG AA, navegação por teclado)
- Checar consistência visual entre modos claro e escuro
- Validar responsividade em breakpoints: 360px, 400px, 768px, 1024px, 1440px
- Identificar código duplicado e sugerir refatorações
- Verificar tratamento de erros (Firebase offline, falha de upload, campos obrigatórios)
- Comparar resultado com a POC visual para garantir fidelidade

**Checklist de aprovação**:
- [ ] Funciona em mobile (400px) e desktop (1200px)
- [ ] Funciona nos modos claro e escuro sem quebras
- [ ] Variáveis CSS usadas (sem cores hardcoded)
- [ ] Fontes corretas (Outfit para títulos, DM Sans para corpo)
- [ ] Inputs têm labels e placeholders adequados
- [ ] Erros são tratados com feedback visual (toasts/alerts)
- [ ] Strings prontas para i18n (no objeto de tradução)
- [ ] Sem console.log de debug
- [ ] Commit message em português, no imperativo

---

### 🧪 Tester — Agente de Testes

**Papel**: Testar funcionalidades de ponta a ponta, simular cenários reais de uso e reportar bugs.

**Responsabilidades**:
- Executar testes funcionais simulando o fluxo real da educadora
- Testar cenários completos (cadastro → empréstimo → conferência → devolução)
- Simular cenários de erro (rede offline, upload falho, dados duplicados)
- Verificar integridade dos dados no Firestore após operações
- Testar em diferentes tamanhos de tela e orientações
- Documentar bugs com passos para reproduzir

**Cenários de teste prioritários**:
1. **Cadastro de turma**: criar as 4 turmas (3ºA, 3ºB, 5ºA, 5ºB), verificar agrupamento por série
2. **Cadastro de alunos**: criar ~30 por turma, editar, listar por turma
3. **Cadastro de livros**: criar com nome + número, testar número duplicado na mesma turma
4. **Empréstimo**: registrar livro → aluno, verificar que livro fica indisponível para outros
5. **Conferência/Devolução**: marcar livros devolvidos, listar pendentes com nome do aluno
6. **Tarefa de casa**: capturar/selecionar foto, salvar por série e data, consultar por semana
7. **Filtro de turma**: chips na home, verificar que estatísticas mudam por turma
8. **Tema claro/escuro**: alternar e verificar todas as telas
9. **PWA**: instalar no celular, verificar ícone, splash e offline
10. **Edge cases**: lista vazia, nome muito longo, foto muito grande, sem internet

**Template de bug report**:
```
## Bug: [título curto]
**Severidade**: crítica / alta / média / baixa
**Tela**: [nome da tela]
**Passos para reproduzir**:
1. ...
2. ...
**Resultado esperado**: ...
**Resultado obtido**: ...
**Ambiente**: [mobile/desktop, claro/escuro, navegador]
```

**Não deve**:
- Corrigir bugs diretamente (reportar para o Dev)
- Alterar dados de produção durante testes
- Pular testes de modo escuro ou mobile
