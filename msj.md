# 🎯 Módulo de Serviço de Jornada (MSJ — “Module for Service Journey”)

Significa: **um módulo especializado que aplica regras específicas de uma jornada de negócio**, combinando MFUs e CSUs conforme necessário para entregar a experiência final de um produto, canal ou modalidade de crédito.

---

# MSJ — Módulo de Serviço de Jornada

O **MSJ (Módulo de Serviço de Jornada)** é a **terceira camada** da arquitetura de crédito.  
Enquanto o MFU representa o **domínio puro**, e o CSU representa o **processo coordenado**, o MSJ representa o **produto**, a **experiência final**, a **jornada específica** que um cliente ou operação precisa executar.

Se o MFU é a “peça fundamental”  
e o CSU é o “encaixe entre as peças”,  
então o MSJ é a **forma final**, o **artefato completo**, o **produto configurado** para atender um caso de uso real de negócio.

Em outras palavras:

> Um MSJ é responsável por **adaptar, especializar e aplicar regras próprias** de uma modalidade — por exemplo: consignado, CDC, cartão de crédito, empréstimo simplificado, renovação, portabilidade — usando MFUs e CSUs como base.

Ele transforma a plataforma estruturante **em produtos reais**, prontos para uso pelas jornadas.

---

# Por que o MSJ existe?

A plataforma de crédito estruturante fornece blocos reutilizáveis (MFUs) e processos coordenados (CSUs).  
Mas cada modalidade de crédito possui regras específicas que **não podem** estar nesses módulos, pois:

* ferem o isolamento de domínio dos MFUs
* acoplam CSUs a regras que não pertencem ao processo genérico
* criam um labirinto de exceções difíceis de manter

Exemplo: a jornada de crédito consignado

1. utiliza a simulação genérica do MFU de Simulação  
2. utiliza o fluxo de formalização do CSU de Formalização  
3. mas aplica regras **exclusivas**, como:  
   * margem consignável  
   * convênio  
   * validade de extrato  
   * bloqueios do órgão pagador  
   * faixas etárias específicas  

Essas regras pertencem à **jornada**, não à plataforma.

Por isso o MSJ existe:  
para adicionar a camada necessária de regras específicas **sem poluir a plataforma**.

---

# Como o MSJ funciona?

O MSJ atua como **uma casca de especialização** ao redor da plataforma:

* chama MFUs quando precisa das regras fundamentais
* chama CSUs quando precisa de processos coordenados
* aplica regras próprias da jornada
* adapta a resposta aos requisitos do produto
* executa validações específicas que **não fazem sentido** para a plataforma estruturante

Um MSJ costuma conter:

* regras específicas da jornada  
* validações próprias  
* adaptações de request/response  
* transformações de dados  
* chamadas para MFUs/CSUs  
* tratativas de exceções do produto  

Ele é o responsável por **encaixar a plataforma estruturante no mundo real**.

---

# Como diferenciar MFU × CSU × MSJ?

| Camada  | O que representa    | Tipo de regra    | Exemplos                                                  |
| ------- | ------------------- | ---------------- | --------------------------------------------------------- |
| **MFU** | Domínio puro        | Regra universal  | cálculo de parcelas, validação de contrato, liberação     |
| **CSU** | Processo entre MFUs | Orquestração     | fluxo de contratação, sequenciamento, compensações        |
| **MSJ** | Produto ou jornada  | Regra específica | simulação-consignado, formalização-cdc, liberação-pessoal |

Ou com uma metáfora:

* **MFU** = tijolo  
* **CSU** = cimento  
* **MSJ** = a parede montada com formato, pintura e função específica  

---

# Quando criar um MSJ?

Crie um MSJ quando:

1. A jornada de crédito precisa de **regras próprias** que não fazem parte do domínio genérico.  
2. É necessário **estender um fluxo genérico** (CSU) com etapas adicionais.  
3. A jornada exige **validações exclusivas**, como:
   * margem consignável  
   * taxa diferenciada  
   * exigências regulatórias  
   * requisitos específicos por canal  
