# DART 공시파일 입수
## 개요
### 금융감독원 오픈 API
FSS에 따르면 DART 오픈API는 다음과 같은 목적으로 활용 가능하다. 

- DART 공시원문 활용 : DART에 공시되는 공시보고서의 원문을 XML형식으로 다운로드받아 원하는 자료를 자유롭게 추출하여 사용할 수 있습니다.   
- 주요공시 및 재무정보 제공 : 사업보고서 주요항목 및 주요재무계정, 지분보고서 종합정보를 데이터 형식으로 바로 활용할 수 있습니다.   
- 대용량 재무정보 제공 : 상장법인(금융업 제외)에서 제출한 전체 재무제표를 분기별로 다운로드 받을 수 있습니다.    
(https://opendart.fss.or.kr/intro/main.do)

오픈API의 이용한도는 개인은 일 10,000건이고 기업은 무제한으로 제공되는 공시목록과 기업개황을 제외한 나머지는 일 10,000건이다. 또한 과도한 네트워크 접속(분당 100회 이상)은 서비스 이용이 제한된다. 실제, 분당 100회 이상 요청을 보내면 IP가 일정기간 차단된다. 그러나, 일당 10,000건(최소 100분)의 조회는 연간 약 3천건이 공시되는 사업보고서와, 약 3만건이 공시되는 감사보고서를 입수하기에는 충분하다. 친지들에게 부탁하여 api_key를 여러개 확보하여 병렬로 입수한다면 1998년부터 20여년의 공시를 단기간에 모두 입수하는 것도 가능하다. 

오픈API의 사용은 다음에 링크하는 OpenDartReader를 사용하는 것이 유용하다. (https://github.com/FinanceData/OpenDartReader#opendartreader)

### From Disclosure to Data
이하 소개하는 코드는 다음과 같은 이유로 오픈API를 보완한다. 

> 첫번째, 입수 코드는 입수단위가 목적에 맞게 분리되어 있다. FSS 오픈 API의 원문정보 자료의 경우 사업보고서와 감사보고서가 한덩어리로 입수되어 후가공에서 추가 분리가 필요하다.   
두번째, 제공하지 않는 정보를 입수하려는 목적에서 사용할 수 있다. 예를 들어, 외부감사실시내역 상 감사시간이나 사업보고서 감사에 관한 사항에 제시되는 감사보수 정보 등의 입수가 필요할 때가 그렇다.

[그림 1] 공시자료에서 가공된 table까지 흐름도

<img src="https://user-images.githubusercontent.com/33425859/102588905-84236280-4151-11eb-9b97-28a015a55db3.png" width="100%"></img>

## 코드 종류
### (A) Disclosure-to-HTML (Python 3)

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
가장 큰 이유는 모든 계층의 문서가 자신만의 주소를 갖고 있어 서버 요청이 전체 단위가 아니라는 점에 있다.
그렇지만 이 코드가 가장 깊은 단계까지 내려가서 HTML을 긁어오는 것은 아니다. 
그렇게 하는 경우 중복 정보를 입수하게 되기도 하지만, 주소를 찾을 때 예외에 취약한 정규표현식을 사용하여야 한다는 점을 고려하였다.


결국 입수와 후처리의 용이성을 고려하여 위 예시의 * 부분을 개별 단위로 입수한다.


#### 사용방법
Anaconda를 설치하면 따라오는 통합개발환경인 Spyder에 코드를 붙여서 쓴다. Anaconda 설치시 BeautifulSoup, requests 같은 라이브러리가 따라오기 때문에 사용이 간편하다.
scraper 코드의 아래 부분을 필요에 맞게 조정하여 입수한다. 필요하다면 코드를 수정하여 코드 실행 후 콘솔에서 필요한 변수를 지정할 수 있다. (설명을 위해 Comment 삭제)

    targetYear = "2019"
    startDate = targetYear + "0101"
    endDate = targetYear + "1231"

    reportType = "A001" 

    delay = 1

    path = r"C:\Users\user\Desktop\\" + reportType + "_" + targetYear
    if os.path.exists(path) != True:
        os.makedirs(path)

    i = 1

<표1: 사용법 설명>
변수(예시) | 설명
--- | ---
targetYear = "2019" | 입수하고자 하는 보고서가 등록된 연도
startDate = targetYear + "0101" | 입력된 MMDD 형식의 날짜부터 입수
endDate = targetYear + "1231" | 입력된 MMDD 형식의 날짜까지 입수
reportType = "A001" | 보고서 타입 (A001 사업보고서, A002 반기보고서, A003 분기보고서, F001 감사보고서, F002 연결감사보고서, F004 회계법인사업보고서. 전체 리스트 DART 오픈API 개발가이드, https://github.com/nKiNk/scrapdart 참조)
path = r"C:\Users\user\Desktop\\" + reportType + '\_' + targetYear | 로컬의 다운로드 경로를 지정. 후속 코드에서 해당 경로가 없는 경우 새로 생성
i = 1 | 다트 공시 화면은 조회 대상 리스트를 만들 때 (default로) 15건당 1페이지 생성. 입력값은 페이지 번호이며, 1 page부터 시작. 오류 등 이유로 코드가 중단되는 경우 i 변수에 들어가있는 값을 찾아 넣어서 중단지점의 근처에서 재시작 


#### 입수파일명
파일명은 입수 자료와 회사의 정보로 구성되어 있다. 아래에 예시 파일과 파일명에 포함된 정보를 설명하는 표를 작성하였다.


> A001_2019.12.31_대동고려삼_[기재정정]사업보고서_(2019.06)_2019.12.31 [정정] 사업보고서_I.회사의개요_178600_코넥스시장_110111-2481044_기타 식료품 제조업_2002-03-26_06월_.html



<표2: 파일명 설명>
예시 | 설명
--- | ---
A001 | 보고서 타입
2019.12.31 | 공시자료 접수일
대동고려삼 | 회사명
[기재정정]사업보고서 | 공시단위
(2019.06) | 공시자료 회계기간 종료월
2019.12. 31 [정정] 사업보고서 | 공시자료 문서명
I.회사의 개요 | 입수단위
178600 | 종목코드
코넥스시장 | 시장구분
110111-2481044 | 법인등록번호
기타 식료품 제조업 | 업종명
2002-03-26 | 설립일
06월 | 결산월


## 입수 데이터
### 설명
아래 표에 위의 코드를 활용하여 입수한 Data를 다운로드 할 수 있는 구글 드라이브 링크가 포함되어 있다.
DART 공시자료의 특성상 기재정정이 될 때 보고 전, 후 첨부문서가 모두 포함된다.
이로 인하여 입수기간에 따라 중복정보가 입수될 수 있다.
중복 입수의 경우 위의 파일명에서 입수 단위의 고유키를 조합(예: 법인등록번호+공시일+보고서명)할 수 있으니 중복은 제거 가능하다.


# 입수 파일 가공
## 개요
위에서 설명한 절차로 입수된 데이터는 사용 목적에 맞는 전처리를 거쳐야 사용할 수 있다.  
