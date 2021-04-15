<h1 align="center">NFT 토큰 만들어 보기</h1>
<h3 align="center">리액트를 활용하여 NFT 토큰 만들어보기</h3>
<p align="center">**해당 내용은 '리액트로 구현하는 블록체인 이더리움 ERC721(NFT)'강의 기반으로 만들었습니다.**</p>

## ERC-721?

### 주요 내용

ERC-721의 주요 내용은 '**대체 불가능(\*non-fungible)**' 과 '**소유권(자격) 증명(deed)**' 이다.
즉, 대체할 수 없는 것들에 대해 자격을 증명해주는 것을 위해 만들어진 표준이다.

_non-fungible: 가치가 서로 상이한 유일한 것_

### 어떻게 정의 내려지는가?

**solidity interface**로 구성되어있다.

1. 구현 부에 아무것도 정의되어져 있지 않다.(정의되어 있을 경우 추상 컨트렉트)
2. 다중 상속이 가능하다.(\*확장 인터페이스가 사용가능)

_확장 인터페이스의 종류: ERC-721TokenReceiver, ERC-721Metadata, ERC-721Enumerable_

### 메인 인터페이스

```
pragma solidity ^0.4.20;
/// @title ERC-721 Non-Fungible Token Standard
/// @dev See https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md /// Note: the ERC-165 identifier for this interface is 0x80ac58cd.
         interface ERC721 /* is ERC165 */ {
event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId); event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);
function balanceOf(address _owner) external view returns (uint256);
function ownerOf(uint256 _tokenId) external view returns (address);
function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes data) external payable; function safeTransferFrom(address _from, address _to, uint256 _tokenId) external payable;
function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
function approve(address _approved, uint256 _tokenId) external payable;
function setApprovalForAll(address _operator, bool _approved) external;
function getApproved(uint256 _tokenId) external view returns (address);
function isApprovedForAll(address _owner, address _operator) external view returns (bool);
}
interface ERC165 {
function supportsInterface(bytes4 interfaceID) external view returns (bool);
}
```

### ERC-721Metadata, ERC-721Enumerable

```
interface ERC721Metadata /* is ERC721 */ {
function name() external view returns (string _name);
function symbol() external view returns (string _symbol);
function tokenURI(uint256 _tokenId) external view returns (string);
}
interface ERC721Enumerable /* is ERC721 */ {
function totalSupply() external view returns (uint256);
function tokenByIndex(uint256 _index) external view returns (uint256);
function tokenOfOwnerByIndex(address _owner, uint256 _index) external view returns (uint256);
}
```

### 추가적인 기타 사항
✓ 솔리디티 컴파일러 버전에 따른 영향 - 함수 정의 변경 </br>
✓ 명시적인 mutability 
```
function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
// external: visibility를 설정하는 자리
// payable: mutability를 설정하는 자리 구현의 목적에 맞게 변경, 'payable'은 eth를 같이 지불할 수 있는 함수를 나타냄
```
✓ 토큰 ID는 애플리케이션이 결정. 이더리움 내에서 유일한 값(컨트랙트 주소+토큰 ID)
*mutability: 상태 변수를 변경할 수 있는 함수인지의 여부*