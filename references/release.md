# Release de uma skill

> Parte da `schematize-skill-dev`. O release é um rito com ordem: gate, bump, snapshot, CHANGELOG,
> zip, git, GitHub Release, derivados. Pular um passo reintroduz o defeito que o passo evita.

## 1. Os lugares da versão

| # | Lugar | Quem grava |
|---|---|---|
| 1 | `VERSION` | release / `scripts/bump-skill.sh` |
| 2 | `current_version` do `skill.toml` | idem |
| 3 | `metadata.version` do frontmatter do `SKILL.md` | idem |
| 4 | linha `**Versão:** ... vX.Y.Z` do `SKILL.md` | idem |
| 5 | `skills.json` do front | derivado dos `skill.toml` (`tools/sync-skills-json.mjs`) |

Mais: `versions/v<ver>/` (snapshot) e a primeira entrada do `CHANGELOG.md` (escrita por gente, com o
porquê, ANTES do release: o release recusa sem ela e não inventa texto).

## 2. No catálogo: `tools/release.mjs` é o ÚNICO caminho

Dentro do catálogo, **não faça nenhum passo à mão** (nem "só o zip", nem "só a tag"): a ferramenta
valida, é idempotente e publica os derivados. O rito manual da seção 3 é fallback para fora dele.

```bash
node tools/release.mjs <slug> <versão> --dry-run        # ensaio: pré-valida, não grava
node tools/release.mjs <slug> <versão> [--notes "..."]
node tools/release.mjs --batch <slug>@<ver> <slug>@<ver> ...   # gate uma vez, publica em série
# --no-push-derived: pula o commit/push de skills.json e grafos (só se pedirem)
```

Ordem interna (para no primeiro erro):

1. **Pré-condições:** versão SemVer e MAIOR que a atual; 1ª entrada do CHANGELOG igual à versão;
   árvore da skill **limpa**; branch `main`; `main` sincronizada com `origin` (nem commit local sem
   push, nem remoto à frente).
2. **Gate:** `tools/ci.sh` (self-test do lint, índice, herdados, `skills.json`, lint). `--skip-gate`
   é dívida declarada, não atalho.
3. **Bump** nos 4 lugares.
4. **Snapshot** `versions/v<ver>/` (`SKILL.md`, `VERSION`, `skill.toml`, `references`, `assets`,
   `scripts`, `gui.json`).
5. **Zip** `skill-<slug>.zip`, top-level `schematize-<slug>/` (acrescenta `CHANGELOG.md` e `LICENSE`).
6. **Git:** `git add` de **lista explícita** (`VERSION`, `skill.toml`, `SKILL.md`, `versions/v<ver>`,
   `CHANGELOG.md`; nunca `-A`), commit `release: v<ver>`, tag anotada `v<ver>`, `push --follow-tags`.
   Fluxo git geral (commit, PR, tag fora de release de skill): `schematize-git`.
   Variável `RELEASE_COAUTHOR` (opcional) acrescenta o trailer `Co-Authored-By`.
7. **GitHub Release** com o zip; se a release já existe, só regrava o asset (`--clobber`). O zip é
   apagado num `finally`.
8. **Derivados:** `sync-skills-json` e `build-graph` (o grafo carrega a versão de cada skill; sem
   regerar, o release SEGUINTE falha no gate por causa deste).
9. **Publica os derivados:** commit só de `content/skills.json` no front
   (`chore(skills): versões do lote ...`) e de `.schematize/grafos/` no catálogo
   (`chore(grafos): versões do lote ...`), com push e `pull --rebase` se recusado; idempotente.
10. **Gate final:** `tools/ci.sh` de novo; o rc é reportado (rc != 0 -> exit 1).

Retomada: tag já no remoto não é recriada; "nada novo a commitar" segue adiante. Lote interrompido
reporta publicadas e faltantes, sem desfazer.

## 3. Fallback, só FORA do catálogo: primeira versão ou repo isolado

Reproduz os passos 3 a 7 à mão, com os scripts desta skill:

```bash
# na pasta que contém skill-<slug>/
bash scripts/check-skill.sh skill-<slug>                # gate verde antes de tudo
bash scripts/bump-skill.sh skill-<slug> <ver>           # só se a versão muda; na 1ª já nasce 0.1.0
bash scripts/pack-skill.sh skill-<slug> /caminho/fora   # zip fora do repo
cd skill-<slug>
git init -b main                                        # só na 1ª vez
git add -- <lista explícita de arquivos>                # nunca -A
git commit -m "release: v<ver>"
git tag -a v<ver> -m "v<ver>"
gh repo create schematizeme/skill-<slug> --public --source . --remote origin --push   # 1ª vez
git push --follow-tags                                  # demais vezes
gh release create v<ver> /caminho/fora/skill-<slug>.zip --title "v<ver>" --notes "<texto>"
```

Na 1ª versão a skill já nasce em `0.1.0` com `versions/v0.1.0/` criado pelo `bump-skill.sh` (ou
copiado à mão com os mesmos itens). Instale e prove: `bash skill-<slug>/install.sh "$HOME"`.

## 4. Depois do release

- No catálogo, `skills.json` e grafos já saem commitados e publicados pela ferramenta. Resta
  `tools/graph-meta.json` (entrada nova de skill nova), do dono do catálogo
  (`references/integracao-catalogo.md`): quem publica diz o que falta; não edita fora do escopo.
- Skills dependentes: mudou um piso na base? Os blocos herdados das outras skills mudam por sync, e
  cada uma precisa do seu release (minor) para o texto novo chegar a quem instalou.

## 5. Erros comuns

| Sintoma | Causa | Correção |
|---|---|---|
| lint: versão dessincronizada | bump manual esqueceu um lugar | `bump-skill.sh` |
| release recusa: árvore suja | arquivo solto na skill | commitar ou descartar antes; nunca `git add -A` para "limpar" |
| release recusa: origin à frente | outro commit no remoto | `git pull`, refaça o gate |
| gate global falha por causa de OUTRA skill | trabalho alheio em andamento | corrija a causa ou publique depois; não use `--skip-gate` sem registrar |
| zip instala em lugar nenhum | top-level errado | sempre `schematize-<slug>/` (use `pack-skill.sh`) |
| comando morto em quem já instalou | skill removeu comando | o `install.sh` poda; reinstale |
| release sem tag | push sem `--follow-tags` | `git push origin v<ver>` |

## 6. Segredo e permissão

O `gh` usa a credencial do usuário; nada de token em arquivo da skill. Release não carrega segredo:
o zip é público. `schematize-infra -> references/cadeia-suprimentos.md` cobre assinatura e proveniência
de artefato, quando o catálogo decidir assinar releases.
