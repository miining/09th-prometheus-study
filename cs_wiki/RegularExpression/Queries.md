# Matcher의 4가지 타입
- Prometheus TSDB의 Select([]matcher) 쿼리에는 총 4가지 Matcher가 존재합니다.
<img width="741" height="299" alt="image" src="https://github.com/user-attachments/assets/b834e51b-8ec8-41b5-9e80-61c329202b1d" />

- 라벨 매칭 연산자는 아래와 같습니다.
- =는 정확히 일치, !=는 불일치, =~는 정규표현식 일치, !~는 정규표현식 불일치입니다.
- ex)
<img width="480" height="241" alt="image" src="https://github.com/user-attachments/assets/ff463dd8-a9b8-493e-b820-e6d5035d615c" />


## Full Anchoring
- 앵커링은 위치를 고정하는 기호를 말합니다.
- 프로메테우스의 모든 정규표현식은 RE2 문법을 사용합니다.
---

- ^ : 문자열의 시작을 의미합니다.
- $ : 문자열의 끝을 의미합니다.
- 일반적인 프로그래밍 언어(Python 등..)에서 정규식으로 foo를 검색하면, 글자 중간에 foo가 끼어있기만 해도 검색을 합니다.
- 하지만 프로메테우스는 사용자가 foo라고만 입력해도, 내부적으로 앞뒤에 anchor(^, $)을 달아버립니다. 즉, ^foo$로 바꿔서 검색합니다.
- 만약 앞뒤에 아무 글자나 와도 괜찮으면 .* 을 명시해주면 됩니다.
- ex) ~"foo.*" 이렇게하면 foobar같은 것을 찾을 수 있습니다.
  - 내부적으로는 ^foo.*$ 로 처리하게됩니다.
 
<img width="745" height="529" alt="image" src="https://github.com/user-attachments/assets/fc1cc9a4-2ccb-40b7-a146-451c66633f8c" />

- '.' : 아무 글자 1개만 뒤에 올 수 있습니다.
  - ex) app. -> app1, appA
-  : ex) app.* -> app, app1, appXUZ
- '+' : 1개는 존재해야합니다.
  - ex) app.+ -> app1, appXYZ
- '?' : 있어도 괜찮고 없어도 괜찮습니다.
  - ex) app? -> app, apps
- 대괄호 [] 안에서는 그 안의 글자 중 딱 1글자만 선택됩니다.
  - ex) [45]xx -> 4xx, 5xx
  - ex) [^abc] -> a,b,c 제외하고 아무거나 가능합니다.
- \d : [0-9]와 완전이 같은 뜻입니다.
  - ex) [45]\d\d -> 400번대, 500번대 에러 찾기
- \w : 알파벳 대소문자(a-z, A-Z), 숫자(0-9), 그리고 언더바(_) 중 딱 1개만을 나타냅니다.
- {n,m} : 앞의 글자가 n번에서 m번 반복됩니다.
  - ex) [a-z]{2,4} -> 소문자가 2~4글자 연속으로 나타납니다.


## 부정 Matcher의 내부 동작
- Not Equal(a!="b")과 Regex Not Equal(a!~"<rgx>")은 내부적으로 Equal/Regex Equal로 변환하여 처리합니다.
- 부정 Matcher만 단독으로 쿼리를 사용할 수 없고 반드시 Equal 또는 Regex Equal Matcher가 하나 이상 있어야 합니다.
- ex) 부정 matcher만 단독으로 사용하면 oo가 아닌 것을 다 가져와야해서 부하가 증가하게 됩니다.
   - 그렇기에 앞에 긍정 조건을 제시하고 부정을 표시해줍니다.
<img width="497" height="145" alt="image" src="https://github.com/user-attachments/assets/ec5fcbfd-cc7c-4156-9eb4-24f650bdd867" />

- job=~"bar.*" 를 먼저 실행합니다. s3인( bar1), s4인 (bar2)가 검색됩니다. -> 결과 집합: {s3, s4}
- 여집합을 활용합니다. status=~"5.."(500번대인 것)를 찾습니다.
  - s2 (501), s4 (501)이 검색됩니다. -> 뺄 집합: {s2, s4}
- {s3, s4} - {s2, s4} -> {s3}

## 빈 라벨값과 매칭
- 어떤 시리즈에 특정 라벨이 존재하지 않는다면, 프로메테우스는 내부적으로 그 라벨의 값이 빈 문자열("")이라고 간주합니다.
- ex)
- 원래 데이터가 http_requests_total{replica="rep-a"}, http_requests_total{environment="development"} 있습니다.
- 사용자가 http_requests_total{environment=""} 해당 쿼리를 날리면 프로메테우스는 1번 데이터는 가져오고 2번 데이터는 가져오지 않습니다.
- 내부적으로 1번 데이터는 http_requests_total{replica="rep-a", environment=""} 2번데이터는 http_requests_total{replica="", environment="development"}로 나타내고 있기 때문입니다.



 
