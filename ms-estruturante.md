# O que é um Microserviço Estruturante

Um Microserviço Estruturante é um microserviço de baixa variabilidade, altamente genérico dentro do domínio de crédito, capaz de ser reutilizado por qualquer jornada, produto ou canal que precise daquela funcionalidade.

Ele não pertence a um produto específico (ex.: consignado, cartão, empréstimo pessoal).  
Ele pertence ao domínio raiz de crédito — e por isso é compartilhável, estável e universal.

## O microserviço estruturante:

- encapsula regras que não mudam de acordo com o produto
- possui valor independente, não apenas como parte da jornada final
- oferece APIs genéricas que servem como lego blocks para que outras equipes montem suas jornadas
- não conhece o produto que o consome
- não aplica regras de negócio específicas de nenhum produto
- resolve parte do fluxo de crédito que é comum em TODAS as jornadas

## Exemplos reais:

| Componente estruturante | O que faz | Por que é estruturante |
|-------------------------|-----------|--------------------------|
| Simulação | Calcula parcelas, juros, CET, prazos | Sempre será usado: consignado, PF, PJ, cartão, refinanciamento |
| Formalização | Gera documentos, verifica assinaturas, coleta aceite | É genérico a qualquer contratação |
| Elegibilidade | Verifica restrições, score, limites | Todo produto precisa disso |
| Análise | Modelos estatísticos e regras de risco | Centraliza política de crédito |
| Liberação de Valores | Integra com tesouraria, pagamento, TED/PIX | Parte final obrigatória |
| Cadastro de cliente | Dados básicos, validações, documentos | Todos usam |

Se você removeu o nome do produto (ex.: “consignado”), e o microserviço continua fazendo sentido, provavelmente ele é estruturante.

>Em resumo:  
>Um microserviço estruturante é um bloco de domínio genérico que compõe qualquer jornada de crédito, garantindo consistência, padronização e reutilização.
