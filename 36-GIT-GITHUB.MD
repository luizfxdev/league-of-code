# Git e GitHub — Guia Completo para Devs Java

---

## 1. O que é Git?

Git é um **sistema de controle de versão distribuído**. Em vez de salvar `ProjetoFinal_v2_AGORA_v3.zip`, você registra cada evolução do código como um **commit** — um snapshot rastreável com autor, data e mensagem.

```
e8aab78 → 38k2ane → c9g57ef → k2f89m3
(commit1)  (commit2)  (commit3)  (commit4)
```

Cada commit tem um hash único. Você pode voltar a qualquer ponto do histórico, comparar versões e colaborar sem sobrescrever o trabalho de outros.

---

## 2. O que é GitHub?

GitHub é um **serviço de hospedagem de repositórios Git remotos**. Funciona como a cópia "oficial" do projeto, acessível pela equipe.

| Característica | Detalhe |
|---|---|
| Interface | github.com |
| Modelo | Gratuito para open-source, pago para privado |
| Alternativas | GitLab, BitBucket |
| Uso profissional | Revisão de PR, histórico de mudanças, CI/CD |

> **No seu dia a dia:** o repositório `gwsistemas/gweb` no GitHub é o remoto. O que você tem na sua máquina é o local.

---

## 3. Repositório local vs. remoto

```
GITHUB (remoto)    ←── push ───  SUA MÁQUINA (local)
                   ──── pull ──→
```

O fluxo básico sempre segue essa lógica:
1. Você **clona** ou **puxa** a versão mais recente do remoto
2. Faz suas alterações localmente (commits)
3. **Empurra** de volta para o remoto

---

## 4. Configuração inicial

### 4.1 Instalar Git

```bash
# Download: https://git-scm.com/downloads
# Verifique:
git --version
```

> Na instalação Windows: **não escolha** "Git Credential Manager" — selecione **None**.

### 4.2 Identificação global

```bash
git config --global user.name "Luiz Felipe de Oliveira"
git config --global user.email "seu@email.com"
git config --list
```

Isso define o autor dos seus commits. Relevante: se você já commitou com dois nomes diferentes (`luizfxdev` e `Luiz Felipe de Oliveira`), ambos aparecem no `git log`. Isso pode dificultar filtrar commits por `--author=` — lembre-se disso ao tentar isolar seus commits num branch poluído.

---

## 5. Chave SSH

O GitHub não aceita mais autenticação só com usuário/senha. A solução é SSH: você registra uma chave pública no GitHub e sua máquina autentica automaticamente.

```bash
# Gerar chave
ssh-keygen -t ed25519 -C "seu@email.com"

# Ver a chave pública gerada
cat ~/.ssh/id_ed25519.pub
```

Cole o conteúdo em: **GitHub → Settings → SSH and GPG keys → New SSH key**

```bash
# Testar conexão
ssh -T git@github.com
# Esperado: "Hi seuusuario! You've successfully authenticated..."
```

---

## 6. Primeira versão de um projeto no GitHub

```bash
git init
git add .
git commit -m "feat: estrutura inicial do projeto"
git branch -M main
git remote add origin git@github.com:seuusuario/seurepositorio.git
git push -u origin main
```

> O `-u` no primeiro push define `origin main` como upstream — nos próximos, basta `git push`.

---

## 7. Ciclo de vida dos arquivos

```
Untracked / Modified / Deleted
           ↓  git add
         Staged
           ↓  git commit
        Committed
           ↓  git push
         Remoto
```

### Comandos do ciclo

```bash
git status                  # ver estado atual dos arquivos
git add .                   # adicionar tudo ao stage
git add src/main/Foo.java   # adicionar arquivo específico
git commit -m "mensagem"    # criar o commit
git push                    # enviar para o remoto
```

---

## 8. Clonar e trabalhar num repositório existente

```bash
git clone git@github.com:gwsistemas/gweb.git
cd gweb

# Fazer alterações...

git add .
git commit -m "fix: corrige validação de km_milhao"
git push
```

---

## 9. Branches

