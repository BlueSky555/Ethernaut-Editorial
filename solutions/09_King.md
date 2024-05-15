# ETHERNAUT SORU 9: KING

> The contract below represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new king. On such an event, the overthrown king gets paid the new prize, making a bit of ether in the process! As ponzi as it gets xD
>
> Such a fun game. Your goal is to break it.
>
> When you submit the instance back to the level, the level is going to reclaim kingship. You will beat the level if you can avoid such a self proclamation.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {
    address king;
    uint256 public prize;
    address public owner;

    constructor() payable {
        owner = msg.sender;
        king = msg.sender;
        prize = msg.value;
    }

    receive() external payable {
        require(msg.value >= prize || msg.sender == owner);
        payable(king).transfer(msg.value);
        king = msg.sender;
        prize = msg.value;
    }

    function _king() public view returns (address) {
        return king;
    }
}
```

Bu soruda git gide kâra geçen bir kontrakt var. Ether gönderdiğinizde eğer en son Ether gönderenden fazla bir miktar gönderir iseniz eski krala o para geçer ve yeni kral siz olursunuz. 
Soruda bizden istenen ise kontraktı bozmak, çalışamaz hale getirmek.

Bu kontraktın açığı ise transfer fonksiyonunda. Başarılı bir transfer gerçekleşmeden kral değişmiyor. Böylelikle eğer krala transfer yapamaz iseniz `revert` oluyor ve fonksiyon kilitleniyor. Bunu yapmak için Ether kabul etmeyen bir kontrakt yazıp onunla kral olabiliriz. Böylelikle bir sonraki kişi kral olmaya çalıştığında bize Ether göndermeye çalışır ve gönderemediğinde `revert` durumu oluşur ve fonksiyon iptal edilir. Böylelikle fonksiyon çağırılamaz bir duruma düşer. Bunun için örnek bir kod.

```solidity
// SPDX-License-Identifier: MIT
pragma  solidity  ^0.8.0;

contract Hack {
function gonder(address  payable a)  public  payable  {
a.call{value:  msg.value}("");

   }
}
```
Bu kod ile oluşturulan kontrakta sorunun adresini veriyoruz. Gönderdiğimiz miktarı soruya gönderiyor ve kral oluyoruz. Bundan sonra her kral olmak isteyen kişi soru kontraktı vesilesiyle bize(yani eski krala) Ether göndermek zorunda. Ama bizim kontraktımızda bir `Receive` ya da `Fallback` fonksiyonu yok. Bu yüzden EVM `revert` komutu veriyor ve bu şekilde hep kral kalmış oluyoruz.