4. As adaptações prejudicariam a clareza dos módulos estruturantes.  
5. O produto final precisa expor **APIs próprias**.  

---

# Estrutura recomendada para um MSJ

O MSJ deve ser:

* leve  
* especializado  
* focado em **regras da jornada**, não do domínio  
* dependente apenas de MFUs e CSUs  
* sem acesso direto a bases dos MFUs  
* sem orquestrações profundas (que pertencem ao CSU)  

Modelos de comunicação:

```
[msj-consignado] ---> [csu-formalizacao] ---> [mfu-formalizacao]
|
---> [mfu-simulacao]
|
---> [csu-liberacao]
```

Ou usando apenas MFUs:

```
[msj-cartao] ---> [mfu-analise-limite]
[msj-cartao] ---> [mfu-emissao]
```

---

# Cenários de Uso (diagramas em PlantUML)

## a) Jornada que utiliza MFUs diretamente

@startuml
actor Cliente
Cliente --> MSJ : solicitar simulacao
MSJ --> MFU_Simulacao : calcular parcelas
MFU_Simulacao --> MSJ : resultado
MSJ --> Cliente : resposta adaptada
@enduml

yaml
Copy code

---

## b) Jornada que utiliza CSUs para processos completos

```
@startuml
MSJ --> CSU_Contratacao : iniciar jornada
CSU_Contratacao --> MFU_Simulacao : simular
CSU_Contratacao --> MFU_Formalizacao : formalizar
CSU_Contratacao --> MFU_Liberacao : liberar valor
CSU_Contratacao --> MSJ : status consolidado
@enduml
```

---

## c) Jornada com regras exclusivas

```
@startuml
MSJ_Jornada_X --> MFU_Simulacao : simular valor base
MSJ_Jornada_X --> MSJ_Jornada_X : aplicar margem adicional
MSJ_Jornada_X --> MFU_Validacao : validar regras específicas
MSJ_Jornada_X --> CSU_Contratacao : iniciar fluxo
@enduml
```
---

# Regras Fundamentais do MSJ

| Regra                                                | Explicação                                           |
| ---------------------------------------------------- | ---------------------------------------------------- |
| **MSJ não duplica regras de MFUs**                   | A lógica do domínio pertence ao MFU.                 |
| **MSJ não orquestra fluxos profundos**               | Processos multi-etapas pertencem ao CSU.             |
| **MSJ aplica regras próprias da jornada**            | Tudo que é específico do produto vive aqui.          |
| **MSJ integra a plataforma ao produto final**        | Ele adapta MFUs e CSUs às necessidades reais.        |
| **MSJ pode chamar MFUs e CSUs, mas não o contrário** | Evita acoplamento invertido e mantém hierarquia.     |

---

# Por que o MSJ é importante?

A plataforma estruturante sozinha não forma produtos.  
O MSJ é o que transforma os blocos centrais em:

* Simulação Consignado  
* Formalização CDC  
* Liberação Cartão  
* Refinanciamento  
* Portabilidade  
* Oferta Pré-Aprovada  

Ele permite que:

* diversos produtos reutilizem a mesma base  
* jornadas evoluam sem quebrar outras  
* times criem produtos diferentes com consistência  
* a plataforma mantenha ordem e governança  

Sem o MSJ:

1. Regras específicas se infiltrariam dentro dos MFUs e CSUs.  
2. Cada jornada viraria um microserviço isolado, duplicando tudo.  

O MSJ cria a camada saudável de especialização do negócio.

---

# Resumo Geral (MFU × CSU × MSJ)

MFU = Regras fundamentais de domínio
CSU = Processos coordenados entre domínios
MSJ = Regras específicas de produtos/jornadas

Ou, metaforicamente:

MFU = tijolo
CSU = cimento
MSJ = parede pronta