Branch é uma linha paralela de desenvolvimento. O padrão do time no `gweb` usa branches de feature/fix separados do `main`/`fix`.

```bash
git branch                        # listar branches locais
git branch -a                     # incluir remotos
git checkout -b feature/6874      # criar e mudar para novo branch
git checkout fix                  # mudar para branch existente
git push -u origin feature/6874   # subir branch novo para o remoto
```

---

## 10. Sincronizar com o remoto

```bash
git fetch origin          # baixa atualizações sem mesclar
git pull origin fix       # fetch + merge do branch fix
git pull --rebase         # fetch + rebase (histórico mais limpo)
```

> **Importante:** sempre faça `git fetch origin fix` antes de um `git reset --hard origin/fix`. Seus colegas podem ter feito force-push no branch durante a sessão — se você resetar para uma referência desatualizada, vai perder commits deles.

---

## 11. Histórico de versões

```bash
git log                        # histórico completo
git log --oneline              # uma linha por commit
git log --oneline --graph      # com grafo de branches
git log --oneline --author="luizfxdev"   # filtrar por autor
```

Exemplo de saída:
```
a3f12c8 fix: corrige fan-out no LEFT JOIN de OrdemServicoDAO
d8b091e feat: adiciona colunas status_estoque e quantidade_total_fisica
c4e2177 fix: encoding 'Disponível' via chr(237)
```

---

## 12. git diff

Mostra exatamente o que mudou antes de commitar.

```bash
git diff                        # mudanças não staged
git diff --staged               # mudanças já no stage
git diff HEAD~1                 # comparar com o commit anterior
git diff abc123 def456          # comparar dois commits
```

No VS Code, a aba **Source Control** exibe o diff graficamente — mais legível para arquivos `.jrxml` e `.jsp`.

---

## 13. git checkout — voltando no tempo

Permite inspecionar o estado do projeto em um commit anterior **sem alterar** o branch atual.

```bash
git checkout c9g57ef            # vai para aquele commit (detached HEAD)
git checkout HEAD~2             # vai 2 commits atrás
git checkout main               # volta para o branch atual
```

> **Antes de fazer checkout**, garanta que não há mudanças não salvas. Se acidentalmente modificou algo:

```bash
git reset
git clean -df
git checkout -- .
```

Esse trio descarta tudo que não foi commitado. Use com cuidado — é destrutivo.

---

## 14. .gitignore

Lista arquivos e pastas que o Git deve ignorar completamente.

```gitignore
# Compilados Java
*.class
*.jar
target/

# IDE
.idea/
*.iml
.vscode/

# SO
.DS_Store
Thumbs.db

# Logs
*.log
```

### Ignorar mas precisar forçar o add

No `gweb`, arquivos `.jasper` (compilados do JasperReports) ficam no `.gitignore` por padrão. Para incluí-los quando necessário:

```bash
git add -f relatorio.jasper
```

O `-f` força o rastreamento mesmo que o arquivo esteja no `.gitignore`.

### Recuperar arquivo deletado pelo GitHub UI

Se você deletou um arquivo pela interface do GitHub (como aconteceu com `RelatorioControlador.java` na Demanda 6773):

```bash
git fetch origin fix
git checkout origin/fix -- src/main/java/com/gw/controlador/RelatorioControlador.java
```

Isso restaura o arquivo do estado do branch remoto sem afetar o restante do working tree.

---

## 15. Cherry-pick — Reaplicar commits seletivamente

`git cherry-pick` pega um commit específico de qualquer lugar do histórico e o aplica no branch atual. É a ferramenta principal para limpar branches poluídos no `gweb`.

### O que é cherry-pick, visualmente

```
branch-base:   A → B → C → D
                         ↑
minha-branch:  A → B → C → X (commit seu) → Y (commit de outro dev)
                                 cherry-pick X
                                 ↓
branch-limpo:  A → B → C → D → X'
```

`X'` é seu commit reaplicado sobre a base atualizada.

---

### Fluxo completo — reset + cherry-pick

Este é o fluxo real usado no `gweb` para limpar um PR poluído com commits de outros devs.

