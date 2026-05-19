## 1. addrTransWrite
### 1. 물리 주소가 없는 경우 (신규 할당)
* **a.** 입력으로 들어온 `logicalSliceAddr`을 블럭 단위로 나눠서 `logicalBlockAddr`을 구한다.
* **b.** `logicalBlockAddr`을 매핑 테이블에 대입하여 `virtualBlockAddr`(물리 블록 주소)를 구한다.
* **c-1.** 물리 주소가 없다면, 페이지 할당 가능한 die 번호를 구하고 이 die 번호를 가지고 `GetFromFbList` 함수를 호출하여 free block을 구한다.
* **d.** free block이 주어졌다면 `Vorg2VsaTranslation` 함수를 호출하여 블록의 첫번째 페이지 주소(즉 `virtualBlockAddr`)를 구한다. (이때 page 번호는 `0`)
* **e.** `virtualBlockMapPtr`에서 block 정보들 초기화한다. (코드 참고)
* **f.** 최종 물리 주소를 반환한다.

### 2. 매핑된 주소가 있는 경우 (기존 블록 사용)
* **c-2.** 매핑된 주소가 있다면, 그 물리 주소를 가지고 `Vsa2dieTranslation`, `Vsa2VblockTrans` 함수를 호출하여 die, block 번호를 구한다.
  * *참고:* `virtualBlockAddr`은 결국 블록의 첫번째 페이지 주소이기 때문에 대입이 가능하다.
* **d.** **페이지 번호 계산:** 현재 할당된 페이지의 **다음 번호**를 사용한다. (`blockMap`에서 `currentPage` 정보가 있는데 그냥 1을 올리면 됨. 물론 범위를 넘어서면 에러 반환)
  * *이유:* 물리적으로 페이지를 순차적으로 할당해야 하기 때문이다. (조교님 설명 내용)
* **e.** `Vorg2VsaTranslation` 함수로 최종 페이지 주소를 구하고 반환한다.

---

## 2. addrTransRead

### 기본 동작 흐름
* `addrTransWrite`와 과정은 동일하다. 
* `logicalSliceAddr`를 `logicalBlockAddr`로 바꾸고 매핑으로 물리 주소를 구한다.
* **주소가 없다면:** 그냥 에러를 반환한다.
* **주소가 있다면:** die와 block 번호를 구한다.

### 페이지 번호(pageNo) 계산 방식
주소가 존재할 때, 페이지 번호는 아래와 같은 공식으로 구한다.

$$\\text{pageNo} = (\\text{logicalSliceAddr} \\% \\text{SLICES\\_PER\\_BLOCK}) \\% \\text{totalPages}$$

* 여기서 `totalPages`는 현재 블록 내 페이지 개수를 의미한다.
* `logicalSliceAddr % SLICES_PER_BLOCK`이 **블록 내 페이지 위치**라면, 그 이후에 `totalPages`로 모듈로(`%`) 연산을 수행한 것은 **현재 페이지 내에서의 위치**가 어디인지를 나타낸다.

### 어거지 포인트
* 물리적 제약으로 인해 페이지를 순차적으로 써야만 한다.
* 이 때문에 **기대되는 페이지 위치**와 **실제 페이지 위치**가 달라지게 되므로, **랜덤 쓰기(Random Write) 모드에서는 실패가 많을 것**으로 예상된다. (솔직히 확실하진 않음)
* 어쨌든 최종적으로는 `Vorg2VsaTranslation` 함수로 최종 페이지 주소를 구하고 반환한다.
"""
