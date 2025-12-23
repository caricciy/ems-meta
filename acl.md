# 🛡️ Camada Anticorrupção (ACL — “Anti-Corruption Layer”)

Significa: **uma camada especializada que protege o domínio de negócio** contra sistemas externos, legados ou terceiros, **traduzindo contratos, modelos e comportamentos**, sem permitir que conceitos externos contaminem o modelo interno.

---

# ACL — Camada Anticorrupção

Uma **ACL (Anti-Corruption Layer)** é um **serviço independente**, posicionado **fora do domínio**, cuja responsabilidade é **intermediar toda a comunicação entre a plataforma e sistemas externos** que não seguem os mesmos princípios arquiteturais, semânticos ou tecnológicos.

No contexto da nossa arquitetura:

* MFUs concentram **regras, invariantes e dados do domínio**
* CSUs coordenam **processos distribuídos e jornadas completas**
* ACLs concentram **integrações complexas, instáveis ou semanticamente incompatíveis**

Em outras palavras:

> A ACL existe para que o domínio continue **coerente, previsível e evolutivo**, mesmo quando precisa se relacionar com sistemas que não são.

Ela é um padrão clássico do **Domain-Driven Design**, descrito por Eric Evans, cujo objetivo central é **preservar a integridade do modelo de domínio**, isolando-o do “mundo externo”.

---

# Por que a ACL existe? (Motivação)

Em um cenário ideal, todos os sistemas:

* falariam o mesmo idioma
* teriam contratos claros e estáveis
* evoluiriam de forma coordenada

Na prática, isso não acontece — especialmente em contextos como:

* saúde pública
* setor financeiro
* setor governamental
* ambientes regulados por LGPD
* ecossistemas com sistemas antigos e heterogêneos

Nesses ambientes, surgem integrações com:

* sistemas legados (Java 5, .NET Framework, bases antigas)
* vendors e parceiros externos
* contratos frágeis ou mal versionados
* payloads confusos e cheios de códigos mágicos
* semânticas divergentes para o mesmo conceito

Sem uma ACL, o efeito é quase sempre o mesmo:

* o domínio começa a absorver conceitos externos
* entidades passam a carregar campos “estranhos”
* regras internas passam a depender do comportamento do legado
* mudanças externas viram impacto direto no core

A ACL existe para **interromper esse efeito**, atuando como **barreira semântica e técnica**.

---

# O papel da ACL na arquitetura MFU / CSU

A regra é simples, clara e inegociável:

> **Nenhum MFU ou CSU se comunica diretamente com sistemas externos ou legados.**

Toda interação externa ocorre **exclusivamente por meio da ACL**.

Fluxo correto:

```
[MFU / CSU] → [ACL] → [Sistema Externo ou Legado]
```

Com isso:

* o domínio nunca conhece contratos externos
* o domínio nunca depende de tecnologias antigas
* mudanças externas ficam contidas na ACL
* o core permanece estável, testável e compreensível

A ACL funciona como um **escudo arquitetural**: tudo que é instável, externo ou difícil fica do lado de fora.

---

# O que a ACL faz na prática?

A ACL não é “apenas uma integração”.
Ela é um **componente arquitetural completo**, com responsabilidades bem definidas.

---

## Ponto único de acesso aos sistemas externos

A ACL centraliza todo o acesso a:

* sistemas legados
* APIs de terceiros
* bureaus
* serviços corporativos antigos
* integrações via arquivo, mensageria ou protocolos proprietários

Isso garante:

* governança de acesso
* controle de throughput
* padronização de chamadas
* isolamento de falhas

Nenhum outro serviço da plataforma acessa esses sistemas diretamente.

---

## Tradução semântica (anticorrupção real)

O coração da ACL é a **tradução de significado**.

Ela é responsável por:

* converter contratos externos em modelos compreensíveis
* eliminar códigos mágicos e campos ambíguos
* normalizar formatos de data, moeda e documentos
* alinhar vocabulários diferentes ao modelo interno

É aqui que acontece a **anticorrupção de verdade**.

O domínio nunca “aprende” o idioma do legado.

---

## Resiliência e confiabilidade

Falhas externas são inevitáveis.
A ACL é o lugar correto para lidar com isso.

Ela concentra políticas como:

* timeout
* retry com backoff
* circuit breaker
* cache de contingência
* idempotência
* controle de taxa (rate limiting)

Essas decisões **não pertencem ao domínio**.
Elas pertencem à borda da arquitetura — à ACL.

---

## Observabilidade e auditoria

Em ambientes regulados, a ACL também é um ponto crítico para:

* logging estruturado
* métricas de uso e falha
* rastreabilidade de chamadas
* trilhas de auditoria
* correlação de transações

Isso é especialmente relevante em cenários com **LGPD**, onde é necessário saber:

* quem acessou
* quando acessou
* com qual finalidade
* quais dados foram envolvidos

---

# Quando criar uma ACL

Crie uma ACL sempre que houver integração com sistemas que:

* não seguem o mesmo modelo de domínio
* não evoluem no mesmo ritmo
* são legados ou tecnologicamente defasados
* pertencem a terceiros
* exigem governança, auditoria ou rastreabilidade
* não podem impactar a estabilidade da plataforma

Em ambientes regulados, a ACL **não é opcional** — é um mecanismo de proteção.

---

# Regras Fundamentais da ACL

| Regra                                   | Explicação                                       |
| --------------------------------------- | ------------------------------------------------ |
| **ACL é externa ao domínio**            | Nunca faz parte do core de MFUs ou CSUs.         |
| **ACL centraliza integrações externas** | Nenhum outro serviço acessa legados diretamente. |
| **ACL traduz semântica**                | Conceitos externos nunca vazam para o domínio.   |
| **ACL concentra resiliência**           | Falhas externas não afetam o core.               |
| **ACL facilita governança e auditoria** | Essencial em ambientes regulados e sensíveis.    |
| **ACL protege a evolução do domínio**   | Mudanças externas não quebram o modelo interno.  |

---

# Nota importante sobre tipos de ACL

Na literatura de Domain-Driven Design, existem **dois tipos de ACL**:

* ACL implementada como lógica interna
* ACL implementada como serviço dedicado

Neste modelo arquitetural, **adotamos exclusivamente a ACL como serviço externo**, por ser:

* mais robusta
* mais governável
* mais observável
* mais adequada a ambientes regulados
* mais segura para cenários com múltiplos legados

Essa escolha é **intencional** e alinhada ao nível de risco, escala e responsabilidade do contexto.

---

# Por que isso importa?

A ACL pode ser comparada a **uma zona alfandegária**:

* tudo que entra passa por inspeção
* tudo que sai é traduzido
* o território interno permanece protegido
* regras externas não se tornam leis internas

Sem ACL:

* o domínio se deteriora
* integrações viram dívida técnica permanente
* o sistema perde clareza e previsibilidade

Com ACL:

* o domínio permanece limpo
* a plataforma cresce com disciplina
* integrações são controladas
* a arquitetura se mantém sustentável no longo prazo

A ACL é um dos pilares que permitem que plataformas complexas — como as de crédito — evoluam **sem colapsar sob o próprio peso**.