#### 0. Faça backup antes de tudo

```bash
git checkout minha-branch
git branch backup-minha-branch
```

Se algo der errado: `git checkout backup-minha-branch`

#### 1. Atualizar a base

```bash
git checkout fix
git fetch origin
git reset --hard origin/fix
```

> `origin/fix` é o remoto atualizado. Seus colegas podem ter feito force-push — sempre `fetch` antes do `reset`.

#### 2. Ir para o branch do PR

```bash
git checkout minha-branch
git branch -v
```

#### 3. Resetar o branch do PR para a base limpa

```bash
git reset --hard fix
```

O branch agora está igual à base. Seus commits ainda existem no `backup-minha-branch`.

#### 4. Identificar seus commits

```bash
# Ver só os commits que existem no backup mas não na base
git log --oneline fix..backup-minha-branch
```

Saída exemplo:
```
c3c4c5c feat: adiciona filtro por local_estoque_id
a7a8a9a fix: corrige encoding Disponível via chr(237)
e1e2e3e raw: adiciona script SQL de migração cst_ibs_cbs
```

> **Atenção:** o `git log` lista do mais novo ao mais antigo. O cherry-pick deve ser feito do **mais antigo para o mais novo** — ou seja, de baixo para cima nessa listagem.

#### 5. Reaplicar com cherry-pick

**Um único commit:**
```bash
git cherry-pick abc1234
```

**Vários commits consecutivos (mais eficiente):**
```bash
# Sintaxe: <hash-mais-antigo>^..<hash-mais-novo>
git cherry-pick e1e2e3e^..c3c4c5c
```

**Commits não consecutivos — um por um, do mais antigo ao mais novo:**
```bash
git cherry-pick e1e2e3e
git cherry-pick a7a8a9a
git cherry-pick c3c4c5c
```

#### 6. Resolver conflitos (se houver)

```bash
# 1. Edite os arquivos com conflito
# 2. Marque como resolvido
git add .
# 3. Continue
git cherry-pick --continue
```

Para desistir e desfazer tudo:
```bash
git cherry-pick --abort
```

#### 7. Conferir histórico

```bash
git log --oneline --decorate -7
```

Resultado esperado: seus commits no topo, commits da base abaixo.

#### 8. Subir o branch

```bash
git push --force-with-lease
```

> Use `--force-with-lease` em vez de `-f`. Ele faz force-push, mas **aborta** se alguém subiu algo no branch desde o seu último `fetch` — evita sobrescrever commits de outros sem querer.

---

### Por que `--author=` nem sempre funciona para filtrar

```bash
# Pode não retornar todos os seus commits:
git log --oneline --author="Luiz"

# Porque seus commits existem sob dois formatos:
# - "luizfxdev" (commits via GitHub UI ou config anterior)
# - "Luiz Felipe de Oliveira" (config atual)
```

Use `git log --oneline fix..backup-minha-branch` — filtra por diferença de histórico, não por autor.

---

## 16. Referenciar commits relativos a HEAD

```bash
HEAD        # último commit do branch atual
HEAD~1      # penúltimo
HEAD~2      # antepenúltimo
HEAD~N      # N commits antes do HEAD
```

Útil para desfazer o último commit sem perder as mudanças:

```bash
git reset --soft HEAD~1    # desfaz o commit, mantém arquivos no stage
git reset --mixed HEAD~1   # desfaz o commit, mantém arquivos mas tira do stage
git reset --hard HEAD~1    # desfaz o commit e descarta as mudanças (DESTRUTIVO)
```

---

## 17. Boas práticas de commit — Commits Semânticos

Commits semânticos tornam o histórico legível para o SM, o QA e qualquer dev que abrir o `git log`. A estrutura é:

```
<tipo>[escopo opcional]: <descrição curta no imperativo>
```

### Tipos e quando usar

