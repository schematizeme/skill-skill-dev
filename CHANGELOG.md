# Changelog — schematize-skill-dev

Todas as mudanças relevantes deste pacote, no formato [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/),
com versionamento [SemVer](https://semver.org/lang/pt-BR/).

## [0.2.0] — 2026-10-14
### Alterado
- `check-skill.sh` recalibrado após rodar nas 45 skills: `lib.sh`/`*.lib.sh` isentos de `script-sem-teste`;
  menos de 3 comandos de trabalho vira AVISO (o mínimo segue erro só para help/load/claude/handoff/cc);
  prefixo de comando vem do `<prefixo>-help.md` (aceita hífen, ex.: `overdev-context-*`); `schematize-help.md`
  aceito na engineering. Casos novos (vermelho e verde) em `check-skill.test.sh`.
- `references/release.md` aponta `schematize-git` para o fluxo git geral; description reduzida a ≤190.
- Porquê: o gate gerava falso positivo em skills legítimas e o orçamento de descriptions do catálogo estourou.

## [0.1.0] — 2026-10-14
Primeira versão. O catálogo cresceu para dezenas de skills e o conhecimento de como fazer uma estava
espalhado em `tools/` (lint, release, sync de herdados, grafo) e na cabeça de quem já publicou. Cada
skill nova reintroduzia os mesmos defeitos (versão em 3 de 4 lugares, description estourada, script sem
teste, comando colidindo com o de outra skill) e só o lint do catálogo, tarde, acusava.

### Adicionado
- `SKILL.md` com 13 pisos, fronteira com `schematize-docs` e `schematize-engineering`, mapa de references.
- `references/`: `anatomia`, `description-e-comandos`, `herdados`, `lint-e-testes`, `release`,
  `integracao-catalogo` e `stack-versoes` (anexo volátil com data e fonte).
- `scripts/check-skill.sh` (gate: estrutura, frontmatter, description, versão nos 4 lugares, CHANGELOG,
  snapshot, comandos, script com teste vermelho e verde, herdados, anexo volátil, emoji, zip, caminho
  privado, CI pinado, contagem de pisos) + `check-skill.test.sh` (40 casos); `bump-skill.sh` e
  `pack-skill.sh`, cada um com teste.
- Comandos: `skill-help`, `skill-load`, `skill-new`, `skill-review`, `skill-bump`, `skill-release`,
  `skill-claude`, `skill-handoff`, `skill-cc`.

### Não verificado
- O rito de release foi reproduzido à mão (tag, zip, `gh release`) na 1ª versão; `tools/release.mjs`
  não foi executado por esta skill (ver `references/stack-versoes.md`).
