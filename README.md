# Moving_Average_Disparity_Strategy

- 한국투자증권 OPEN API의 모의투자 환경에서 삼성전자(005930)를 대상으로 하는 초단기 이격도 매매전략
- 전략은 한국투자증권의 github repository인 open-trading-api의 strategy_05를 참고하였음
- Prompt는 교수님의 notion을 참고하였음 (https://financial-engineering.notion.site/Prompts-32d7aa6d9f28800bb792c009879bf7c3)

## 1. 전략 설명

- 역추세 유형의 전략으로, 이동평균 비율로 과열과 침체판단. 과열이라고 판단 시 매도, 침체라고 판단 시 매수.
- 또한 익절 조건과 손절 조건을 설정하여 역추세 전략의 단점 보완.
- 초단기 전략이므로 3분이상 주식을 보유하지 않음.
- 1회 매수,매도 수량: 10주
- 물타기 금지

1. 매수: 최근 10개 현재가 평균보다 0.2% 이상 낮아지면 매수 (이격도 <= 99.8)

2. 매도: 최근 10개 현재가 평균보다 0.2% 이상 높아지면 매도 (이격도 >=100.2)

3. 익절: 매수가 대비 +0.6%면 청산

4. 손절: 매수가 대비 -0.4%면 청산

5. 시간청산: 3분 이상 보유하면 청산

- 이격도 = (현재가 / 최근 10개 현재가 평균) * 100


## 2. 폴더구조

```text
Mocking_Average_Disparity_Strategy
├── README.md              # 프로젝트 설명서
├── requirements.txt       # 필요한 Python 패키지 목록
├── token_cache.json       # 하루 동안 재사용할 access token 캐시 파일
│
├── main.py                # 프로그램 실행 시작점
├── config.py              # API 설정, 거래 전략 설정, 환경변수 로드
├── logger.py              # 로그 출력 설정
├── auth.py                # 한국투자증권 access token 발급 및 캐싱
├── api_client.py          # REST API GET/POST 요청 처리
├── market_data.py         # 현재가 조회 기능
├── account.py             # 계좌 잔고 및 보유 주식 조회
├── orders.py              # 시장가 매수/매도 주문 기능
└── trader.py              # 자동매매 핵심 로직
```


## 3. 코딩방법

- Prompt를 Chatgpt에 넣어서 얻은 파일을 현 repository에 업로드하였음.
- 추가로 프로그램을 실행하면서 코드를 수정하였음.

<img width="296" height="331" alt="image" src="https://github.com/user-attachments/assets/0f4eb22c-83a1-4e60-b152-757612ebae8e" />


## 4. 프로그램 실행

### 4-1. 한투 모의투자 계정
<img width="247" height="497" alt="image" src="https://github.com/user-attachments/assets/af26750f-2a22-49c9-bf19-d8c4b1baedea" />

### 4-2. Codespace
1. 토큰 발급, 최근 10개 현재가 수집
```text
2026-06-08 14:08:59 | INFO | samsung_auto_trader | Reusing cached access token for today.
2026-06-08 14:08:59 | INFO | samsung_auto_trader | Trader started. mock_only=True symbol=005930
2026-06-08 14:08:59 | INFO | samsung_auto_trader | Current price. symbol=005930 price=303750
2026-06-08 14:08:59 | INFO | samsung_auto_trader | Collecting price window. current_count=1 required=10
2026-06-08 14:09:11 | INFO | samsung_auto_trader | Current price. symbol=005930 price=303750
2026-06-08 14:09:11 | INFO | samsung_auto_trader | Collecting price window. current_count=2 required=10
2026-06-08 14:09:22 | INFO | samsung_auto_trader | Current price. symbol=005930 price=304000
2026-06-08 14:09:22 | INFO | samsung_auto_trader | Collecting price window. current_count=3 required=10
2026-06-08 14:09:34 | INFO | samsung_auto_trader | Current price. symbol=005930 price=304000
```

2. 최근 10개 현재가 수집 완료 후, ma(이동평균)과 disparity(이격도) 계산.
```text
2026-06-08 14:10:50 | INFO | samsung_auto_trader | Disparity calculated. price=303000 ma=303600.00 disparity=99.802
```

3. 계좌 조회: holding_qty = 보유주식  avg_price = 보유주식의 평단가  account_value = 총평가금액
```text
2026-06-08 14:10:55 | INFO | samsung_auto_trader | Account snapshot. symbol=005930 holding_qty=0 avg_price=0 account_value=499488472
```

4. 매수
   - 이격도가 99.775로, 매수기준인 99.8을 넘었으므로 매수.
   - 매수하기 전에 잔고에 주식이 있는지 확인. 주식이 없을 경우에만 매수. (물타기 없음)
```text
2026-06-08 12:55:21 | INFO | samsung_auto_trader | Disparity calculated. price=310500 ma=311200.00 disparity=99.775
2026-06-08 12:55:21 | INFO | samsung_auto_trader | Buy signal detected. disparity=99.775 threshold=99.800
2026-06-08 12:55:21 | INFO | samsung_auto_trader | Holdings before buy. quantity=0
2026-06-08 12:55:21 | INFO | samsung_auto_trader | Buy order request. symbol=005930 quantity=10
2026-06-08 12:55:35 | INFO | samsung_auto_trader | Account snapshot. symbol=005930 holding_qty=10 avg_price=310925.0 account_value=499605340
2026-06-08 12:55:35 | INFO | samsung_auto_trader | Holdings after buy. quantity=10
2026-06-08 12:55:35 | INFO | samsung_auto_trader | Buy execution seems to have occurred.
```

5. 매도(이격도조건)
   -이격도가 100.227로, 매도기준인 100.2를 넘었으므로 매도.
```text
2026-06-08 14:50:57 | INFO | samsung_auto_trader | Sell signal detected. disparity=100.227 return_rate=0.00336 holding_seconds=90.3
2026-06-08 14:50:57 | INFO | samsung_auto_trader | Holdings before sell. quantity=10
2026-06-08 14:50:57 | INFO | samsung_auto_trader | Sell order request. symbol=005930 quantity=10
2026-06-08 14:51:17 | INFO | samsung_auto_trader | Account snapshot. symbol=005930 holding_qty=0 avg_price=0 account_value=499454657
```

6. 매도(손절조건)
   - 손실이 0.491%로, 손절조건인 -0.4%가 넘었으므로 매도.
```text
2026-06-08 13:54:44 | INFO | samsung_auto_trader | Sell signal detected. disparity=99.542 return_rate=-0.00491 holding_seconds=57.3
2026-06-08 13:54:44 | INFO | samsung_auto_trader | Holdings before sell. quantity=10
2026-06-08 13:54:44 | INFO | samsung_auto_trader | Sell order request. symbol=005930 quantity=10
2026-06-08 13:55:03 | INFO | samsung_auto_trader | Account snapshot. symbol=005930 holding_qty=0 avg_price=0 account_value=499507148
2026-06-08 13:55:03 | INFO | samsung_auto_trader | Holdings after sell. quantity=0
2026-06-08 13:55:03 | INFO | samsung_auto_trader | Sell execution seems to have occurred.
```

7. 매도(시간청산)
   - 주식 보유 시간이 189.9초로, 시간청산 기준인 180초를 넘었으므로 매도.
```text
2026-06-08 13:51:03 | INFO | samsung_auto_trader | Sell signal detected. disparity=100.155 return_rate=0.00328 holding_seconds=189.9
2026-06-08 13:51:03 | INFO | samsung_auto_trader | Holdings before sell. quantity=10
2026-06-08 13:51:03 | INFO | samsung_auto_trader | Sell order request. symbol=005930 quantity=10
2026-06-08 13:51:21 | INFO | samsung_auto_trader | Account snapshot. symbol=005930 holding_qty=0 avg_price=0 account_value=499524109
2026-06-08 13:51:21 | INFO | samsung_auto_trader | Holdings after sell. quantity=0
2026-06-08 13:51:21 | INFO | samsung_auto_trader | Sell execution seems to have occurred.
```

## 5. 프로그램 실행 전후 잔고 (6월8일 11:57:04 ~ 15:00:56)
1. 자동매매 실행 전 잔고
<img width="346" height="75" alt="image" src="https://github.com/user-attachments/assets/cb898de1-85ec-437b-a778-96f5730d8255" />


2. 자동매매 실행 후 잔고
<img width="330" height="72" alt="image" src="https://github.com/user-attachments/assets/63c843dc-0995-45b2-ad23-0bb24748ea90" />

## 6. 마무리
<img width="308" height="150" alt="image" src="https://github.com/user-attachments/assets/25378131-1801-4a47-b2e4-e403a2537711" />

- 횡보장이었던 12:00~13:00 경에는 이격도 전략을 통해 + 수익을 냈었음.
- 13:00 이후 하락장이 시작되면서 -수익으로 전환되었음.
- 하락장일 때 이격도 조건 매도보다 손절조건 매도와 시간청산 매도가 더 많이 일어났음.
- 역추세 유형 전략의 장단점을 느낄 수 있었음.
