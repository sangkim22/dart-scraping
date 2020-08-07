# DART 공시파일 HTML로 저장/입수 Data의 1차 가공
## 개요
FSS는 오픈API(https://opendart.fss.or.kr/intro/main.do) 에서 Macro Data를 제공하고 있다. 여기서 제시하는 여러 코드는 FSS 제공 Data의 범위(종류, 기간)를 벗어난 정보가 필요할 때 사용한다.


예를 들면 외부감사실시내역 상 감사시간이나 사업보고서 감사에 관한 사항에 제시되는 감사보수 정보 등이 그렇다. 

## 코드 종류
### Disclosure-to-HTML
#### 코드: scraper
DART 공시파일 중, 예를 들면, 사업보고서는 다음과 같은 깊이가 3인 문서 계층 구조를 사용하고 있다.


- 사업보고서
    - 대표이사의 확인 (*)
    - ... 
- 감사보고서
    - 독립된 감사인의 감사보고서 (*)
    - (첨부)재무제표 (*)
        - 재무상태표
        - 손익계산서
        - ...
    - ...

코드는 특정일에 공시된 사업보고서 전체를 하나의 HTML로 입수하지 않는다.
가장 큰 이유는 모든 계층의 문서가 자신만의 주소를 갖고 있기 때문이다. 


그렇지만 이 코드가 가장 깊은 단계까지 내려가서 HTML을 긁어오는 것은 아니다.
입수의 용의성과 실용성을 고려하여 위 예시의 * 부분을 입수 단위로한다.


#### 사용방법

#### 입수파일명
