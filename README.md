ems-meta
qual seria a melhor forma de fazer o rollback de uma versao implantada em producao com base no modelo de branch adotado abaixo e levando em consideracao que utilizamos ARO, ARGOCD, githubactions:
# MODELO DE BRANCHING – MANUAL OPERACIONAL COMPLETO (GITFLOW SIMPLIFICADO COM BRANCH DEVELOPMENT)

Este documento descreve **de forma extremamente detalhada**, passo a passo, o modelo de branching adotado no Modelo B (com branch `development`), explicando **não apenas o que deve ser feito**, mas **por que deve ser feito**, permitindo que até um estagiário sem experiência prévia em Git compreenda o fluxo.

O objetivo é que este README.md sirva como **guia oficial** para uso em repositórios que seguem este modelo de versionamento.

---

# 1. RESUMO DO MODELO DE BRANCHING

Este modelo utiliza três níveis de branches permanentes ou semi-permanentes:

| Branch          | Propósito                                                                         | Deploy    |
| --------------- | --------------------------------------------------------------------------------- | --------- |
| **main**        | Versão estável. Sempre representa o que está ou estará em produção.               | prd       |
| **development** | Branch de integração da sprint. Todas as features são unificadas e testadas aqui. | dev / hml |
| **release/**    | Criada no final da sprint para preparar a release.                                | hml       |
| **feature/**    | Trabalho individual de desenvolvimento                                            | —         |
| **bugfix/**     | Correções encontradas em QA/HML                                                   | —         |
| **hotfix/**     | Correções emergenciais em produção                                                | prd       |
| **tech/**       | Refactors e melhorias técnicas                                                    | —         |

O fluxo de desenvolvimento gira em torno do ciclo:

```
feature -> development -> release -> main
```

---

# 2. FUNDAMENTAÇÃO DO MODELO

Este modelo foi escolhido porque:

1. **Agiliza a integração**: todos os desenvolvedores integram suas branches em `development`, evitando conflitos tardios.
2. **Permite testes seguros**: `development` pode ser implantada em DEV e HML sem risco de levar código incompleto para produção.
3. **Facilita a preparação de releases** através da branch `release/x.y.z`.
4. **Viabiliza hotfixes limpos** sem interferir no trabalho da sprint.
5. **Permite rollback declarativo** via tag + ArgoCD.

---

# 3. ENTENDENDO OS CONCEITOS DO GIT

Antes do fluxo operacional, é essencial entender o que algumas operações significam.

## 3.1. O que é uma branch?

Uma branch é apenas um ponteiro para um commit específico. Ela representa uma linha de desenvolvimento.

## 3.2. O que é HEAD?

HEAD é o ponteiro que indica **onde você está trabalhando no momento**.

## 3.3. O que é um merge?

Merge une o histórico de duas branches e cria um commit que indica a junção.

## 3.4. O que é um PR (Pull Request)?

PR é um processo de revisão antes de executar um merge.

## 3.5. O que é tag?

Uma tag é um "marcador" fixo em um commit, utilizada normalmente para identificar releases.

---

# 4. FLUXO OPERACIONAL COMPLETO

## 4.1. FLUXO DE FEATURE – PASSO A PASSO

### Objetivo

Desenvolver uma nova funcionalidade de maneira isolada, segura e rastreável.

### Passos Detalhados

### Step 1 – Atualizar a branch main localmente

```bash
git checkout main
git pull origin main
```

Por que fazemos isso?

* Para garantir que estamos trabalhando sobre a versão mais recente do código.
* Para evitar conflitos futuros.

---

### Step 2 – Atualizar a branch development

```bash
git checkout development
git pull origin development
```

Motivação:

* A feature deve nascer a partir da última versão integrada da sprint.

---

### Step 3 – Criar a branch de feature

```bash
git checkout -b feature/SIM-101-melhora-simulacao-parcelas
```

Motivação:

* Uma branch dedicada evita que mudanças inacabadas atrapalhem o time.
* Permite versionamento organizado.

---

### Step 4 – Desenvolver a funcionalidade

Aqui o desenvolvedor:

* Implementa código
* Cria testes
* Atualiza documentação
* Valida regras de negócio
* Executa testes localmente

Commits devem ser pequenos, exatos e contextualizados:

```bash
git add .
git commit -m "feat: melhora cálculo de simulação de parcelas"
```

Se precisar escrever um commit mais detalhado:

```bash
git commit
```

Editor abre e você descreve:

```
feat: adiciona nova metodologia de cálculo
- aplica regra de mínimos
- corrige arredondamento
- adiciona testes unitários
```

---

### Step 5 – Enviar branch para o repositório remoto

```bash
git push -u origin feature/SIM-101-melhora-simulacao-parcelas
```

Motivação:

* Permite abertura do Pull Request.
* Salva o progresso no servidor.

---

### Step 6 – Abrir Pull Request para development

Motivação:

* PR garante revisão técnica.
* CI é executado obrigatoriamente.

Checklist do PR:

* Descrição completa
* Link para o ticket
* Impactos
* Como testar
* Mudanças relevantes

---

### Step 7 – CI executa validações

CI executa:

* Build Maven
* Testes unitários
* Análise estática
* Segurança

Se falhar, o PR **não pode** ser aprovado.

---

### Step 8 – Code Review

O revisor verifica:

* Padrões de código
* Arquitetura
* Impacto em outras áreas
* Testes e cobertura

Após aprovação, segue para merge.

---

### Step 9 – Merge em development

Executado via Squash & Merge.

Por quê?

* Para manter histórico limpo.
* Porque cada feature vira um único commit em development.

---

### Step 10 – Deploy automático para dev

Após o merge, uma action GitHub pode gerar uma tag automática do tipo:

```
v2.6.0-dev.001
```

ArgoCD consome o repositório de manifests e atualiza o ambiente dev.

---

## 4.2. FLUXO DE RELEASE

### Step 1 – Criar a branch de release no final da sprint

```bash
git checkout development
git pull origin development
git checkout -b release/2.6.0
```

Motivação:

* Congelar escopo da sprint
* A partir daqui, apenas correções podem entrar

---

### Step 2 – Deploy automático em hml

Action gera tag:

```
v2.6.0-rc.001
```

ArgoCD deploya em hml.

---

### Step 3 – Correções de homologação

Correções criadas em branches:

```bash
git checkout -b bugfix/HML-450-ajuste-liberacao
```

E mescladas em release/2.6.0.

---

### Step 4 – Aprovação de release

Com HML validado, merge para main:

```bash
git checkout main
git merge release/2.6.0
```

Action criando tag final:

```
v2.6.0
```

ArgoCD aplica em produção.

---

# 4.3. FLUXO DE HOTFIX

Existem dois cenários:

## Cenário 1 – main = produção

```bash
git checkout main
git pull origin main
git checkout -b hotfix/INC-777-erro-liberacao
```

Após correção:

```bash
git push -u origin hotfix/INC-777-erro-liberacao
```

Abrir PR para main.

Após merge, tag:

```
v2.6.1
```

Deploy PRD.

---

## Cenário 2 – main contém features que não podem ir para PRD

Criar hotfix a partir da tag da produção:

```bash
git checkout v2.6.0
git checkout -b hotfix/INC-777-erro-liberacao
```

Após correção, criar tag:

```
v2.6.1
```

Deploy PRD.

PR deve ser mergeado em main posteriormente.

---

# 5. ROLLBACK COMPLETO VIA ARGOCD

Rollback via ArgoCD deve ser sempre preferido em vez de tentar revert manualmente.

## Passos

1. Acessar ArgoCD
2. Selecionar a aplicação
3. Ir em "History and Rollback"
4. Identificar versão estável
5. Clicar em Rollback
6. Validar aplicação no cluster

ArgoCD restaura:

* imagem
* variáveis
* réplicas
* configurações de deployment

---

# 6. DIAGRAMA COMPLETO EM MERMAID (MODELO B)

```mermaid
gitGraph
    commit id: "start-main" tag: "v2.5.1"
    branch development
    commit id: "sprint-start"

    %% FEATURE – Simulação
    branch feature/SIM-101-melhora-simulacao-parcelas
    commit id: "sim1-1"
    commit id: "sim1-2"
    checkout development
    merge feature/SIM-101-melhora-simulacao-parcelas tag: "merge-sim1"

    %% FEATURE – Formalização
    branch feature/FORM-212-assinatura-eletronica
    commit id: "form1-1"
    commit id: "form1-2"
    checkout development
    merge feature/FORM-212-assinatura-eletronica tag: "merge-form1"

    %% Bugfix HML/QA
    branch bugfix/ANL-309-corrige-score-calculo
    commit id: "bugfix-anl-1"
    checkout development
    merge bugfix/ANL-309-c*/

