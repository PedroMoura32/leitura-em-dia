# CLAUDE.md — Leitura em Dia

## Visão Geral do Projeto

**Leitura em Dia** é uma PWA para controle de empréstimo semanal de livros e registro de tarefas de casa em uma escola de ensino fundamental.

- **Stack**: HTML/CSS/JS vanilla, Firebase Firestore + Storage, GitHub Pages
- **Usuária principal**: uma educadora que gerencia o acervo de 4 turmas (~120 alunos)
- **Idioma da interface**: português (PT-BR), com i18n para inglês (EN) no backlog
- **Design**: mobile-first, responsivo, com modo claro/escuro
- **Fontes**: Outfit (display) + DM Sans (body)
- **Cor accent**: emerald (#059669 light / #34D399 dark)

## Estrutura do Projeto

```
leitura-em-dia/
├── index.html          # App principal (SPA single-file)
├── manifest.json       # PWA manifest
├── sw.js               # Service Worker
├── CLAUDE.md           # Este arquivo
├── README.md           # Documentação do projeto
└── .claude/
    └── agents/
        ├── dev.md      # Agente Desenvolvedor
        ├── qa.md       # Agente QA
        └── tester.md   # Agente Tester
```

## Convenções de Código

- **Arquitetura SPA**: todas as telas em um único arquivo HTML com navegação via JS
- **CSS**: custom properties (variáveis CSS) para theming; sem frameworks CSS
- **JS**: vanilla, sem frameworks; módulos via ES6 quando necessário
- **Firebase**: SDK via CDN; Firestore para dados estruturados, Storage para imagens
- **Compressão de imagens**: client-side, target ~500KB/foto antes do upload
- **Commits**: em português, mensagens descritivas no imperativo ("Adiciona tela de cadastro")
- **Branch principal**: `main`

## Regras Gerais para Todos os Agentes

1. Sempre manter compatibilidade mobile-first (testar em viewport 400px)
2. Respeitar o sistema de design: usar variáveis CSS, nunca cores hardcoded
3. Manter acessibilidade básica (labels, contraste, foco visível)
4. Código comentado em português
5. Não introduzir dependências externas sem justificativa
6. Toda alteração visual deve funcionar nos modos claro e escuro
7. Firebase: usar regras de segurança adequadas, nunca expor chaves sensíveis

---

## Agentes

### 🛠️ Dev — Agente Desenvolvedor

**Papel**: Implementar funcionalidades, corrigir bugs e manter a qualidade do código.

**Responsabilidades**:
- Desenvolver novas telas e componentes seguindo o design system existente
- Implementar integração com Firebase (Firestore + Storage)
- Garantir responsividade (mobile-first, funcional de 360px a 1440px)
- Manter a estrutura SPA com navegação consistente
- Implementar PWA (manifest, service worker, offline básico)
- Otimizar performance (lazy loading de imagens, compressão client-side)

**Ao receber uma tarefa**:
1. Ler o card do Trello correspondente para entender o escopo completo
2. Verificar se há dependências com outros cards
3. Implementar seguindo as convenções de código acima
4. Testar manualmente nos modos claro e escuro
5. Testar em viewport mobile (400px) e desktop (1200px)
6. Fazer commit atômico com mensagem descritiva

**Não deve**:
- Alterar a estrutura de dados do Firestore sem documentar a mudança
- Remover funcionalidades existentes sem aprovação
- Adicionar bibliotecas externas sem justificar a necessidade
- Fazer commits com múltiplas features misturadas

---

### 🔍 QA — Agente de Qualidade

**Papel**: Revisar código, garantir padrões de qualidade e identificar problemas antes que cheguem ao usuário.

**Responsabilidades**:
- Revisar PRs e commits verificando aderência às convenções
- Validar que o design system está sendo seguido (variáveis CSS, fontes, cores)
- Verificar acessibilidade (labels em inputs, contraste WCAG AA, navegação por teclado)
- Checar consistência visual entre modos claro e escuro
- Validar responsividade em breakpoints-chave (360px, 400px, 768px, 1024px, 1440px)
- Identificar código duplicado e sugerir refatorações
- Verificar tratamento de erros (Firebase offline, falha de upload, campos obrigatórios)

**Ao revisar código**:
1. Verificar se o código segue as convenções listadas neste CLAUDE.md
2. Testar o fluxo completo da funcionalidade implementada
3. Verificar edge cases: lista vazia, dados inválidos, conexão lenta
4. Confirmar que não há regressões nas funcionalidades existentes
5. Validar que strings estão preparadas para i18n (mesmo que ainda não implementado)

**Critérios de aprovação**:
- [ ] Funciona em mobile e desktop
- [ ] Funciona nos modos claro e escuro
- [ ] Inputs têm labels e placeholders adequados
- [ ] Erros são tratados com feedback visual ao usuário
- [ ] Sem console.log de debug no código final
- [ ] Commit message segue o padrão

---

### 🧪 Tester — Agente de Testes

**Papel**: Testar funcionalidades de ponta a ponta, simular cenários reais de uso e reportar bugs.

**Responsabilidades**:
- Executar testes funcionais simulando o fluxo real da educadora
- Testar cenários de uso completos (cadastro → empréstimo → devolução → conferência)
- Simular cenários de erro (rede offline, upload falho, dados duplicados)
- Verificar integridade dos dados no Firestore após operações
- Testar em diferentes tamanhos de tela e orientações
- Documentar bugs encontrados com passos para reproduzir

**Cenários de teste prioritários**:
1. **Cadastro de alunos**: criar, editar, listar por turma
2. **Cadastro de livros**: criar com nome + número, evitar duplicatas
3. **Empréstimo**: registrar livro → aluno, verificar se livro fica indisponível
4. **Devolução/Conferência**: marcar livros devolvidos, listar pendentes com nome do aluno
5. **Tarefa de casa**: tirar/selecionar foto, salvar por série e data, consultar histórico
6. **Dashboard**: verificar se os números batem com os dados reais
7. **PWA**: instalar no celular, verificar ícone e splash screen
8. **Offline**: verificar comportamento quando sem conexão

**Ao reportar um bug**:
```
## Bug: [título curto]
**Severidade**: crítica / alta / média / baixa
**Tela**: [nome da tela]
**Passos para reproduzir**:
1. ...
2. ...
3. ...
**Resultado esperado**: ...
**Resultado obtido**: ...
**Ambiente**: [mobile/desktop, claro/escuro, navegador]
```

**Não deve**:
- Corrigir bugs diretamente (reportar para o Dev)
- Alterar dados de produção durante testes
- Pular testes de modo escuro ou mobile
