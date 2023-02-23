# 誰でも任意のNFTをburnできる

## カテゴリー
👑 | Access Control

## バグの詳細
``` solidity
  //burn
  function burn(uint256 tokenId) external virtual {
    _burn(tokenId);
  }
```
Ref: https://etherscan.io/address/0xd93704f2a0ea3db109de194d4a51ff3e5e77cefd#code#L3745
``` solidity
  function _burn(uint256 tokenId) internal virtual {
      address from = ownerOf(tokenId);
      _beforeTokenTransfers(from, address(0), tokenId, 1);
      _burnedToken.set(tokenId);

      emit Transfer(from, address(0), tokenId);

      _afterTokenTransfers(from, address(0), tokenId, 1);
  }
```
Ref: https://etherscan.io/address/0xd93704f2a0ea3db109de194d4a51ff3e5e77cefd#code#L2610

`burn`関数の修飾子は`external`である
そのためどんな人でもetherscanなどでこの関数を実行することができる。
`_burn`関数では入力された`tokenId`のownerが`msg.sender`だったかどうかの確認がないため、ユーザーは任意のNFTを何枚でも`burn`できる

## インパクト
全てのNFTをburnすることができた
この画像から考えると被害額は769ETHになる
![image](https://user-images.githubusercontent.com/84496536/220872463-d05679d1-68af-4e0b-b013-842334bbfd8f.png)

## 修正方法

``` solidity 
    // Before
    function _burn(uint256 tokenId) internal virtual {
        address from = ownerOf(tokenId);
        _beforeTokenTransfers(from, address(0), tokenId, 1);
        _burnedToken.set(tokenId);
        
        emit Transfer(from, address(0), tokenId);

        _afterTokenTransfers(from, address(0), tokenId, 1);
    }

    // After
    function _burn(uint256 tokenId) internal virtual {
        address from = ownerOf(tokenId);
        require(from == msg.sender,"ERROR"); /* add */
        _beforeTokenTransfers(from, address(0), tokenId, 1);
        _burnedToken.set(tokenId);
        
        emit Transfer(from, address(0), tokenId);

        _afterTokenTransfers(from, address(0), tokenId, 1);
    }
```

## 類似のバグ
このバグのようにユーザーに対して投票の権利を`delegate`する際にdelegate元のユーザーから投票の権利をburnしなければ、無限に投票できてしまうというバグも存在する
https://github.com/code-423n4/2022-05-velodrome-findings/issues/154

こちらからは以上です
著者 Tomosuke Chiba : https://twitter.com/tom_eth_dev
