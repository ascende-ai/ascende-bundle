# Publicar uma release (desktop)

O aplicativo mostra **notas em português** na tela **Configurações → Atualizações**: texto vem do **corpo da GitHub Release** do repositório `ascende-bundle`. O auto-updater também usa esse conteúdo.

## Checklist antes de publicar

1. Escrever o corpo da release em **português**, focado no que o **usuário percebe** (fluxos, telas, confiabilidade), não em nomes de commits ou PRs.
2. Usar o **template** abaixo (seções e bullets curtos).
3. Conferir se a tag/versão bate com o binário e com o que o `electron-updater` espera.
4. Publicar a release (não deixar em rascunho se quiser que apareça no app e no histórico público).

## Template do corpo da release (copiar e preencher)

```markdown
### Novidades

- …

### Melhorias

- …

### Correções

- …
```

**Boas práticas**

- Cada bullet responde a “o que muda para mim?” em uma frase.
- Evite jargão interno (`refactor`, nome de branch, IDs de ticket) na interface visível ao usuário.
- Se algo for só interno/infra e invisível no app, não precisa entrar nas notas (ou cite numa linha única em “Melhorias” só se afetar desempenho ou estabilidade percebida).

## Onde isso aparece

- Banner e Configurações → Atualizações no app desktop (via `releaseNotes` do updater).
- Lista **Histórico de versões**, carregada pela API pública de releases do GitHub (repositório precisa permitir leitura anônima das releases, como já exigido pelo auto-updater).
