# 📦 Módulo Fundamental (MFU — “Modular Fundamental Unit”)

Significa: **a menor unidade de negócio autossuficiente**, que serve como **bloco de construção** para serviços maiores.

---

# MFU — Módulo Fundamental

Um **MFU (Módulo Fundamental)** é a menor unidade de construção dentro da plataforma de serviços estruturantes.
Ele representa um **domínio de negócio bem definido**, contendo suas **regras, validações, modelos, casos de uso e persistência**, sem depender de outros serviços para funcionar.

Em outras palavras:

> Um MFU é **autocontido**, **independente**, **previsível**, e pode evoluir com segurança sem quebrar outros sistemas.

Essa característica faz com que os MFUs sejam como **tijolos estruturais**: cada um resolve um problema específico, mas se conectam de forma organizada para construir soluções maiores.

---

# Quando criar um MFU? (Cenários de uso)

## a) Cenário padrão — MFU isolado com base única

Este é o modelo ideal e recomendado.

O MFU deve possuir **uma única base ou schema de dados**, que contém todo o estado necessário para executar suas regras de negócio.

```

[mfu-dominio-x] ---> [base de dados própria]

```

Por que isso é importante:

* evita acoplamento entre domínios
* facilita mudanças internas sem quebrar o ecossistema
* permite escalabilidade independente
* simplifica governança de dados e segurança
* reduz custos operacionais e cognitivos

---

## b) MFUs compartilhando a mesma base de dados (Não permitido)

MFUs **não podem** acessar o schema ou banco de dados de outros MFUs.
Cada módulo deve ter **seu próprio espaço de dados**.

```

[mfu-a] -----+
             |--X-->[schema]  (proibido!)
[mfu-b] -----+

```

Dois problemas graves surgem quando esse princípio é quebrado:

1. **Acoplamento invisível**
   Mudanças no schema de um MFU passam a impactar outro.

2. **Perda do isolamento de domínio**
   Cada MFU perde sua autonomia, sua capacidade de evoluir e até seus limites de responsabilidade.

Também não é permitido que um MFU acesse **mais de uma base**:

```
       +--------> [base 1]
[mfu-x]
      +---X-----> [base 2]   (proibido!)

```

Regra de ouro:

> **Um MFU = Uma base**

---

## c) Integração direta entre MFUs (Não permitido)

MFUs **nunca devem se chamar diretamente** via HTTP, REST, webservice ou RPC.

Exemplo proibido:

```

[mfu-a] ----X----> [mfu-b]

```

Por quê?

* cria acoplamento por dependências
* gera cadeias de chamadas difíceis de observar
* aumenta latência e fragilidade da solução
* dificulta testes, versionamento e rollback
* cria integrações ponto a ponto difíceis de manter

A comunicação correta entre MFUs deve ser **assíncrona**, através de **eventos** ou filas, preservando o desacoplamento:

```

[mfu-a] ----> (evento) ----> [mfu-b]

```

Ou seja:

> MFUs reagem a eventos, não a chamadas diretas.

Cada módulo faz o que precisa fazer quando recebe um evento, mantendo a autonomia de cada domicílio.

---

# Resumo das Regras Fundamentais do MFU (Módulo Fundamental)

| Regra                                 | Explicação                                                       |
| ------------------------------------- | ---------------------------------------------------------------- |
| **MFU é autocontido**                 | Não deve depender de outros serviços para aplicar suas regras.   |
| **Possui uma única base de dados**    | Um MFU = um schema; nunca acessar base de outro módulo.          |
| **Não chama outros MFUs diretamente** | Nada de integração ponto a ponto; comunicação apenas assíncrona. |
| **Estrutura pequena e coesa**         | Representa o menor bloco de construção da plataforma.            |

---

# Por que isso importa? 

Imagine que MFUs são **peças de LEGO**:

* Cada peça é completa por si só
* Uma não mexe dentro da outra
* Você conecta apenas pelos pontos certos
* Você pode trocar uma peça sem desmontar tudo

Quando você cria sistemas assim:

* fica fácil testar
* fica fácil corrigir
* fica fácil escalar
* fica fácil trocar componentes no futuro
* fica difícil quebrar o sistema sem querer

MFUs permitem que a plataforma cresça **com ordem**, não com "puxadinhos".
