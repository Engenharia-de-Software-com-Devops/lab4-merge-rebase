# Laboratório 4 — Merge, Rebase e Resolução de Conflitos

Integrar trabalho de branches diferentes: merge com e sem conflito, rebase, e conflito em Pull Request resolvido localmente.

> O Exercício 8 (conflito de PR resolvido **pela interface web**) está em repositórios separados: [lab4-ex08-pr-interface](https://github.com/Engenharia-de-Software-com-Devops/lab4-ex08-pr-interface) e [lab4-ex08-pr-interface-v2](https://github.com/Engenharia-de-Software-com-Devops/lab4-ex08-pr-interface-v2).

## Sumário
1. [Conceitos](#1-conceitos)
2. [Histórico final](#2-histórico-final)
3. [Etapa 1 — Merge sem conflitos](#etapa-1--merge-sem-conflitos)
4. [Etapa 2 — Provocando e resolvendo conflitos](#etapa-2--provocando-e-resolvendo-conflitos)
5. [Etapa 3 — Rebase](#etapa-3--rebase)
6. [Etapa 4 — Conflito em Pull Request](#etapa-4--conflito-em-pull-request)
7. [Merge × Rebase](#merge--rebase)
8. [Revisão rápida](#revisão-rápida)

## 1. Conceitos

| Termo | O que é |
|---|---|
| **Fast-forward** | A `main` não andou: o merge só avança o ponteiro e não cria commit de merge |
| **Merge de três vias** | As duas branches andaram: o Git compara o ancestral comum com as duas pontas e cria um **commit de merge com dois pais** |
| **Conflito** | As duas branches alteraram o **mesmo trecho**. O Git não escolhe sozinho e para o merge |
| **Rebase** | Reaplica os commits da branch em cima da `main` atual: histórico linear, **hashes novos** |
| **`--abort`** | Cancela um merge ou rebase em andamento e volta ao estado anterior |

Marcadores de conflito:
```text
<<<<<<< HEAD                 <- versão da branch em que você está (main)
versao=3
=======                      <- separador
versao=2
>>>>>>> feature/configuracao <- versão da branch que está entrando
```

## 2. Histórico final

```text
*   4aa4655 merge: resolve conflito do pull request        <- Etapa 4
|\
| * 32401af docs: atualiza descricao principal             <- main (lado conflitante)
* | 90aa066 docs: altera descricao do projeto              <- feature/pr-conflito
|/
* a08e271 chore: adiciona atualizacao na main              <- Etapa 3
*   70e712c merge: resolve conflito de configuracao        <- Etapa 2
|\
| * 4e7c2e0 feat: atualiza versao para 2                   <- feature/configuracao
* | b9656df chore: atualiza versao para 3                  <- main
|/
* 0e0985f chore: adiciona configuracao inicial
* c0fcb06 docs: atualiza README                            <- Etapa 1 (fast-forward)
* ce0fbdb base: README inicial
```

Cada "bolha" (`|\` ... `|/`) é um ponto em que duas linhas divergiram e foram reunidas por um commit de merge.

## Etapa 1 — Merge sem conflitos

```bash
git switch -c feature/documentacao
# adiciona a seção "Documentacao" no README.md
git commit -am "docs: atualiza README"
git switch main
git merge feature/documentacao
```

**Resultado:** `c0fcb06` entrou direto na `main`, **sem commit de merge**.
**Por quê?** A `main` não tinha commits novos desde a criação da branch, então o Git fez **fast-forward** (só moveu o ponteiro).

## Etapa 2 — Provocando e resolvendo conflitos

| Passo | Branch | `config.txt` linha 2 | Commit |
|---|---|---|---|
| 1 | `main` | `versao=1` | `0e0985f` |
| 2 | `feature/configuracao` | `versao=2` | `4e7c2e0` |
| 3 | `main` | `versao=3` | `b9656df` |

```bash
git merge feature/configuracao
# CONFLICT (content): Merge conflict in config.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

**Resolução:** editei `config.txt` deixando `versao=3`, removi os marcadores e finalizei:
```bash
git add config.txt                                   # é o add que "marca como resolvido"
git commit -m "merge: resolve conflito de configuracao"   # 70e712c, com dois pais
```

**Respostas**
- *Por que o Git não resolveu sozinho?* As duas branches mudaram a **mesma linha** a partir do mesmo ancestral. Não há como saber qual intenção vale.
- *O que é `HEAD` no conflito?* O lado da branch atual (`main`). O outro lado é a branch que está sendo integrada.
- *Como o Git sabe que foi resolvido?* Quando o arquivo volta ao index com `git add`, sem marcadores.
- *Resolver × cancelar:* resolver conclui o merge com um commit; `git merge --abort` desiste e volta ao estado de antes. Ele só funciona **durante** o merge, por isso não foi executado depois da resolução.

## Etapa 3 — Rebase

```bash
git switch -c feature/rebase
# cria rebase.txt
git commit -m "feat: adiciona exemplo de rebase"
git switch main
# cria main-update.txt
git commit -m "chore: adiciona atualizacao na main"      # a08e271
git switch feature/rebase
git rebase main
```

**O que acontece:** o commit da `feature/rebase` é **reaplicado** em cima de `a08e271`. O conteúdo é o mesmo, mas o **hash muda**, porque o pai mudou. Não é criado nenhum commit de merge e o histórico fica linear.

> A branch `feature/rebase` foi usada só localmente para observar o efeito e não foi enviada ao GitHub. Por isso só `a08e271` aparece no histórico remoto.

Durante um rebase com conflito: resolver, `git add <arquivo>`, `git rebase --continue`. Para desistir: `git rebase --abort`.

## Etapa 4 — Conflito em Pull Request

```bash
git switch -c feature/pr-conflito
# altera a linha "Descricao:" do README
git commit -am "docs: altera descricao do projeto"     # 90aa066
git push -u origin feature/pr-conflito                 # PR aberto para main
git switch main
# altera a MESMA linha de outro jeito
git commit -am "docs: atualiza descricao principal"    # 32401af
git push origin main                                   # o PR passa a ter conflito
```

**Resolução local**, trazendo a `main` para dentro da branch do PR:
```bash
git fetch origin
git switch feature/pr-conflito
git merge origin/main                                  # conflito no README.md
# edita, remove marcadores
git add README.md
git commit -m "merge: resolve conflito do pull request"    # 4aa4655
git push origin feature/pr-conflito
```

**Resultado:** o PR recebeu o commit de resolução e o GitHub deixou de acusar conflito. Depois o PR foi integrado, e a `main` aponta para `4aa4655`.

## Merge × Rebase

| | Merge | Rebase |
|---|---|---|
| Histórico | Preserva a divergência (bolhas) | Linear |
| Commits existentes | Intactos | Reescritos (hashes novos) |
| Commit extra | Commit de merge | Nenhum |
| Branch compartilhada | Seguro | **Evitar**: quem já baixou a branch fica com um histórico diferente do remoto |

## Revisão rápida
- [ ] Sei quando o Git faz fast-forward e quando cria commit de merge.
- [ ] Sei ler os marcadores `<<<<<<<` / `=======` / `>>>>>>>`.
- [ ] Sei que `git add` marca o conflito como resolvido.
- [ ] Sei por que o rebase muda o hash dos commits.
- [ ] Sei resolver um conflito de PR localmente: `fetch`, `merge origin/main`, `push`.

## Referências
- [git-merge](https://git-scm.com/docs/git-merge) · [git-rebase](https://git-scm.com/docs/git-rebase)
- [Pro Git — Branching e Merging](https://git-scm.com/book/pt-br/v2/Ramifica%C3%A7%C3%A3o-Branching-no-Git-O-b%C3%A1sico-de-Ramifica%C3%A7%C3%A3o-Branch-e-Mesclagem-Merge)

---

## Arquivo original do exercício

O conteúdo abaixo é o README usado nos exercícios. A linha `Descricao:` é a que gerou o conflito da Etapa 4.

# Projeto Lab4
Descricao: projeto principal do laboratorio 4.

## Documentacao
Como usar o projeto.
