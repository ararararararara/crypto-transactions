# :key: crypto-transactions

# 영지식 증명
암호학에서 누군가가 상대방에게 어떤 사항이 참임을 증명할 때, 그 문장이 참 거짓 여부를 제외한 어떤 것도 노출되지 않는 상호작용 절차를 말한다.
정보를 발설하지 않고 그 정보를 알고 있다는 것을 증명하는 것
코인의 경우 코인의 고유정보나 거래 노출없이 코인을 보유, 거래하는 것을 의미

## 1. 완전성
 어떤 문장이 참이면, 정직한 증명자는 검증자에게 이사실을 납득시킬 수 있어야한다.
## 2. 건실성
 어떤 문장이 거짓이며느 어떠한 부정직한 증명자라도 정직한 검증자에게 이 문장이 사실이라고 납득시킬 수 없어야 한다.
## 3. 영지식성
 어떤문장이 참이면 검증자는 문장의 참 거짓 이외에는 아무것도 알 수 없어야 한다.
 
## :girl:비유를 통한 예시
![300px-Zkip_alibaba2](https://user-images.githubusercontent.com/89236248/150671970-dfe6e42d-d827-46c9-8ed2-a402a4a39b19.png)


증명자와 검증자가 있을때 증명자는 동굴안에 있는 비밀 문의 열쇠를 갖고 있다.
동굴은 A와 B통로로 되어있고 입구 반대편에 비밀문이 있다.
증명자는 검증자에게 열쇠를 갖고 있다는 것을 알려야 하지만 다른 사람ㅁ에게 자신에 관한 비밀이 알려지는 것을 원치 않는다.
이때 검증자는 동굴밖에서 기다리고 증명자가 A와 B중 아무곳으로 들어간다. 검증자는 증명자가 어디로 들어갔는지 모른다.
검증자가 A나 B아무거나 골라 증명자에게 나오라고 했을때 증명자는 검증자가 부른 통로로 나온다.
열쇠가 있다면 증명자는 어떤 통로를 골라도 그 통로로 나올수 있고 열쇠가 없다면 50%확률로 나올 수 있게 된다.
열쇠가 없다면 이 실험을 20번만 반복해도 100만분의 1확률이 된다.

이 실험은 검증자 외에 어떤 정보도 주지 않고 캠코더로 녹화해도 다른 사람들에겐 증명이 안되지만 검증자에게만 유효한 증명이 된다.

현재 영지식 증명을 사용하는 코인들은 Zcash, Zcoin, Pvix, Kurrent, Spectre등이 있다.

# 공개 키 암호화 및 서명
공개 키 암호화 에는 비밀 키와 공개 키라는 키 쌍이 있습니다. 
![Digital_signatures](https://user-images.githubusercontent.com/89236248/150672018-93cf8496-fd81-4206-bb7c-ca153e4a95d2.png)


공개 키는 비밀 키에서 파생될 수 있지만 비밀 키는 공개 키에서 파생될 수 없습니다. 
공개 키(이름에서 알 수 있듯이)는 누구에게나 안전하게 공유될 수 있습니다.

## 개인키 및 공개키
유효한 개인키는 임의의 32 바이트 문자열

유효한 공개키는 64바이트 문자열

# 거래구조

트랜잭션은 입력과 출력의 두가지 요소로 구성

출력은 코인이 전송되는 위치를 지정하고 
입력은 실제로 전송된 코인이 첨음에 존재하고 보낸사람이 소유하고 있다는 증거를 제공

# 트랜잭션 출력
![transactions](https://user-images.githubusercontent.com/89236248/150672027-437c11e5-d3ee-4e4d-9408-f959a6460c38.png)

```
class TxOut {
    constructor (address, amount){
        this.address = address;///주소 : ECDSA 공개키
        this.amount = amount;// 코인수량
    }
}
```

# 트랜잭션 입력

```
class TxIn {
    constructor(txOutId, yxOutIndex, signature){
        this.txOutId = txOutId;
        this.TxOutIndex = TxOutIndex;
        this.signature = signature;
    }
}
```
![transactions2](https://user-images.githubusercontent.com/89236248/150672036-66a1094e-ef1b-45c5-8475-ef6baa886568.png)


# 트랜잭션 구조
```
class Transaction {
    constructor (id, txIns, txOuts){
        this.id = id;
        this.txIns = txIns;
        txIns.txOuts = txOuts;
    }
}
```

# 거래 아이디
트랜잭션 ID는 트랜잭션 내용에서 해시를 가져와서 계산됩니다. 그러나 txIds의 서명은 나중에 트랜잭션에 추가되므로 트랜잭션 해시에 포함 되지 않습니다

```
const getTranxaction = (transaction) => {
    const txInContent = transaction.txIns
    .map((txIn) => txIn.txOutId + txIn.TxOutIndex )
    .reduce((a,b) => a + b, '')

    const txOutContent = transaction.txOuts
    .map((txOut) =>  txOut.address + txOut.amount)
    .reduce((a,b) => a + b, '')

    return CryptoJS.SHA256(txInContent + txOutContent).toString();

}
```

# 거래 서명
```
const signTxIn = (transaction, txInIndex, privateKey, aUnspentTxOuts) => {
    const txIn = transaction.txIns[txInIndex];
    const dataToSign = transaction.id;
    const referencedUnspentTxOut = findUnspentTxOut(txIn.txOutId, txIn.txOutIndex, aUnspentTxOuts);
    const referencedAddress = referencedUnspentTxOut.address;
    const key = ec.keyFromPrivate(privateKey, 'hex');
    const signature = toHexString(key.sign(dataTosing).toDER());
    return signature;

}
```
거래가 서명된 후에는 거래 내용을 변경할 수 없다는 것이 중요합니다. 거래가 공개되기 때문에 블록체인에 포함되기 전에도 누구나 거래에 액세스할 수 있습니다.

트랜잭션 입력에 서명할 때 txId만 서명됩니다. 트랜잭션의 내용이 수정되면 txId가 변경되어 트랜잭션과 서명이 무효화됩니다.

# 미사용 트랜잭션 출력

 미사용 트랜잭션 출력의 데이터 구조
 
 ```
 class UnspentTxOut {
    txOutId;;
    txOutIndex ;
    address;
    amount;

    constructor(txOutId, txOutIndex, address, amount) {
        this.txOutId = txOutId;
        this.txOutIndex = txOutIndex;
        this.address = address;
        this.amount = amount;
    }
}
 ```
 # 미사용 트랜잭션 출력 업데이트
 
  먼저 미사용 트랜잭션 출력 검색
  
 ```
  const newUnspentTxOuts = aTransactions
        .map(t => {
            return t.txOuts.map((txOut, index) => (
                new UnspentTxOut(t.id, index, txOut.address, txOut.amount)
            ))
        })
        .reduce((a, b) => a.concat(b), []);
 ```
 
 새 트랜잭션 검색
 
 ```
  const consumedTxOuts = aTransactions
        .map((t) => t.txIns)
        .reduce((a, b) => a.concat(b), [])
        .map((txIn) => new UnspentTxOut(txIn.txOutId, txIn.txOutIndex, '', 0));
 ```
 
 기존 트랜잭션 출력 제거하고 추가
 
 ```
   const resultingUnspentTxOuts = aUnspentTxOuts
        .filter(((uTxO) => !findUnspentTxOut(uTxO.txOutId, uTxO.txOutIndex, consumedTxOuts)))
        .concat(newUnspentTxOuts);

 ```
 
  # 트랜잭션 유효성 검사
  
  올바른 트랜잭션 구조
 ```
 const validateTransaction = (transaction, aUnspentTxOuts) => {
    if (!isValidTransactionStructure(transaction)) {
        return false;
    }
 ```
 
 트랜잭션 ID 검증
```
 if (getTransactionId(transaction) !== transaction.id) {
        console.log('invalid tx id: ' + transaction.id);
        return false;
    }
```

트랜잭션 txIns 검증
```
 const hasValidTxIns = transaction.txIns
        .map((txIn) => validateTxIn(txIn, transaction, aUnspentTxOuts))
        .reduce((a, b) => a && b, true);
    if (!hasValidTxIns) {
        console.log('referenced txOut not found: ' + transaction.id);
        return false;
    }
```

트랜잭션 txOuts의 검증 - txIns의 합의 값과 txOuts의 합이 같은지?
 ```
   const totalTxInValues = transaction.txIns
        .map((txIn) => getTxInAmount(txIn, aUnspentTxOuts))
        .reduce((a, b) => (a + b), 0);
    const totalTxOutValues = transaction.txOuts
        .map((txOut) => txOut.amount)
        .reduce((a, b) => (a + b), 0);
    if (totalTxOutValues !== totalTxInValues) {
        console.log('totalTxOutValues !== totalTxInValues in tx: ' + transaction.id);
        return false;
    }
 ```
 # 코인베이스 거래
 
 ```
 const COINBASE_AMOUNT = 50; // 입력은 없고 출력만..블럭을 찾으면 50개동전을 보상
 ```
 
 코인베이스 유효성 검사
 ```
 const validateCoinbaseTx = (transaction, blockIndex) => {
    console.log("transaction============ \n", transaction)
    console.log("transaction.txouts============ \n", transaction.txOuts)
    if (transaction == null) {
        console.log('the first transaction in the block must be coinbase transaction');//첫 transaction이 코인베이스 거래인지
        return false;
    }
    if (getTransactionId(transaction) !== transaction.id) {
        console.log('invalid coinbase tx id: ' + transaction.id); // 유효한 transaction id 인지
        return false;
    }
    if (transaction.txIns.length !== 1) {
        console.log('one txIn must be specified in the coinbase transaction'); //txIn 하나는 거래하나씩 지정
        return;
    }
    if (transaction.txIns[0].txOutIndex !== blockIndex) {
        console.log('the txIn index in coinbase tx must be the block height'); //처음 txIn인덱스는 블록높이와 같아야 함
        return false;
    }
    if (transaction.txOuts.length !== 1) {
        console.log('invalid number of txOuts in coinbase transaction'); // txOuts 하나는 거래하나씩 지정
        return false;
    }
    if (transaction.txOuts[0].amount !== COINBASE_AMOUNT) {
        console.log('invalid coinbase amount in coinbase transaction'); // txOuts양이 코인 전체의 값과 같은지
        return false;
    }
    return true;
};
 ```
 
