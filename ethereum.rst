ethereum

- JPMD, a USD deposit token on Base, the Ethereum Layer 2 blockchain built within Coinbase (alternative to stablecoins)
https://www.jpmorgan.com/payments/newsroom/kinexys-usd-digital-deposit-tokens

https://ethereum.stackexchange.com/questions/268/ethereum-block-architecture/6413#6413

например если смарт контракт следующий

if today > 2025-01-02
   send(50$ to adress 1234)


Transaction Receipt (Квитанция)

если условие ложно

{
  "root": "0x8dfa3b...c1b59a",
  "leaves": [
    {
      "key": "0x00",
      "value": {
        "status": "0x01", // УСПЕХ (условие не выполнилось, но транзакция не упала)
        "cumulativeGasUsed": "0x...",
        "logsBloom": "0x0000000000000000000000000000000000000000000000000000000000000000...", // Пустой блум-фильтр
        "logs": [], // ПУСТОЙ МАССИВ - нет события Transfer!
        "transactionHash": "0x...",
        "contractAddress": null,
        "gasUsed": "0x...", // Меньше газа (только проверка условия)
        "blockHash": "0x...",
        "transactionIndex": "0x00"
      }
    }
    // ... квитанции других транзакций
  ]
}

если условие истинно

{
  "root": "0x8dfa3b...c1b59a",
  "leaves": [
    {
      "key": "0x00", // Соответствует индексу транзакции
      "value": {
        "status": "0x01", // УСПЕХ (1 = success)
        "cumulativeGasUsed": "0x...",
        "logsBloom": "0x...000800...", // Блум-фильтр с событием Transfer
        "logs": [
          {
            "address": "0x...адрес_вашего_контракта...",
            "topics": [
              "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef", // Transfer signature
              "0x...адрес_контракта...", // from
              "0x0000000000000000000000000000000000001234" // to (адрес 1234)
            ],
            "data": "0x00000000000000000000000000000000000000000000000002c68af0bb140000" // 50$ в wei (~0.1 ETH)
          }
        ],
        "transactionHash": "0x...",
        "contractAddress": null,
        "gasUsed": "0x...", // Газ использован (больше чем в false случае)
        "blockHash": "0x...",
        "transactionIndex": "0x00"
      }
    }
    // ... квитанции других транзакций
  ]
}



тоесть в новом блоке сохраниться только хэш от следющих деревьев
Global State Trie
Storage Trie
Transactions Trie
Receipts Trie

// Заголовок блока (Block Header) - Фиксируется в блокчейне
{
  "parentHash": "0x...",
  "number": "15742102",
  "stateRoot": "0x89145a1c...a62d98",     // <-- Сводный хэш ВСЕГО состояния после ВСЕХ транзакций
  "transactionsRoot": "0x98cb1c...d82f37", // <-- Сводный хэш списка ВСЕХ транзакций
  "receiptsRoot": "0x8dfa3b...c1b59a",     // <-- Сводный хэш ВСЕХ квитанций (включая вашу)
  "...": "..."
}

receiptsRoot
------------


transactionsRoot
----------------
- transactionsRoot

Блок когда условие НЕ выполнилось (today = 2025-01-01):

// transactionsRoot содержит:
{
  "transactions": [
    {
      "from": "0xuser",  // с коко списывать газ за выполнение
      "to": "0xcontract", // какой контракт вызывать
      "value": "0", //  Сколько ETH приложено к транзакции
      "data": "0xcheckAndSend" // имя функции в контракте которую вызывать
    }
  ]
}

- если функция меняет свойство контракта но вызвана через .call то она не поменяет контракт и для вызова не будет записи в transactions

contract Example {
    uint256 public value; // public переменная = view функция

    // ✅ Transaction функция (изменяет состояние)
    function setValue(uint256 newValue) public {
        value = newValue; // Изменяет состояние
    }

    // ✅ Call функция (только чтение)
    function getValue() public view returns (uint256) {
        return value; // Не изменяет состояние
    }

    // ✅ Call функция (только чтение)
    function calculate() public pure returns (uint256) {
        return 2 + 2; // Даже не читает состояние
    }
}

// ❌ Call-вызов - изменения НЕ сохранятся
const result = await contract.setValue.call(100, { from: userAddress });

// ✅ Transaction-вызов - изменения сохранятся, будет добавлена запись в transactions
const tx = await contract.setValue.sendTransaction(100, {from: userAddress, gas: 50000});