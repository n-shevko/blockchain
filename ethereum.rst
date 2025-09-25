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
- Квитанции не хранятся изначально в дереве - дерево строится на основе уже собранного списка транзакций для данного блока

transactionsRoot
----------------
- transactionsRoot

- Транзакции не хранятся изначально в дереве - дерево строится на основе уже собранного списка транзакций для данного блока

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


-------------------

Ethereum Node
Blockchain Layer
├── Block Headers ✅ (состояние - исторические данные)
├── Block Bodies ✅ (состояние - исторические транзакции)
└── Transaction Pool ✅ (состояние - текущие неподтвержденные транзакции)

State Layer
├── World State ✅ (состояние - текущие аккаунты и балансы)
├── Account Storage ✅ (состояние - данные контрактов)
└── Contract Code ✅ (состояние - cкомпилированный байткод байткод контрактов)

Execution Layer
├── EVM ❌ (алгоритм - интерпретатор байткода)
├── Gas Accounting ❌ (алгоритм - расчет комиссий)
└── Transaction Processing ❌ (алгоритм - обработка транзакций)

Network Layer
├── P2P Protocol ❌ (алгоритм - обмен данными)
├── Discovery Protocol ❌ (алгоритм - поиск узлов)
└── Sync Protocols ❌ (алгоритм - синхронизация)

Consensus Layer
├── Beacon Chain ❌ (алгоритм - управление PoS)
├── Validator Management ❌ (алгоритм - управление валидаторами)
└── Finality Gadget ❌ (алгоритм - финализация блоков)

World State (включает в себя Account Storage)
-----------

world_state = {
    # Внешние аккаунты (EOA - управляются приватными ключами)
    "0x5aAeb6053F3E94C9b9A09f336": {
        "nonce": 15,                    # Счетчик транзакций этого аккаунта
        "balance": 2500000000000000000, # Баланс: 2.5 ETH (в wei)
        "storageRoot": "0x",           # Пусто для EOA
        "codeHash": "0xc5d2460186f7"   # Хэш пустого кода для EOA
    },

    "0x52908400098527886E0F703006": {
        "nonce": 8,
        "balance": 100000000000000000,  # 0.1 ETH
        "storageRoot": "0x",
        "codeHash": "0xc5d2460186f7"
    },

    # Контрактные аккаунты
    "0x89d24A6b4CcB1B6fAA2625f": {     # Контракт DAI Stablecoin
        "nonce": 1,                    # У контрактов обычно nonce=1
        "balance": 50000000000000000,  # 0.05 ETH на балансе контракта
        "storageRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",  #  mercle root tree от Account Storage данных контракта
        "codeHash": "0x9b7c7b8b8f...", # Хэш байткода контракта DAI
    },

    "0xC02aaA39b223FE8D0A0e5C4F": {    # Контракт WETH
        "nonce": 1,
        "balance": 1000000000000000000, # 1 ETH заблокирован в контракте
        "storageRoot": "0x...",
        "codeHash": "0x...",
    }
}

Account Storage
---------------

- Например для контракта
// SimpleStorage.sol
contract SimpleStorage {
    uint256 public number;        // slot 0
    address public owner;         // slot 1
    bool public isActive;         // slot 2
    uint256 public count;         // slot 3
}

storage_of_contract_0x123... = {
    # Простые переменные
    "slot_0x0": "0x0000000000000000000000000000000000000000000000000000000000000000",  # number = 0
    "slot_0x1": "0x0000000000000000000000000000000000000000000000000000000000000000",  # owner = 0x0
    "slot_0x2": "0x0000000000000000000000000000000000000000000000000000000000000000",  # isActive = false
    "slot_0x3": "0x0000000000000000000000000000000000000000000000000000000000000000",  # count = 0
}

-
contract Token {
    mapping(address => uint256) public balances;       // slot 0 (но не хранится напрямую!)
    mapping(address => mapping(address => uint256)) public allowances; // slot 1
    string public name;                                // slot 2
    uint256 public totalSupply;                        // slot 3
}

# balances[0xuser1] = 1000
# balances[0xuser2] = 500

# Ключ для mapping вычисляется: keccak256(key + slot)
slot_balances = 0

# Для balances[0xuser1]:
key1 = keccak256(abi.encode(0xuser1, 0))  # → "0x1234abcd..."
# Для balances[0xuser2]:
key2 = keccak256(abi.encode(0xuser2, 0))  # → "0x5678efgh..."

storage_of_token_contract = {
    # Простые переменные
    "slot_0x2": "0x...",  # name (особое кодирование для string)
    "slot_0x3": "0x0000000000000000000000000000000000000000000000000000000000000000",  # totalSupply

    # Mapping balances
    "slot_0x1234abcd...": "0x00000000000000000000000000000000000000000000000000000000000003e8",  # 1000
    "slot_0x5678efgh...": "0x00000000000000000000000000000000000000000000000000000000000001f4",  # 500

    # Mapping allowances (вложенное)что инициирует какие контракты нужно запускать в определенный момент времени
    # allowances[0xowner][0xspender] = 200
    "slot_0x9abc123...": "0x00000000000000000000000000000000000000000000000000000000000000c8",  # 200
}

- Хранятся в виде

# Отдельная база данных для хранилищ контрактов
account_storage_database = {
    # Ключ: адрес контракта + слот
    # Значение: данные хранилища

    "0x89d24A6b4CcB1B6fAA2625f:0x123...abc": 1000000000000000000000,  # DAI баланс
    "0x89d24A6b4CcB1B6fAA2625f:0x456...def": 500000000000000000000,   # DAI баланс
    "0x89d24A6b4CcB1B6fAA2625f:0x789...ghi": 1,                       # totalSupply

    "0xC02aaA39b223FE8D0A0e5C4F:0x...": 500000000000000000,           # WETH депозит
}

в этом хэше храняться данные всех контрактов.
но когда нужно рассчитать storageRoot определенного контракта то просто выбираются значения для нужного контракта.

Следующий вопрос
----------------
что инициирует какие контракты нужно запускать в определенный момент времени