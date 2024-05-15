# ETHERNAUT SORU 8: VAULT

> Unlock the vault to pass the level!
```solidity
 // SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract Vault {
    bool public locked;
    bytes32 private password;
    
    constructor(bytes32 _password) {
        locked = true;
        password = _password;
    }
    
    function unlock(bytes32 _password) public {
        if (password == _password) {
            locked = false;
        }
    }
}
```

Bize burda verilen kod *Constructor* ile belirlenen şifre sağlandığında açılan bir kasanın kodudur. Doğru şifre ile `unlock` fonksiyonu çağrıldığında `locked` değişkenini *false* yapar ve kasanın kilidini açmış olur. Bizden istenen ise bu kilidi açmak ve bu değeri *false* yapmaktır.

Sorunun çözümü aslında oldukça basittir. Burdaki hata şifreyi kontrol eden sistemdedir. Sistem şifrenin gizli kalması durumunda güvenlidir ama blokzincirde hiçbir değişken gizli değildir. *Constructor*'da değer atanan `bytes32 private password` değeri blokzincirde depolanmaktadır. Bu veriye eriştikten sonra kolaylıkla kasanın kilidini açabiliriz. Bu veriye `web3.eth.getStorageAt({Kontraktın Adresi}, {Veri İndeksi})` ile tarayıcı konsolundan ulaşabiliriz. Veri indeksini bulmak için ise değişkenin kaçıncı sırada tanımlandığına bakıyoruz ve 2. sırada tanımlandığını görüyoruz. İndeksler 0'dan başladığı için oraya indeks olarak 1 yazmamız lazım. Bu komudun sonucu ise bize şifreyi hex biçiminde veriyor. Bu şifre ile `contract.unlock({Şifre})` fonksiyonunu çağırarak seviyeyi çözmüş oluyoruz.

![Çözüm](https://i.imgur.com/Tp6opIu.png)
