# ETHERNAUT SORU 12: Privacy

> The creator of this contract was careful enough to protect the sensitive areas of its storage.
>
>Unlock this contract to beat the level.
>
>Things that might help:
>
>-   Understanding how storage works
>-   Understanding how parameter parsing works
>-   Understanding how casting works
>
>Tips:
>
>-   Remember that metamask is just a commodity. Use another tool if it is presenting problems. Advanced gameplay could involve using remix, or your own web3 provider.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Privacy {
    bool public locked = true;
    uint256 public ID = block.timestamp;
    uint8 private flattening = 10;
    uint8 private denomination = 255;
    uint16 private awkwardness = uint16(block.timestamp);
    bytes32[3] private data;

    constructor(bytes32[3] memory _data) {
        data = _data;
    }

    function unlock(bytes16 _key) public {
        require(_key == bytes16(data[2]));
        locked = false;
    }

    /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
    */
}
```

Bu soruda bize büyük ve gizlenmiş bir kontrakt verirmiş. Amacımız ise `locked` değişkenin değerini değiştirmek. Bunun için soru zaten bize gerekli kısımı vermiş ve geri kalan kontrakı verme gereksinimi duymamış. 

Soruyu çözmek için en bariz yöntem `unlock` fonksiyonunu çalıştırmak. Bu fonksiyon girdi olarak 16 bayt alıyor ve bunu `data` dizisinin 3. elemanının ilk 16 baytı ile karşılaştırıyor. Eğer bu data değişkenini öğrenebilirsek soruyu çözebilecek bir girdi üretebiliriz. 

`Data` değişkeni gizli olsa da sonuçta blokzincirde depolanmakta. Bu değişkene temel bir blokzincir uygulaması ile erişip görüntüleyebiliriz. Bunun için zaten ethernautun içine gömülmüş `web3` kütüphanesini kullanacağız.

Bu değerlere erişmeden önce değişkenlerin nasıl saklandığını bilmemiz gerekiyor. Ethereum'da 32 baytlık bölümler şeklinde hafıza bölümleri vardır. Bu bölümlere veri tek seferde yazılır ya da okunur. Bu bölümler hep 32 baytlıktır yani biz 1 bit bile depolarsak bu alanın hepsini kullanmış sayılırız. Bundan dolayı küçük depolama gerektiren bazı değişkenler için yerden tasarruf mekanizması vardır. Bu mekanizma küçük değişkenleri birlikte gruplayarak tasarruf sağlar. Önrek olarak `uint8 private flattening` sadece 8 bayt kullanır ve 32 bayt israf olmasın diye yanına diğer değişkenler konulur. Bu değişkeni içerisinde barındıran bölüm `0x000000000000000000000000000000000000000000000000000000001068ff0a` şeklindedir ve `0a` `flattening` değişkeninin 10 değerine karşılıktır. Diğer değişkenler ise bu alana sığdıkları içi buraya dahil edilmiştir. Bundan daha büyük olan bir değişken (Ör. `Data`) buraya sığamayacağı için farklı bir alanda olmak zorundadır. Bu alanlar ardışıktır ve böylelikle kolaylıkla hepsine erişebiliriz.

Bu değerlere direkt olarak Ethernaut'u açarak tarayıcımızın konsolundan `web3.eth.getStorageAt("kontrakt adresi", indis)` şeklinde erişebiliriz. 0, 1 ve 2. indiste işimize yaramayacak değerler yer almaktadır. 3. indisten itibaren ise `Data` değişkeni başlar. Bu değişken 3 alan kaplamaktadır ve bize son alandaki değeri lazımdır çünkü kontrakt bu değişkenin son indisini kullanmaktadır. Bunun için 5. indisi kullanabiliriz ve bize `0xcd0c497558547109efdcb18d2907ed6661a91e1947ab15f933e3c054a2b5563d` değeriini verecektir(*Bu değer sorunun nasıl hazırlandığına bağlı olarak her kontrakt için farklı olabilir*). Bu değerin ilk 16 baytı yani ilk 32 karakterini alıp `contract.unlock("anahtar")` şeklinde `unlock` fonksiyonunu çalıştırırsak `locked` değişkeninin değeri değişecektir ve sorumuz çözülmüş olacaktır.   
