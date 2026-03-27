# Test Cases – Fluxo de Pagamento (Terminal de Pagamento)

Este documento descreve casos de teste funcionais para simular o fluxo de pagamento de um terminal de pagamento (TPA), usando APIs testadas com Postman. O foco é validar integração básica, tratamento de erros e consistência de dados, alinhado com boas práticas de Quality Assurance em sistemas de pagamentos.

---

## Caso 1 – Pagamento bem‑sucedido (Happy Path)

**Objetivo:**  
Validar o fluxo de pagamento bem‑sucedido com dados de cartão válidos.

**Pré‑condições:**  
- Terminal online e com sessão ativa.  
- Cliente disponibiliza dados de cartão válidos (número, data de validade, CVV).  
- Montante de pagamento definido (ex.: 100.00 €).

**Passos:**  
1. Terminal envia requisição `POST /payments` com:  
   - `amount`, `currency`, `cardNumber`, `expiryDate`, `cvv`, `terminalId`, `userId`.  
2. Backend processa o pagamento e devolve resposta com `status: "approved"` e `transactionId`.  
3. Terminal exibe mensagem de “pagamento autorizado”.

**Resultados esperados:**  
- Código HTTP: `200`.  
- Resposta contém:  
  - `status: "approved"`,  
  - `transactionId` único,  
  - `amount` e `currency` corretos.  
- Montante apresentado é o mesmo enviado no request.  
- Nenhum valor é cobrado sem confirmação explícita de `status: approved`.

**Tipo de teste:** Funcional / API (Postman).




## Caso 2 – Pagamento recusado

**Objetivo:**  
Validar o comportamento do sistema quando o pagamento é recusado (cartão inválido, saldo insuficiente, etc.).

**Pré‑condições:**  
- Terminal online.  
- Cliente insere dados de pagamento inválidos ou suspeitos (ex.: montante muito alto, CVV inválido, data de expiração passada).

**Passos:**  
1. Terminal envia `POST /payments` com dados inválidos ou inconsistentes.  
2. Backend reconhece erro (saldo insuficiente, cartão bloqueado, etc.).  
3. Backend devolve `status: "declined"` com `errorCode` e `message`.

**Resultados esperados:**  
- Código HTTP: `200` (ou `400`/`402`, conforme o design da API, para este caso mantemos 200 como mock).  
- Resposta contém:  
  - `status: "declined"`,  
  - `errorCode` definido,  
  - `message` descritiva (ex.: “Fundo insuficiente”).  
- Terminal exibe mensagem clara de “pagamento recusado” e não realiza o débito.  
- Nenhum montante é debitado do cliente neste cenário.

**Tipo de teste:** Funcional / API (Postman) – cenário de erro.





## Caso 3 – Cancelamento de pagamento autorizado

**Objetivo:**  
Validar a operação de cancelamento de um pagamento já autorizado, antes da captura final.

**Pré‑condições:**  
- Existe um pagamento já com `status: "approved"` e `transactionId` registado.  
- Terceiro que o pagamento ainda não foi capturado/finalizado.

**Passos:**  
1. Operador/cliente solicita cancelamento no terminal.  
2. Terminal envia `POST /payments/cancel` com:  
   - `transactionId`,  
   - `reason` do cancelamento,  
   - `terminalId` e `userId`.  
3. Backend valida o `transactionId` e o estado (“approved”).  
4. Backend atualiza a transação para `status: "canceled"` ou equivalente.

**Resultados esperados:**  
- Código HTTP: `200`.  
- Resposta contém:  
  - o mesmo `transactionId`,  
  - `status` atualizado para cancelado.  
- O montante não é efetivamente debitado nem transferido após o cancelamento.  
- Histórico da transação no sistema reflete o estado cancelado com motivo registado.

**Tipo de teste:** Funcional / API (Postman).