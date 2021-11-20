# Naivecoin 분석

## Note

### 어떻게 올바른 transaction 인지 검증할까?

어떻게 올바른 transaction 인지 검증하는 방법을 validateTransaction 함수에 주석을 달아서 설명했다.

validateTransaction 함수를 보면서 왜 코인이 암호화폐라고 불리는지 이제 알게되었다.

지갑은 프라이빗 키가 비밀키이고 지갑 주소가 공개키인 비대칭키 구조로 되어있다.

따라서 transaction을 만들 때 지갑 주소와 코인 수량으로 이뤄진 UTXO(UnspentTxOut)를 참조하는 txIn을 transaction에 포함시키고 txIn과 txOut을 해싱해서 만든
transaction.id를 지갑의 비밀키로 암호화화한 signature를 포함시키므로서 UTXO에 대한 통제권을 가진 transaction임을 증명하여 지갑 주인만 자신의 잔액에 대한 통제권을 가진 송금이
가능해진다.

```typescript
const validateTransaction = (transaction: Transaction, aUnspentTxOuts: UnspentTxOut[]): boolean => {

  // txInContent, txOutContent 해싱을 통해 transaction.id가 올바른지 체크
  if (getTransactionId(transaction) !== transaction.id) {
    console.log('invalid tx id: ' + transaction.id);
    return false;
  }
  // txIn이 transaction을 서명한 지갑의 UTXO만 참조하는 것을 확인함
  const hasValidTxIns: boolean = transaction.txIns
    .map((txIn) => validateTxIn(txIn, transaction, aUnspentTxOuts))
    .reduce((a, b) => a && b, true);

  if (!hasValidTxIns) {
    console.log('some of the txIns are invalid in tx: ' + transaction.id);
    return false;
  }
  // 트랜잭션 결과 txIn이 참조하는 UnspentTxOuts의 잔액합과 txOut의 잔액합이 동일한지 검증
  const totalTxInValues: number = transaction.txIns
    .map((txIn) => getTxInAmount(txIn, aUnspentTxOuts))
    .reduce((a, b) => (a + b), 0);

  const totalTxOutValues: number = transaction.txOuts
    .map((txOut) => txOut.amount)
    .reduce((a, b) => (a + b), 0);

  if (totalTxOutValues !== totalTxInValues) {
    console.log('totalTxOutValues !== totalTxInValues in tx: ' + transaction.id);
    return false;
  }

  return true;
};
```

## Usage

The repository for the naivecoin tutorial: https://lhartikk.github.io/

```
npm install
npm start
```

##### Get blockchain

```
curl http://localhost:3001/blocks
```

##### Mine a block

```
curl -X POST http://localhost:3001/mineBlock
``` 

##### Send transaction

```
curl -H "Content-type: application/json" --data '{"address": "04bfcab8722991ae774db48f934ca79cfb7dd991229153b9f732ba5334aafcd8e7266e47076996b55a14bf9913ee3145ce0cfc1372ada8ada74bd287450313534b", "amount" : 35}' http://localhost:3001/sendTransaction
```

##### Query transaction pool

```
curl http://localhost:3001/transactionPool
```

##### Mine transaction

```
curl -H "Content-type: application/json" --data '{"address": "04bfcab8722991ae774db48f934ca79cfb7dd991229153b9f732ba5334aafcd8e7266e47076996b55a14bf9913ee3145ce0cfc1372ada8ada74bd287450313534b", "amount" : 35}' http://localhost:3001/mineTransaction
```

##### Get balance

```
curl http://localhost:3001/balance
```

#### Query information about a specific address

```
curl http://localhost:3001/address/04f72a4541275aeb4344a8b049bfe2734b49fe25c08d56918f033507b96a61f9e3c330c4fcd46d0854a712dc878b9c280abe90c788c47497e06df78b25bf60ae64
```

##### Add peer

```
curl -H "Content-type:application/json" --data '{"peer" : "ws://localhost:6001"}' http://localhost:3001/addPeer
```

#### Query connected peers

```
curl http://localhost:3001/peers
```
