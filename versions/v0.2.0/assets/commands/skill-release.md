---
description: schematize-skill-dev — rito de release de uma skill: gate, bump, snapshot, zip, tag, GitHub Release, grafo e skills.json.
argument-hint: "<slug> <versão X.Y.Z>"
---

# /skill-release — o release é um rito, não um push

Argumentos: `${ARGUMENTS}`. Detalhe e erros comuns em `references/release.md`.

1. **Pré-condições:** CHANGELOG com a entrada da versão no topo (com o porquê); árvore da skill
   limpa; branch `main` sincronizada com `origin`; gate da skill verde
   (`scripts/check-skill.sh`) e testes passando.
2. **Com o catálogo ao lado** (existe `tools/release.mjs`): rode primeiro `--dry-run`, depois
   `node tools/release.mjs <slug> <versão>`. Ele roda o gate global, bumpa, grava o snapshot, empacota,
   commita com lista explícita, cria a tag anotada e o GitHub Release, e regenera grafo e `skills.json`.
   Lote: `--batch <slug>@<ver> ...`.
3. **Sem o catálogo (1ª versão, repo isolado):** `scripts/bump-skill.sh` (4 lugares + snapshot),
   `scripts/pack-skill.sh` (zip com top-level `schematize-<slug>/`), `git add` com lista explícita,
   commit, tag anotada `v<ver>`, `git push --follow-tags` e `gh release create v<ver>
   skill-<slug>.zip`. O zip nunca é commitado.
4. **Instale e prove:** `bash skill-<slug>/install.sh "$HOME"` e confira o comando novo.
5. **Depois:** `skills.json` do front e `tools/graph-meta.json` (entrada nova) são do dono do
   catálogo; diga o que falta em vez de editar fora do seu escopo.
6. Release que falhou no meio retoma sem duplicar: tag existente não é recriada e o asset do release
   é regravado (`--clobber`).
