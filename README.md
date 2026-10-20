# Leitura em Dia

App para acompanhamento do empréstimo semanal de livros e registro de tarefas de casa em uma escola de ensino fundamental.

## Sobre o Projeto

**Leitura em Dia** é uma PWA (Progressive Web App) criada para auxiliar uma educadora no controle do rodízio semanal de livros entre alunos. O app permite registrar empréstimos, acompanhar devoluções, consultar o histórico de leitura de cada criança e registrar os deveres de casa semanais com foto.

### Público-alvo

- Professora/educadora responsável pelo acervo de livros da escola

### Estrutura escolar

- 4 turmas agrupadas em 2 séries: 3º ano (2 turmas) e 5º ano (2 turmas)
- ~30 alunos por turma (~120 alunos no total)
- Acervo de 60–100 livros por turma, diferenciado por série

## Funcionalidades

### MVP (Fase 1)

- Cadastro de turmas e alunos
- Cadastro de livros (nome + número identificador)
- Registro de empréstimo: livro → criança
- Conferência da cesta: marcar devoluções e ver quais livros faltam e com quem estão
- Registro de tarefa de casa semanal com foto (por série)

### Melhorias Futuras

- Etiquetas com QR Code nos livros (leitura via câmera)
- Dashboard com relatórios cruzados (por aluno, por turma, por livro)
- Histórico anual de leituras por criança
- Exportar histórico para análise externa
- Seletor de idioma PT/EN
- Modo claro/escuro

## Stack Técnica

| Camada       | Tecnologia                          |
| ------------ | ----------------------------------- |
| Frontend     | HTML + CSS + JavaScript (vanilla)   |
| Formato      | PWA (mobile-first, responsivo)      |
| Hospedagem   | GitHub Pages                        |
| Banco        | Firebase Firestore (plano gratuito) |
| Armazenamento| Firebase Storage (fotos de tarefas) |
| Gráficos     | Chart.js 4.4.1                      |
| Fontes       | Outfit (display) + DM Sans (body)   |
| QR Code      | html5-qrcode (futuro)              |

## Organização

- **Trello**: [Projeto Leitura em Dia](https://trello.com/b/UW9SuxjC/projeto-leitura-em-dia) — Kanban com backlog detalhado
- **Google Doc**: Brainstorm e decisões de produto

## Como Rodar Localmente

```bash
# Clone o repositório
git clone https://github.com/PedroMoura32/leitura-em-dia.git

# Abra index.html no navegador ou use um servidor local
npx serve .
```

## Licença

Projeto pessoal — uso privado.
