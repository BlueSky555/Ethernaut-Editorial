# ETHERNAUT SORU 29: SWITCH

> Just have to flip the switch. Can't be that hard, right? ...right??

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Switch {
    bool public switchOn; // switch is off
    bytes4 public offSelector = bytes4(keccak256("turnSwitchOff()"));

    modifier onlyThis() {
        require(msg.sender == address(this), "Only the contract can call this");
        _;
    }

    modifier onlyOff() {
        // we use a complex data type to put in memory
        bytes32[1] memory selector;
        // check that the calldata at position 68 (location of _data)
        assembly {
            calldatacopy(selector, 68, 4) // grab function selector from calldata
        }
        require(selector[0] == offSelector, "Can only call the turnOffSwitch function");
        _;
    }

    function flipSwitch(bytes memory _data) public onlyOff {
        (bool success,) = address(this).call(_data);
        require(success, "call failed :(");
    }

    function turnSwitchOn() public onlyThis {
        switchOn = true;
    }

    function turnSwitchOff() public onlyThis {
        switchOn = false;
    }
}
```

### Genel Gözlem

Bu soruyu çözmek için `switchOn` adlı değişkeni `true` yapmalıyız. Soruda verilen fonksiyonlara bakarsak `turnSwitchOn` ve `turnSwitchOff` adlı iki fonksiyonun bu değişkeni değiştirdiğini görüyoruz. Ancak bu iki fonksiyon `onlyThis` adlı "modifier"a sahip ve sadece kontrakt içinden çağırılabiliyor. Kodda bulunan `flipSwitch` fonksiyonu tam da bu işe uygun görünüyor. Ancak bu fonksiyonda da `onlyOff` adlı bir modifier var. Bu modifier'ı geçersek soru biter.

### Çözüm

Bahsedilen `flipSwitch` fonksiyonu verilen argüman ile `this` kontraktını çağırmaktadır. Yani argüman olarak `turnSwitchOff` fonksiyonunun seçicisini (selector) vererek `turnSwitchOff` fonksiyonunu çağırabiliriz. Ancak bu fonksiyonun sahip olduğu modifier buna engel oluyor.

Bu modifier `CALLDATA`'nın 68. byte'ından başlayarak 4 byte'lık okuma yapıyor. Ve eğer okuduğu değer `turnSwitchOn` fonksiyonunun seçicisi değilse fonksiyon çalışmıyor ve geri alınıyor. Bunu daha iyi açıklamak gerekirse şunu söyleyebiliriz. Eğer `flipSwitch` fonksiyonunu `turnSwitchOff` fonksiyonun seçicisi ile çağırmak istersem yapacağım call'ın `CALLDATA`'sı aşağıdaki gibi olur:

```
30c13ade // 0-3 BYTES, fonksiyon seçicisi (flipSwitch)
0000000000000000000000000000000000000000000000000000000000000020 // 4-35 BYTES, data argümanının başlangıç offseti
0000000000000000000000000000000000000000000000000000000000000004 // 36-67, data (bytes) uzunluğu
76227e1200000000000000000000000000000000000000000000000000000000 // 68-99, data
```

68-71 byteları `turnSwitchOn`'un seçicisi olmadığından fonksiyon çalışmaz.

Bu kısmı daha iyi anlamak için [bu dokümentasyon](https://docs.soliditylang.org/en/v0.8.19/abi-spec.html#argument-encoding) önerilir.

Yani bir şekilde, `CALLDATA`'mızın 68. byte'ında `turnSwitchOff`'un seçicisi olmalı ve aynı zamanda bu `CALLDATA` `turnSwitchOn`'u çağırmalı. Bunun için offseti 32 (0x20) byte yerine 96 (0x60) byte yaparız. Böylece 4-99 byteları boş kalır. 68-71 byteları bu boşlukta olduğu için oraya istediğimiz değeri yerleştiririz.

Yeni `CALLDATA` aşağıdaki gibi olur:

```
30c13ade // 0-3 BYTES, fonksiyon seçicisi (flipSwitch)
0000000000000000000000000000000000000000000000000000000000000060 // 4-35 BYTES, data argümanının başlangıç offseti
0000000000000000000000000000000000000000000000000000000000000004 // 36-67, BOŞ KALDI
20606e1500000000000000000000000000000000000000000000000000000000 // 68-99, BOŞ KALDI (turnSwitchOn'u yerleştirdik)
0000000000000000000000000000000000000000000000000000000000000004 // 36-67, BOŞ
76227e1200000000000000000000000000000000000000000000000000000000 // 68-99, data
```

Kontrakt bu data ile çalıştırıldığında soru çözülür.
