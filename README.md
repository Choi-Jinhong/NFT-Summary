<h1 align="center">NFT 토큰 만들어 보기</h1>
<h3 align="center">리액트를 활용하여 NFT 토큰 만들어보기</h3>
<p align="center">**해당 내용은 '리액트로 구현하는 블록체인 이더리움 ERC721(NFT)'강의 기반으로 만들었습니다.**</p>
<p align="center">**강의가 만들어진지 오래되어 실습은 다른 강의를 통해 진행할 예정.**</p>

## ERC-721?

### 주요 내용

ERC-721의 주요 내용은 '**대체 불가능(\*non-fungible)**' 과 '**소유권(자격) 증명(deed)**' 이다.</br>
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
/// @dev See https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md 
// Note: the ERC-165 identifier for this interface is 0x80ac58cd. 
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
✓ 명시적인 mutability </br>
```
function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
// external: visibility를 설정하는 자리
// payable: mutability를 설정하는 자리 구현의 목적에 맞게 변경, 'payable'은 eth를 같이 지불할 수 있는 함수를 나타냄
```
✓ 토큰 ID는 애플리케이션이 결정. 이더리움 내에서 유일한 값(컨트랙트 주소+토큰 ID)</br>
*mutability: 상태 변수를 변경할 수 있는 함수인지의 여부*

---

## ERC-721 인터페이스

### ERC165
컨트랙트가 표준 인터페이스를 구현한 것인지 확인하기 위한 "Standard Interface Detection"
> 자신이 사용한 표준 인터페이스에 대해 boolean 값으로 return 하라는 의미
- 예제
ERC-721 표준 인터페이스를 구현하는 컨트랙트는 ERC-165를 구현하여 그 여부를 확인할 수 있어야 한다는 규칙이 있어야 한다.
> ERC-721을 구현할 때에는 반드시 ERC-165를 구현해야 한다느 표준이 있기 때문에 이를 ERC-165를 통해 나타냄
>
> ERC-721에서 구현한 함수의 시그니처를 모두 XOR연산 했을 때의 값(16진수로 표현)을 ERC-165에 넣었을 때 반환 값이 true면 해당 표준으로 구현했다라는 의미

- 코드
```
supportedInterfaces[0x80ac58cd] = true; //ERC721
function supportsInterface(bytes4 interfaceID) external view returns (bool){ return supportedInterfaces[interfaceID];
}
```

### ERC721 이벤트
3개의 이벤트 - 이벤트 발생 시 화면(UI)에 알림용</br>
✓ Transffer: 소유권 이전 시 발생 </br>
✓ Approval: 소유권 이전 승인 시 발생(하나의 토큰에 대해)</br>
✓ ApprovalAll: 소유권 이전 승인 시 발생(모든 토큰에 대해)</br>

### ERC721 함수
9개의 함수 
✓ balanceOf(address _owner): 소유 계정의 토큰 수 <br/>
✓ ownerOf(uint256 _tokenId): 토큰의 소유 계정<br/>
✓ **safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes data)**: 소유권 이전, _to가 컨트랙트일 때(code size > 0)는 그 컨트랙트의 onERC721Received 함수를 호출하여 그것이 함수 selector와 일치하는지 확인<br/>
✓ **safeTransferFrom(address _from, address _to, uint256 _tokenId)**: 소유권 이전, _to가 컨트랙트일 때(code size > 0)는 그 컨트랙트의 onERC721Received 함수를 호출하여 그것이 함수 selector와 일치하는지 확인<br/>
✓ **transferFrom(address _from, address _to, uint256 _tokenId)**: 소유권 이전<br/>
✓ approve(address _approve, uint256 _token_id): 소유권 이전 승인 계정 지정(토큰 하나)<br/>
✓ setApproveForAll(address _operator, bool _approved): 소유권 이전 승인 계정 지정(토큰 모두)<br/>
✓ getApproved(uint256 _tokenId): 소유권 이전 승인 계정 조회<br/>
✓ isApprovedForAll(address _owner, address _operator): 소유권 이전 승인 여부 조회<br/>

### ERC721Enumerable
> 확장 인터페이스
>
> 발행된 토큰을 쓸 수 있는 또는 인덱싱 할 수 있는 함수들을 정의하는 인터페이스

✓ totalSupply() : 발행된 *유효한 토큰의 총 수<br/>
> *유효한 토큰: 소유한 계정이 존재하는 토큰

✓ tokenByIndex(uint256 _index) : _index에 해당하는 토큰ID 인덱스는 _index < totalSupply()<br/>
> 발행된 토큰에 대한 순서를 매기는 함수
>
> 꼭 발행된 순서대로 인덱스가 구성될 필요는 없다.
>
> 해당 인덱스는 totalSupply()보다 적어야 한다.
>
> 기존에 있던 토큰이 삭제되고 인덱스가 totalSupply()의 값보다 클 경우 이를 수정하는 로직을 구성해줘야 함

✓ tokenOfOwnerByIndex(address _owner, uint256 _index) : 소유 계정이 가진 토큰 중 인덱스에 해당하는 토큰ID<br/>

### ERC721Metadata
✓ name(): 토큰 이름</br>
✓ symbol() : 토큰 심볼</br>
✓ tokenURI(uint256 _tokenId) : 토큰 ID가 가리키는 리소스 정보(JSON)</br>
> ERC721은 단순히 토큰의 이름만 부여된 것이고 실제 어떤 자산을 가르키고 있는 지는 나와있지 않음
>
> 블록체인에 기재되어있는 자산이 아닌 이상 오프체인에 있는 내용은 알 수 없음
>
> return 형태는 URL, IPFS(탈중앙화된 파일 시스템에 존재하는 해당 파일의 해쉬값), JSON 스키마

---

## ERC721 인터페이스 구현 로직
*해당 내용은 실제 구현을 어떻게 할 것인지에 대한 내용을 나타내고 있어, 최대한 개념적인 부분만 정리할 예정*

### 토큰 Enumeration
tokenByIndex()(인덱스를 통해 토큰을 조회/토큰ID를 통해 인덱스 조회)를 위해서 만들어야할 구조<br/>
✓ unit256[] public 구조명;: tokeyByIndex에서 사용</br>
✓ mapping(uint256 => uint256) private 구조명;: 토큰 ID로 인덱스를 조회할 때 사용</br>

### 이러한 표준이 필요한 이유는?
> totalSupply()에서 _index < totalSupply()한 규칙 떄문에 다음과 같은 표준을 만들어야 함

### 토큰의 삭제 과정
1. 폐기되는 토큰 인덱스와 마지막 토큰 인덱스를 구한다.
2. 폐기되는 토큰 인덱스에 마지막 인덱스의 토큰 아이디를 업데이트한다.
3. 마지막 인덱스의 토큰 아이디의 인덱스를 바뀐 인덱스로 업데이트한다.
4. 배열의 길이를 줄인다.
```
allValidTokenIds.length = allValidTokenIds.length.sub(1); // 가변 길이 배열에선 길이를 변경할 수 있어 가능
```