```
---

gitGraph
    commit id: "start-main" tag: "v2.5.1"
    branch development
    commit id: "sprint-start"

    %% ============================================
    %% Feature 1 – Simulação de crédito
    %% ============================================
    branch feature/SIM-101-melhora-simulacao-parcelas
    commit id: "sim1-1"
    commit id: "sim1-2"
    checkout development
    merge feature/SIM-101-melhora-simulacao-parcelas tag: "merge-sim1"

    %% ============================================
    %% Feature 2 – Formalização
    %% ============================================
    branch feature/FORM-212-assinatura-eletronica
    commit id: "form1-1"
    commit id: "form1-2"
    checkout development
    merge feature/FORM-212-assinatura-eletronica tag: "merge-form1"

    %% ============================================
    %% Bugfix em QA / HML
    %% ============================================
    branch bugfix/ANL-309-corrige-score-calculo
    commit id: "bugfix-anl-1"
    checkout development
    merge bugfix/ANL-309-corrige-score-calculo tag: "merge-anl-bugfix"

    %% 🔵 deploy-dev — início dos testes integrados
    commit id: "deploy-dev"

    %% ============================================
    %% Tech branch – atualização de libs
    %% ============================================
    branch tech/PLAT-002-update-spring
    commit id: "tech1"
    checkout development
    merge tech/PLAT-002-update-spring tag: "merge-tech-spring"

    %% 🔶 deploy-hml — validação de negócio
    commit id: "deploy-hml"

    %% ============================================
    %% Criando release 2.6.0
    %% ============================================
    branch release/2.6.0
    commit id: "prepare-hml-release"

    %% Bugfix de homologação – liberação de valor
    branch bugfix/HML-450-ajuste-validacao-liberacao
    commit id: "hmlfix1"
    commit id: "hmlfix2"
    checkout release/2.6.0
    merge bugfix/HML-450-ajuste-validacao-liberacao tag: "merge-hml-fixes"

    %% 🔶 deploy-hml — RC validado
    commit id: "deploy-hml-rc"

    %% ============================================
    %% Merge para produção
    %% ============================================
    checkout main
    merge release/2.6.0 tag: "v2.6.0"

    %% 🔴 deploy-prd — release em produção
    commit id: "deploy-prd"

    %% ============================================
    %% Hotfix em produção
    %% ============================================
    branch hotfix/INC-777-erro-liberacao-valor
    commit id: "hotfix1"
    checkout main
    merge hotfix/INC-777-erro-liberacao-valor tag: "v2.6.1"

    %% 🔴 deploy-prd — hotfix aplicado
    commit id: "deploy-prd-hotfix"
