# ETHERNAUT SORU 14: Gatekeeper Two

> This gatekeeper introduces a few new challenges. Register as an entrant to pass this level.
>
>##### Things that might help:
>
>-   Remember what you've learned from getting past the first gatekeeper - the first gate is the same.
>-   The `assembly` keyword in the second gate allows a contract to access functionality that is not native to vanilla Solidity. See [here](http://solidity.readthedocs.io/en/v0.4.23/assembly.html) for more information. The `extcodesize` call in this gate will get the size of a contract's code at a given address - you can learn more about how and when this is set in section 7 of the [yellow paper](https://ethereum.github.io/yellowpaper/paper.pdf).
>-   The `^` character in the third gate is a bitwise operation (XOR), and is used here to apply another common bitwise operation (see [here](http://solidity.readthedocs.io/en/v0.4.23/miscellaneous.html#cheatsheet)). The Coin Flip level is also a good place to start when approaching this challenge.



```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperTwo {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        uint256 x;
        assembly {
            x := extcodesize(caller())
        }
        require(x == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
}
```


Bu soruda bir `entrant` değişkeni var. Bu değişkene `enter` fonksiyonunu başarılı bir şekilde çalıştıran kullanıcının adresi atanıyor. Soruda bizden istenen şey ise bu `entrant` değişkenine bir değer atamak. Bunu yapmak için ise öncelikle `enter` fonksiyonunu ve onun `modifier`larını başarılı bir şekilde çalıştıracak bir çağrı yapmamız lazım.

```solidity
// SPDX-License-Identifier: MIT
pragma  solidity  ^0.8.0;
import  "chall.sol";

contract Solution{  

constructor(address hedef){

	GatekeeperTwo gatekeeper = GatekeeperTwo(hedef);
	gatekeeper.enter(bytes8(keccak256(abi.encodePacked(address(this))))  ^  bytes8(type(uint64).max));
	
	}
}
```
Sorunun çözümü için öncelikle `modifier`lara tek tek bakalım.

1. `gateOne()`
	- Bu fonksiyon `msg.sender` ve `tx.origin` değerlerinin farklı olmasını istiyor. İlk değer fonksiyon çağrısını yapan adrestir ve ikinci değer de bu çağrının oluşması için ilk çağrıyı yapan adrestir. Bunu bir kontrakt aracılığı ile kolaylıkla çözebiliriz. Öncelikle istediğimiz eylemleri yapacak bir kontrakt yazarız ve kendi `EOA`mız ile bu kontrakt üzerinden hedef ile iletişime geçeriz. Böylelikle `tx.origin` bizim adresimiz iken `msg.sender` kontraktın adresi olmuş olur.

2. `gateTwo()`
	- Sorunun belkide en zor kısmı burası. `extcodesize(caller())` op kodu fonksiyonu çağıran adresin hafızasının büyüklüğünü verir. Yani başka bir deyişle bu hafıza 0 dan büyükse gönderenin kontrakt olduğunu ifşa eder. Yalnız burda küçük bir açık bulunmaktadır. Kontraktlar `constructor` fonksiyonlarını çalıştırırken hiç kod depolamamış olarak görünürler. `constructor` içerisinden yapılacak bir çağrı ile bu güvenlik katmanı geçilebilir.

3. `gateThree()`
	- Bu fonksiyon bizden bir anahtar talep etmektedir. Bu anahtar ile adresimiz `XOR` operasyonunu geçirdikten sonra 8 bytelik en büyük değeri bize verecek bir anahtar olmalıdır. Neyse ki `XOR` tersine de hesaplanabilen bir işlem. Soruda kendi adresimizi içeren ifade ile sonuçta karşılaştırılan ifadeyi `XOR` operasyonundan geçirirsek bize gereken anahtarı verecektir.

Son olarak bunların hepsini birleştirerek bu kodu yazabiliriz ve soruyu çözebiliriz.
