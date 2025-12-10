# ⚙️ Módulo Composto (CSU — “Composite Service Unit”)

Significa: **uma unidade de processo que coordena múltiplos MFUs**, garantindo que operações distribuídas aconteçam de forma ordenada, consistente e segura.

---

# CSU — Módulo Composto

Um **CSU (Composite Service Unit)** é a segunda camada da plataforma, construída sobre os MFUs.
Enquanto o MFU representa a **unidade mínima de negócio**, o CSU representa a **unidade mínima de processo**, responsável por conectar diferentes domínios para executar fluxos que dependem de mais de um MFU.

Em outras palavras:

> Um CSU é o responsável por **orquestrar** operações entre múltiplos MFUs, garantindo que cada etapa do processo aconteça da maneira correta, na ordem certa e com mecanismos de compensação caso algo dê errado.

Assim como o MFU é um “tijolo estrutural”, o CSU funciona como o **encaixe entre esses tijolos** — permitindo construir processos maiores sem violar a independência dos domínios.

Ele não possui regras de negócio primárias.
Ele não guarda lógica que pertence aos MFUs.
Seu papel é: **coordenar, ordenar e garantir integridade entre serviços distintos**.

---

# Por que o CSU existe? (Motivação)

Em um ecossistema de microserviços, cada MFU:

* tem sua própria base de dados
* controla seu próprio domínio
* evolui de forma independente

Isso é excelente do ponto de vista de arquitetura, mas cria um desafio importante:

> Como garantir a consistência de um processo que depende de **vários MFUs**, cada um com seu estado, sua base e suas regras?

Um exemplo simples:

1. O MFU de Crédito valida a simulação.
2. O MFU de Formalização gera e registra o contrato.
3. O MFU de Liberação libera o valor.

Se qualquer etapa falhar, o sistema precisa:

* desfazer ações anteriores
* manter os dados coerentes
* registrar o estado correto do processo

É aqui que o CSU se torna essencial.

Ele implementa o padrão **Saga**, garantindo que processos distribuídos funcionem como uma “transação lógica”, mesmo quando os MFUs não compartilham a mesma base de dados.

---

# Tipos de CSU (Cenários de uso)

Existem dois tipos principais de CSU, escolhidos de acordo com a complexidade e criticidade do processo que está sendo executado.

---

## a) CSU Stateless — Chamadas coordenadas sem persistir estado

Modelo simples e direto.

O CSU Stateless funciona como um **encadeador de etapas**, chamando múltiplos MFUs de maneira sequencial, mas sem armazenar informações internas sobre o andamento do fluxo.

```
              +-------->[mfu-a] ----->[schema]
[csu-stateless]
              +-------->[mfu-b]------>[schema]
```

Quando utilizar:

* processos rápidos
* integrações sem efeitos colaterais complexos
* operações que não exigem rollback elaborado

Limitação:

* se algo falhar no meio, o CSU Stateless não possui histórico para desfazer etapas anteriores

---

## b) CSU Stateful — Consistência completa com registro de etapas

O CSU Stateful é a implementação mais robusta.

Ele **persiste o estado do processo**, registrando o andamento de cada etapa, o que foi enviado, o que foi concluído e o que precisa ser compensado caso algo dê errado.

```
             +-------->[mfu-a] ----->[schema]
             |
[csu-stateful]--->[estado interno]
             |
             +-------->[mfu-b]------>[schema]
```

Usado quando:

* existe risco de inconsistência entre domínios
* o processo tem alto impacto financeiro ou regulatório
* rollback é obrigatório
* o processo precisa ser retomado após falhas

Benefícios:

* garante consistência de ponta a ponta
* permite reprocessamento
* oferece rastreabilidade total
* reduz impacto de falhas parciais

---

# Comunicação com serviços externos

CSUs também podem integrar MFUs com APIs externas.
Exemplo para stateless:

```
              +-------->{external api}
[csu-stateless]
              +-------->[mfu-b]------>[schema]
```

E para stateful:

```
             +-------->{external api}
             |
[csu-stateful]--->[estado]
             |
             +-------->[mfu-b]------>[schema]
```

Aqui o CSU garante que a lógica de integração não fique espalhada pelos MFUs, preservando seus domínios.

---

# Comunicação com eventos e filas

CSUs podem enviar ou consumir eventos, o que reduz acoplamento e melhora escalabilidade.

```
(evento)---consome->[csu-stateless]---publica-->(evento)
```

E na versão stateful:

```
                          +----->[estado]
                          |
(evento)---consome->[csu-statefull]---publica-->(evento)

```

Essa abordagem é recomendada quando:

* os MFUs não podem ficar dependentes de respostas imediatas
* existem partes do processo que são naturalmente assíncronas
* é necessário reduzir o impacto de chamadas síncronas em cadeia

---

# Comunicação com sistemas legados (via ACL)

Quando é necessário falar com sistemas antigos, o CSU nunca chama o legado diretamente.
Para isso existe a camada **ACL — Anti-Corruption Layer**, que traduz chamadas e formatos.

```
[csu-stateless] ----->[acl]------>[legado]---->[schema]
```

E na versão stateful:

```
      +---->[estado]
      |
[csu-stateful] ----->[acl]------>[legado]---->[schema]
```

Assim:

* MFUs continuam protegidos de tecnologias antigas
* mudanças no legado não quebram os domínios modernos
* o CSU mantém a responsabilidade do fluxo

---

# Resumo das Regras Fundamentais do CSU

| Regra                                            | Explicação                                                           |
| ------------------------------------------------ | -------------------------------------------------------------------- |
| **CSU coordena MFUs, mas não implementa regras** | Regras de negócio pertencem ao MFU; CSU apenas conduz o processo.    |
| **Pode ser stateless ou stateful**               | Dependendo da necessidade de rollback e consistência entre domínios. |
| **Nunca acessa bases de MFUs diretamente**       | Toda interação ocorre via APIs dos MFUs, preservando isolamento.     |
| **Evita acoplamento ponto a ponto**              | Pode usar eventos para reduzir dependências síncronas.               |
| **É a segunda camada da plataforma**             | Conecta MFUs sem ferir autonomia de domínio.                         |

---

# Por que isso importa?

Podemos imaginar um CSU como **o maestro de uma orquestra**:

* os instrumentos são os MFUs
* cada um toca sua parte, no seu tempo
* mas é o maestro que garante que tudo aconteça na ordem correta
* e se um instrumento falha, o maestro ajusta a performance para manter a harmonia

Sem o CSU:

* processos que envolvem vários domínios se tornam confusos
* erros podem deixar sistemas em estados inconsistentes
* cada MFU teria que conhecer outros MFUs, criando acoplamento irregular
* fluxos de compensação ficam espalhados e difíceis de manter

Com CSU:

* processos complexos tornam-se previsíveis
* a plataforma consegue crescer sem virar uma teia de dependências
* regras permanecem nos MFUs, enquanto orquestração fica isolada
* há rastreabilidade, consistência e segurança operacional

O CSU permite que a empresa construa soluções grandes **com disciplina, modularidade e evolução controlada**, sem sacrificar a autonomia dos domínios fundamentais.
