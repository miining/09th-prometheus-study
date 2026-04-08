# 목차
1. Label Offset Table & Label Index i
2. Posting Offset Table & Postings i
---

Index 파일

<img width="503" height="599" alt="image" src="https://github.com/user-attachments/assets/ebb80e42-8e13-4d3a-8bc9-22e5f41e57a1" />


- TOC
<img width="655" height="343" alt="image" src="https://github.com/user-attachments/assets/80716116-9a3f-40ad-a6e5-0da38af22ad1" />


## Label Offset Table & Label Index i
- 이 두 가지는 더 이상 사용되지 않지만 알아두면 좋다고 해서 한번 정리했습니다.
- 'Label Index i'는 인덱스 내에 있는 'Label Index 1'부터 'Label Index N'까지 중 어떤 하나의 Label Index를 가리킵니다.
- ex)
  - {a="b1", x="y1"}과 {a="b2", x="y2"}라는 두 개의 시리즈가 있다면,
  - 이 섹션은 레이블 이름 a에 대해 가능한 값이 [b1, b2]이고, x에 대해서는 [y1, y2]임을 식별할 수 있게 해줍니다.
  - 그런데 이러한 인덱싱은 프로메테우스에서 사용하지 않습니다.
- ex1)
[b1, b2]가 있다면 아래와 같이 표현됩니다.
<img width="419" height="73" alt="image" src="https://github.com/user-attachments/assets/f4326272-90f1-49f6-b123-fbde8127556b" />


- ex2)
[(b1, y1), (b2, y2)]는 아래와 같이 표현됩니다.
<img width="627" height="73" alt="image" src="https://github.com/user-attachments/assets/d6d7fa6b-4637-4cc0-b126-8a1adc7c6c2b" />


- 24 (len): 전체의 크기가 24바이트라는 뜻입니다.

- 2 (names): 묶여 있는 레이블 이름의 개수입니다. 여기서는 (a, x)처럼 2개의 레이블이 한 세트로 묶였다는 뜻입니다.

- 2 (entries): 가능한 값의 조합이 몇 세트인지 나타냅니다. 여기서는 (b1, y1)과 (b2, y2) 이렇게 2세트가 들어있다는 뜻입니다.

## Posting Offset Table & Postings i
- 'Postings i'는 포스팅들의 리스트를 저장하고, 'Postings Offset Table'은 오프셋을 통해 이 항목들을 참조합니다.
- 포스팅이란 '시리즈의 ID'를 나타냅니다.

<img width="438" height="284" alt="image" src="https://github.com/user-attachments/assets/7339cf22-ab1d-43b4-9e11-356d3e94bd69" />

- Posting i 입니다.
- ex)
  - 두 개의 시리즈를 예로 들어보면, 시리즈 ID 120인 {a="b", x="y1"} 과, 시리즈 ID 145인 {a="b", x="y2"}가 있습니다.
  - a="b"라는 레이블 쌍은 두 시리즈 모두에 존재하기에, [120, 145] 리스트를 저장해야 합니다.
  - 반면 x="y1"과 x="y2"는 각각 하나의 시리즈에만 나타납니다. 그러므로 각각 [120]과 [145] 리스트를 저장합니다.
  - 오직 시리즈에서 실제로 발견되는 레이블 쌍에 대해서만 리스트를 저장하기에, a="y1"이나 x="b" 같은 레이블 쌍은 어떤 시리즈에도 등장하지 않기 때문에 리스트를 만들어서 저장하지 않습니다.

<img width="468" height="351" alt="image" src="https://github.com/user-attachments/assets/33aee85b-8786-4501-ae3b-8fb3d2a4870d" />

- Posting Offset Table 입니다.
- Label Offset Table과 다른점은 Lable value가 추가된 것입니다.
- n은 항상 2를 나타내기에, (a="b", x="y1") 같은 [레이블 이름, 레이블 값] 한 쌍만 저장한다는 뜻입니다. 하지만 실제로는 이렇게 하지 않습니다.
  - 왜냐하면 조합의 경우의 수가 많아지고, 이미 정렬된 리스트를 가지고 있기 때문입니다
  - ex) method="GET"에 해당하는 ID 리스트: [1, 5, 10, 20] 

- 각 항목의 끝에는 해당되는 포스팅 리스트(Postings i)가 시작되는 파일 내 위치(Offset)가 적혀 있습니다.
  - ex) name="a", value="b" 항목은 [120, 145] 리스트가 있는 곳을 가리킵니다.
- 항목들은 레이블 이름과 값을 기준으로 정렬되어 있습니다. (레이블 찾을때 속도가 증가합니다.)
- name과 value에 symbol table을 쓰지 않고 raw string을 바로 적어서 I/O 속도를 증가 시킵니다.


