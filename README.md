# 외식 산업 AI 챗봇 고도화

## 프로젝트 시나리오

- 기존 고객 이탈 방지와 AI기술 도입을 통한 투자 유치 전략을 수행하기 위해 AI 챗봇이 개발 되었습니다. 하지만 사내 테스트를 진행 해본 결과 만족스러운 성능을 가지고 있지 못함을 확인하였고, 정식 서비스에 활용하기 위해 다각도고도화가 필요함을 확인하였습니다.
  - 이를 서비스 하려는 업체는 임의로 '온더보더'(https://ontheborder.co.kr/)로 지정
  - 목적 달성을 위한 목표
    1. 자연어 데이터의 탐색적 데이터 분석
    2. AI 모델 성능 향상
    3. UIUX 개선

## 기존 서비스 파악

- 프로세스
  1. 사용자에게서 주문 정보(데이터)를 수신
  2. 자연어 데이터 전처리
  3. AI 모델을 통한 입력 데이터의 의도 및 개체명 분석
  4. '학습 DB'에 저장된 답변을 검색
  5. 답변 출력
  6. 주문 정보에 맞는 답변 반환

- AI 모델
  - 의도 분석 모델과 개체명 분석 모델 두 종류 존재
  - 단순 CNN 구조이며 모델 성능은 98%~99% 정도로 측정
  - 실제 사용시 측정된 만큼의 성능을 보이지 못함
    - 직전의 판단 결과가 이후의 판단에도 영향을 미치는 패턴 발생
    - 데이터 라벨링이 잘못되거나 목적과 연관이 없는 데이터, 최신화 되지 못한 데이터 등의 학습 데이터 문제

- UIUX
  - Docker를 사용해 Bot 서버를 실행하고 Client 파일을 실행하는 방식
  - 적절한 UI가 없으며 터미널에서 사용
  - Docker 터미널의 문제로 한글 디코드 오류가 종종 발생

 ## 모델 개선
 
  - 머신러닝 모델 성능 향상을 위한 조정
    - LSTM, 양방향 LSTM 등 다양한 구조 적용 후 파라미터를 조절하여 테스트 했지만 데이터 자체의 문제점 등 상술 된 문제를 해결하지 못했다.
    - 임의로 가게를 설정하여 좁혀진 데이터 범위, 외식 플랫폼의 AI 챗봇이라는 기능의 경우의 수가 한정 됨에서 착안하여 룰 베이스 모델로 전환하기로 결정
   
  - 데이터
    - 기능의 갯수 = 의도 데이터 라벨의 갯수 인 구조이며 기능 추가를 위해 새로운 라벨을 추가하여 9개로 데이터 재라벨링
    - 라벨 오류 수정 및 외식 산업과 관련 없는 데이터 삭제
    - 외식 산업에서 발생 할 것으로 예측되는 상황과 온더보더 메뉴에 적합한 데이터 약 30,000개 추가 생성
    - 개체명 분석 모델을 위해 국립국어원 2021 개체명 분석 데이터 사용
      - 위키피디아 데이터 기반
      - 약 300,000개의 항목을 Dictionary 형태로 라벨링한 데이터
      - 150여개의 세부분류된 개체명 분
      - 이를 필요에 따라 단어 추가, 재 라벨링, 개체명 축소/통폐합 등의 가공 수
   
  - 의도모델 개선
    - ML 모델에서 룰 베이스 모델로 전환
    - 같은 키워드가 조합에 따라 여러 의도로 사용 될 수 있는 가능성이 존재
        1. 하나의 라벨을 체크 할 때 여러 단어를 동시에 체크
        2. 특정 클래스 소속 여부를 수 차례 체크
        3. 우선순위를 설정

  - 개체명 모델 개선   
    - ML 모델에서 Dictionary data 참조로 전환
    - 동음이의어에 취약하나 범위가 외식 산업으로 한정 지어졌기 때문에 동음이의어 처리가 크게 문제되지 않을 것으로 판단
    - 연산량이 월등히 낮은 장점
   
  - 분류를 위한 개선
    - 기존 분류는 모델을 통해 의도명, 개체명을 얻어내고 개체명에서 문장의 핵심 단어를 복구하여 처리
      - 문장 '3시 예약': 3시 -> (개체명-시간), (의도명-예약)
    - 세세한 분류를 위해 의도태그(Intent_tag) 추가
    - (예시 적을것)

## 웹 제작 및 배포
  - 웹 UI 및 기능
    - 챗봇: 메시지 수신 및 송신, 챗봇을 통한 기능 제어
    - 예약: 예약, 예약 취소.
      - ex)"3시 2명 예약합니다.", "3시 예약 취소합니다."
    - 주문: 주문, 주문 취소.
    - 메뉴 안내: 메뉴 소개, 가격 안내
    - 메뉴 추천
    - 메뉴판
    - 문의: FAQ, 지점 정보, 이벤트

  - (그림 넣으면 좋을 듯?)
  - 봇 서버: 개발된 모델을 사용하는 서버.
    - 질문을 수신하면 의도명, 개체명, 의도태그를 파악하고 그에 맞는 답변을 검색, JSON 형태로 송신
  - 웹 서버: Flask를 사용하며, 웹 UI와 봇 서버를 연결하는 중간다리로 사용된다.
    - 웹 UI에서 질문을 수신 - 해당 질문을 봇 서버에 송신 - 답변 데이터를 봇 서버에서 수신 - 답변 데이터를 처리해서 웹 UI로 송신
  - Docker를 사용하고 있었으며, 내부에 봇 서버와 웹 서버 존재

  - (도커 사용 포트 문제 해결)
  - (웹UI 기능 소개해야 할듯)
  - (배포 방식 소개?)

## 결과

  - 달성 목표
    - 메뉴 최신화
    - 챗봇 기능 다양화
    - 모델 개선을 통한 성능과 동작 속도 문제 해결
    - UIUX 개선
   
  - 
