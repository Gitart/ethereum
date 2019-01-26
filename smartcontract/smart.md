## Here's a simple example of how you can interact with a smart contract in Go. Assuming you have solc and abigen installed.

```sol
solc --abi Store.sol > Store.abi
solc --bin Store.sol > Store.bin
abigen --bin=Store.bin --abi=Store.abi --pkg=store --out=Store.go
```

## Store.sol

```sol
pragma solidity ^0.4.24;

contract Store {
  event ItemSet(bytes32 key, bytes32 value);

  string public version;
  mapping (bytes32 => bytes32) public items;

  constructor(string _version) public {
    version = _version;
  }

  function setItem(bytes32 key, bytes32 value) external {
    items[key] = value;
    emit ItemSet(key, value);
  }
}
```

### main.go

```golang
package main

import (
    "context"
    "crypto/ecdsa"
    "fmt"
    "log"
    "math/big"

    "github.com/ethereum/go-ethereum/accounts/abi/bind"
    "github.com/ethereum/go-ethereum/common"
    "github.com/ethereum/go-ethereum/crypto"
    "github.com/ethereum/go-ethereum/ethclient"

    store "./contracts" // for demo
)

func main() {
    client, err := ethclient.Dial("https://rinkeby.infura.io")
    if err != nil {
        log.Fatal(err)
    }

    privateKey, err := crypto.HexToECDSA("fad9c8855b740a0b7ed4c221dbad0f33a83a49cad6b3fe8d5817ac83d38b6a19")
    if err != nil {
        log.Fatal(err)
    }

    publicKey := privateKey.Public()
    publicKeyECDSA, ok := publicKey.(*ecdsa.PublicKey)
    if !ok {
        log.Fatal("error casting public key to ECDSA")
    }

    fromAddress := crypto.PubkeyToAddress(*publicKeyECDSA)
    nonce, err := client.PendingNonceAt(context.Background(), fromAddress)
    if err != nil {
        log.Fatal(err)
    }

    gasPrice, err := client.SuggestGasPrice(context.Background())
    if err != nil {
        log.Fatal(err)
    }

    auth := bind.NewKeyedTransactor(privateKey)
    auth.Nonce = big.NewInt(int64(nonce))
    auth.Value = big.NewInt(0)     // in wei
    auth.GasLimit = uint64(300000) // in units
    auth.GasPrice = gasPrice

    address := common.HexToAddress("0x147B8eb97fD247D06C4006D269c90C1908Fb5D54")
    instance, err := store.NewStore(address, client)
    if err != nil {
        log.Fatal(err)
    }
    ```
    [Link for sample](https://github.com/miguelmota/ethereum-development-with-go-book)
