# Fork ZipDev do bmad-loop

Este é um fork do [bmad-loop](https://github.com/bmad-code-org/bmad-loop) mantido pela Softlink
para o projeto [ZipDev](https://github.com/softlinkbrasil/zipdev). Existe por causa de um problema
de compatibilidade real entre duas peças do ecossistema BMad que evoluem de forma independente:

- O **bmad-loop** (este projeto) sabe rodar em "modo stories" (`stories.yaml`), despachando cada
  story com o texto `/<dev-skill> Spec folder: X. Story id: Y.` — um formato introduzido pela
  PR [bmad-code-org/BMAD-METHOD#2549](https://github.com/bmad-code-org/BMAD-METHOD/pull/2549)
  (mergeada em 06/07/2026).
- O **BMAD-METHOD** (o `bmad-build-auto`, skill que o bmad-loop dispara) evoluiu depois disso:
  entre 11 e 26/09/2026 ele substituiu esse mecanismo pelo sistema de "árvore de tickets"
  (`tickets.toml` + `tickets.py find <ref>`). A skill `bmad-build-auto` publicada hoje não
  reconhece mais o formato "Spec folder / Story id" — só referências de ticket.

Resultado: o modo stories do bmad-loop, do jeito que está publicado, não consegue mais disparar o
`bmad-build-auto` atual. `bmad-loop validate` recusa com `FAIL: bmad-build-auto lacks folder+id
dispatch`. Confirmado na prática em 28/09/2026 (ver `docs/roadmap.md` no repositório do ZipDev).

## O patch

Branch `zipdev-patch`, em cima do `upstream/main`. Dois arquivos, ~20 linhas:

- `src/bmad_loop/stories_engine.py` (`_stories_dev_prompt`) e `src/bmad_loop/cli.py`
  (`_dry_run_stories`): trocam o texto de despacho de `Spec folder: X. Story id: Y.` para
  `Ticket <id> in <pasta-do-epico> (resolve with tickets.py find).` — um formato que o
  `bmad-build-auto` atual já sabe resolver via `tickets.py find <pasta> <id>`.
- `src/bmad_loop/install.py`: o preflight que bloqueia o modo stories (`missing_stories_support`)
  procurava o texto `"folder+id dispatch"` no `step-01-clarify-and-route.md` instalado — texto que
  não existe mais na skill atual. Trocado para procurar `"tickets.py"`, que é o marcador do
  mecanismo que o patch acima realmente usa.

Validado de ponta a ponta em 28/09/2026: `bmad-loop validate` passa, `bmad-loop run` despacha uma
sessão real do Claude Code, o `bmad-build-auto` (publicado, sem nenhuma alteração) resolve o ticket
corretamente via `tickets.py find`, completa dev → revisão → commit real.

## Como atualizar este fork

```bash
cd /home/celso/bmad-loop-zipdev
git fetch upstream
git checkout zipdev-patch
git rebase upstream/main
# resolver conflitos se o upstream tiver mexido nos mesmos trechos
```

Um conflito aqui é esperado eventualmente — os textos que patcheamos já mudaram de nome/lugar uma
vez neste projeto. Se o upstream corrigir o problema por conta própria (o ideal), este fork vira
descartável: voltamos a instalar o bmad-loop oficial via `uv tool install bmad-loop`.

## Como instalar/usar

```bash
uv tool install --editable /home/celso/bmad-loop-zipdev
# ou, sem tool install, isolado:
uv run --project /home/celso/bmad-loop-zipdev bmad-loop <comando>
```

## Publicação

Ainda não publicado no GitHub (sem `gh` CLI nem chave SSH configurada para GitHub nesta VPS no
momento da criação deste fork). Repositório local em `/home/celso/bmad-loop-zipdev`, remotes:
`upstream` = `https://github.com/bmad-code-org/bmad-loop.git`, sem `origin` ainda.