| Tipo | Quando usar | Exemplo real no `gweb` |
|---|---|---|
| `feat` | Novo recurso ou funcionalidade | `feat: adiciona coluna cst_ibs_cbs_reg_id em UfIbsCbsDAO` |
| `fix` | Correção de bug | `fix: corrige fan-out no LEFT JOIN de OrdemServicoDAO` |
| `refactor` | Mudança de código sem alterar comportamento | `refactor: extrai lógica de filtro para método privado em RelatorioBO` |
| `perf` | Melhoria de performance | `perf: substitui LEFT JOIN por CTE com locais agrupados em vestoque_produto_frota` |
| `style` | Formatação, indentação, semicolons — sem lógica alterada | `style: corrige indentação em cadastrar_Saida.jsp` |
| `docs` | Mudanças em documentação — sem alterar código | `docs: atualiza README com instruções de deploy no Tomcat 9` |
| `test` | Criação ou alteração de testes — sem alterar código | `test: adiciona teste para validação de km_milhao no serviço preventivo` |
| `build` | Modificações em build e dependências | `build: atualiza versão do jasperreports no pom.xml` |
| `chore` | Tarefas de configuração e administração — sem alterar código | `chore: adiciona target/ e *.jasper ao .gitignore` |
| `ci` | Mudanças em pipelines de integração contínua | `ci: adiciona step de lint no workflow do GitHub Actions` |
| `raw` | Arquivos de configuração, dados, parâmetros | `raw: adiciona script SQL de migração da coluna cst_ibs_cbs` |
| `cleanup` | Remoção de código comentado ou trechos desnecessários | `cleanup: remove console.log e código morto em ordemServico.js` |
| `remove` | Exclusão de arquivos, diretórios ou funcionalidades obsoletas | `remove: deleta relatório antigo rel_estoque_v1.jrxml não utilizado` |

### Escopo (opcional mas útil)

Indica a área afetada — ajuda bastante em PRs grandes com múltiplos arquivos:

```
feat(estoque): adiciona filtro por local_estoque_id na view
fix(jasper): corrige parâmetro FILTRO_LOCAL_OSP no exits branch
refactor(dao): move query de certificados para método isolado
```

### Regras práticas

- Imperativo: **"corrige"**, não "corrigido" ou "corrigindo"
- Sem ponto final na descrição
- Máximo ~72 caracteres na linha do título
- Se precisar de mais contexto, deixe uma linha em branco e escreva no corpo:

```bash
git commit -m "fix: corrige encoding de Disponível em rel_estoque

Substituído 'Disponível' literal por 'Dispon' || chr(237) || 'vel'
para compatibilidade com encoding Latin-1 do banco em produção.
Demanda 6618."
```

> O SM e o QA leem essas mensagens na revisão do PR — uma boa mensagem elimina perguntas desnecessárias.

---

## 18. Fluxo completo — do zero ao PR

```bash
# 1. Atualizar base
git checkout fix
git pull origin fix

# 2. Criar branch da demanda
git checkout -b feature/6874-certificados-vencidos

# 3. Trabalhar... commits parciais ao longo do dia
git add .
git commit -m "fix: ajusta query de certificados vencidos no JasperReports"

# 4. Antes do PR, checar se o branch base evoluiu
git fetch origin fix
git rebase origin/fix    # ou git merge origin/fix

# 5. Subir o branch
git push -u origin feature/6874-certificados-vencidos

# 6. Abrir PR no GitHub apontando para fix (não para main)
```

---

## Referência rápida

```bash
git init                          # inicializa repositório
git clone <url>                   # clona repositório remoto
git status                        # estado atual
git add .                         # adiciona tudo ao stage
git add -f <arquivo>              # força add de arquivo ignorado
git commit -m "mensagem"          # cria commit
git push                          # envia commits para o remoto
git pull                          # puxa atualizações do remoto
git fetch origin                  # atualiza referências remotas sem mesclar
git log --oneline                 # histórico resumido
git diff                          # mudanças não staged
git checkout <branch>             # troca de branch
git checkout -b <branch>          # cria e troca
git reset --hard origin/<branch>  # reseta para estado do remoto
git cherry-pick <hash>            # aplica commit específico no branch atual
git push -f origin <branch>       # force-push (use com consciência)
git checkout origin/<branch> -- <arquivo>  # recupera arquivo do remoto
```