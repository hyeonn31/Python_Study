# Pandas 데이터 입출력 정리

> 한 줄 요약
> 파일을 표로 읽어 올 때는 `pd.read_xxx()`, 표를 파일로 저장할 때는 `df.to_xxx()`를 쓴다.
> 읽어 온 표가 어떻게 생겼는지는 `head`, `info`, `describe` 같은 도구로 살펴본다.

---

## 0. 먼저 알아야 할 단어들

이 단어들을 모르면 아래 내용이 하나도 안 읽힌다. 오랜만에 보면 여기부터 다시 읽자.

### 데이터프레임 (DataFrame)

pandas가 다루는 **표**다. 엑셀 시트 한 장이라고 생각하면 된다.
보통 변수 이름을 `df`(DataFrame의 줄임)로 짓는다.

```
       name  age  city        ← 열 이름 (columns)
0     Jerry   20  Seoul
1      Riah   22  Busan       ← 한 줄 = 행 (row)
2      Paul   21  Daegu
↑
행 이름 (index)
```

### 행, 열

- **행(row)**: 가로 한 줄. 사람 한 명, 자동차 한 대처럼 데이터 하나.
- **열(column)**: 세로 한 줄. 이름, 나이처럼 같은 종류의 정보 모음.

### 인덱스 (index)

**행의 이름표**다. 엑셀 왼쪽에 붙은 1, 2, 3 줄 번호 같은 것.
따로 정해 주지 않으면 pandas가 `0, 1, 2 ...`를 자동으로 붙인다.
원하면 특정 열(예: `name`)을 인덱스로 쓸 수 있다. 그러면 `0, 1, 2` 대신 `Jerry, Riah, Paul`이 행 이름이 된다.

### 시리즈 (Series)

데이터프레임에서 **열 하나만 꺼낸 것**이다. `df['age']`처럼 꺼내면 시리즈가 된다.
표가 데이터프레임이라면, 시리즈는 그 표의 세로 한 줄이다.

### 자료형 (dtype)

각 열에 어떤 종류의 값이 들어 있는지를 말한다.

| 자료형 | 뜻 | 예 |
|---|---|---|
| `int64` | 정수 | `8`, `1970` |
| `float64` | 소수 | `18.0`, `3.5` |
| `str` (또는 `object`) | 글자 | `'Jerry'`, `'?'` |

숫자 열이어야 더하기, 평균 같은 계산을 할 수 있다. 글자 열이면 계산이 안 된다.

### NaN

**비어 있는 칸**을 뜻한다 (Not a Number). 값이 없다는 표시일 뿐, 에러가 아니다.

---

## 1. 큰 그림

데이터는 CSV, 엑셀, JSON, 웹페이지 표처럼 여러 모양의 "포장지"에 담겨 온다.
pandas는 어떤 포장지든 뜯어서 **똑같은 모양의 표(데이터프레임)**로 바꿔 준다.
반대로 표를 원하는 포장지에 다시 싸서 파일로 저장할 수도 있다.

| 파일 종류 | 읽기 (파일 → 표) | 저장 (표 → 파일) |
|---|---|---|
| CSV | `pd.read_csv()` | `df.to_csv()` |
| Excel | `pd.read_excel()` | `df.to_excel()` |
| JSON | `pd.read_json()` | `df.to_json()` |
| HTML (웹페이지 표) | `pd.read_html()` | `df.to_html()` |

💡 **외우는 법**

- 읽기는 `pd.`로 시작한다. 아직 표가 없으니까 pandas에게 "이 파일 좀 읽어 줘"라고 부탁하는 것.
- 저장은 `df.`로 시작한다. 이미 표가 있으니까 표에게 "너 자신을 파일로 저장해"라고 시키는 것.

---

## 2. 파일 경로 읽는 법

파일을 읽으려면 파일이 어디 있는지 알려 줘야 한다.

```python
'./Data-Pandas/read_csv_sample.csv'
```

| 부분 | 뜻 |
|---|---|
| `.` | 지금 노트북 파일이 있는 폴더 |
| `/Data-Pandas` | 그 안의 Data-Pandas 폴더 |
| `/read_csv_sample.csv` | 그 안의 파일 |

`.`은 "여기"라는 뜻이다. 그래서 노트북을 다른 폴더로 옮기면 "여기"도 바뀐다.
`FileNotFoundError`(파일을 못 찾음)가 나면 경로부터 확인하자.

---

## 3. CSV 파일 읽기

### CSV란?

Comma-Separated Values, **쉼표(,)로 칸을 나눈 글자 파일**이다.
메모장으로 열면 이렇게 생겼다.

```
c0,c1,c2,c3      ← 첫 줄 : 열 이름
0,1,4,7
1,2,5,8
2,3,6,9
```

엑셀 없이도 열 수 있고 단순해서 데이터를 주고받을 때 가장 많이 쓴다.

### 3-1. 기본으로 읽기

```python
import pandas as pd

file_path = './Data-Pandas/read_csv_sample.csv'
df1 = pd.read_csv(file_path)
print(df1)
```

```
   c0  c1  c2  c3
0   0   1   4   7
1   1   2   5   8
2   2   3   6   9
```

- 파일의 첫 줄(`c0,c1,c2,c3`)이 **열 이름**이 됐다.
- 행 이름은 안 정해 줬으니 `0, 1, 2`가 자동으로 붙었다.

옵션을 안 써도 첫 줄을 열 이름으로 쓰는 이유는 `header=0`이 기본값이기 때문이다.
`header=0`은 "0번째 줄(= 첫 줄)이 열 이름이야"라는 뜻이다. (파이썬은 0부터 센다.)

### 3-2. `header=None` : "이 파일엔 열 이름 줄이 없어"

```python
df2 = pd.read_csv(file_path, header=None)
```

```
    0   1   2   3
0  c0  c1  c2  c3     ← 원래 열 이름이 데이터 취급됨
1   0   1   4   7
2   1   2   5   8
3   2   3   6   9
```

- 첫 줄도 그냥 데이터로 읽는다.
- 열 이름이 없으니 pandas가 `0, 1, 2, 3`을 자동으로 붙인다.

**언제 쓰나?** 파일이 첫 줄부터 바로 데이터로 시작할 때. (6장의 자동차 데이터가 그런 파일이다.)

⚠️ 열 이름이 있는 파일에 `header=None`을 쓰면 `c0` 같은 글자가 숫자들과 같은 열에 섞인다.
열 안에 글자가 하나라도 있으면 pandas는 그 열 전체를 글자로 취급해서 계산이 안 된다.

### 3-3. `index_col` : "이 열을 행 이름으로 써"

```python
df4 = pd.read_csv(file_path, index_col='c0')
```

```
    c1  c2  c3
c0
0    1   4   7
1    2   5   8
2    3   6   9
```

- `c0` 열이 왼쪽으로 빠져서 **행 이름표**가 됐다.
- 그래서 열 목록에는 이제 `c1, c2, c3`만 남는다.

**왜 쓰나?** 학생 이름, 날짜처럼 각 행을 구분하는 열이 있을 때, 그걸 행 이름으로 쓰면 나중에 "Jerry의 점수"처럼 이름으로 바로 찾을 수 있다.

`index_col=None`이 기본값이고, 이때는 3-1처럼 `0, 1, 2`가 붙는다.

### 정리: header와 index_col

| 옵션 | 정하는 것 | 기본값 |
|---|---|---|
| `header` | 몇 번째 줄이 **열 이름**인지 | `0` (첫 줄) |
| `index_col` | 어떤 열이 **행 이름**인지 | `None` (0, 1, 2 자동) |

---

## 4. Excel 파일 읽기

```python
file_path = './Data-Pandas/남북한발전전력량.xlsx'
df1 = pd.read_excel(file_path, engine='openpyxl')
print(df1)
```

- 사용법은 CSV와 같다. `header`, `index_col` 옵션도 똑같이 쓴다.
- `engine='openpyxl'`은 "엑셀 파일을 열 때 openpyxl이라는 도구를 써"라는 뜻이다.
  pandas는 엑셀을 직접 못 읽어서 이런 도우미 도구가 필요하다. 없으면 `pip install openpyxl`로 설치한다.
  요즘 pandas는 `.xlsx`면 알아서 openpyxl을 쓰니까 생략해도 된다.
- 엑셀 파일에 시트가 여러 장이면 `sheet_name='시트이름'`으로 고른다. 안 쓰면 첫 번째 시트를 읽는다.

### 결과에 NaN이 보이는 이유

```
  전력량 (억㎾h) 발전 전력별  1990  1991 ...
0        남한     합계  1077  1186
1       NaN     수력    64    51
2       NaN     화력   484   573
```

엑셀에서 '남한' 칸은 여러 줄을 하나로 합친 **병합 셀**이었다.
pandas는 병합 셀을 읽을 때 첫 칸에만 값을 넣고 나머지는 빈칸(NaN)으로 둔다.
에러가 아니라, 표를 사람이 보기 좋게 꾸민 흔적이다.

---

## 5. JSON 파일 읽기

### JSON이란?

파이썬 딕셔너리처럼 `{ "이름": 값 }` 모양으로 데이터를 적은 글자 파일이다.
웹에서 서버와 데이터를 주고받을 때 많이 쓴다.

```json
{
    "year":      {"pandas": 2008, "NumPy": 2006, "matplotlib": 2003},
    "developer": {"pandas": "Wes Mckinneye", ...},
    "opensource":{"pandas": "True", ...}
}
```

### 읽기

```python
df = pd.read_json('./Data-Pandas/read_json_sample.json')
print(df)
```

```
            name  year        developer opensource
pandas            2008    Wes Mckinneye       True
NumPy             2006  Travis Oliphant       True
matplotlib        2003   John D. Hunter       True
```

### JSON이 표로 바뀌는 규칙

JSON은 딕셔너리 안에 딕셔너리가 들어 있는 2층 구조다. pandas는 이렇게 바꾼다.

- **바깥 키** (`year`, `developer` ...) → **열 이름**
- **안쪽 키** (`pandas`, `NumPy` ...) → **행 이름**

즉 `"year": {"pandas": 2008}`은 "year 열, pandas 행의 값은 2008"이라는 뜻이다.

💡 `name` 열이 비어 보이는 건 파일 안에 빈 글자(`""`)가 들어 있어서다.
`opensource`의 `True`도 따옴표 안에 든 글자 `"True"`라서, 진짜 참/거짓 값이 아니라 그냥 네 글자짜리 문자다.

---

## 6. HTML(웹페이지) 표 읽기

웹페이지 안의 표는 `<table>`이라는 태그로 만들어져 있다.
`pd.read_html()`은 페이지 안의 `<table>`을 **전부** 찾아서 표로 바꿔 준다.

```python
tables = pd.read_html('./Data-Pandas/sample.html')

print(len(tables))       # 2  → 표가 2개 있었다
print(type(tables))      # <class 'list'>
print(type(tables[0]))   # <class 'pandas.DataFrame'>
```

### 핵심: 결과는 표가 아니라 "표들의 리스트"

한 페이지에 표가 여러 개 있을 수 있으니, pandas는 찾은 표를 모두 리스트에 담아서 준다.

```
tables = [ 표0, 표1 ]
           ↑     ↑
     tables[0] tables[1]
```

표가 하나뿐이어도 리스트로 준다. 그래서 꼭 `tables[0]`처럼 번호로 꺼내서 써야 한다.

### 모든 표 출력하기

```python
for i in range(len(tables)):    # len(tables) = 2 → i = 0, 1
    print(f"tables[{i}]")
    print(tables[i])
```

### 원하는 표 꺼내서 행 이름 정하기

```python
df = tables[1]                          # 두 번째 표 꺼내기
df.set_index(['name'], inplace=True)    # name 열을 행 이름으로
print(df)
```

```
            year        developer  opensource
name
NumPy       2006  Travis Oliphant        True
matplotlib  2003   John D. Hunter        True
pandas      2008    Wes Mckinneye        True
```

- `set_index('열')`: 이미 만들어진 표에서 그 열을 행 이름으로 바꾼다. (3-3의 `index_col`은 읽으면서 바로 하는 것, `set_index`는 읽은 다음에 하는 것)
- `inplace=True`: "원래 표를 직접 바꿔"라는 뜻. 이걸 안 쓰면 바뀐 새 표를 돌려줄 뿐 `df`는 그대로다.

💡 `Unnamed: 0`이라는 열이 보일 때가 있는데, HTML 표의 첫 칸이 비어 있어서 pandas가 "이름 없음 0번"이라고 붙인 것이다.

⚠️ `read_html`은 lxml이라는 도구가 필요하다. 없으면 `pip install lxml`.

---

## 7. 파일로 저장하기

### 7-1. 저장할 표 만들기

```python
data = {'name' : ['Jerry', 'Riah', 'Paul'],
        'algol': ['A', 'A+', 'B'],
        'basic': ['C', 'B', 'B+'],
        'c++'  : ['B+', 'C', 'C+']}

df = pd.DataFrame(data)
df.set_index('name', inplace=True)
```

```
      algol basic c++
name
Jerry     A     C  B+
Riah     A+     B   C
Paul      B    B+  C+
```

딕셔너리의 **키가 열 이름**, **리스트가 그 열의 값들**이 된다.

### 7-2. CSV, JSON, Excel로 저장

```python
df.to_csv("./output/df_sample.csv")
df.to_json("./output/df_sample.json")
df.to_excel("./output/df_sample.xlsx")
```

알아 둘 것:

- **저장할 폴더가 미리 있어야 한다.** `output` 폴더가 없으면 에러가 난다. pandas는 폴더를 만들어 주지 않는다.
- **행 이름(인덱스)도 같이 저장된다.** 빼고 싶으면 `to_csv(..., index=False)`.
- **한글이 깨질 때:** 한글이 든 CSV를 엑셀로 열었는데 글자가 깨지면 `to_csv(..., encoding='utf-8-sig')`로 저장한다.

### 7-3. 확장자는 이름표일 뿐이다

```python
df.to_json("./output/df_sample.xlsx")   # 이름은 .xlsx 지만...
```

이렇게 하면 이름만 `.xlsx`이고 **내용은 JSON 글자**인 가짜 엑셀 파일이 생긴다. 엑셀로 열면 "파일 형식이 잘못됐다"고 나온다.
파일 내용을 정하는 건 확장자가 아니라 **`to_` 뒤의 함수 이름**이다. 엑셀로 저장하려면 반드시 `to_excel`을 써야 한다.

### 7-4. 읽기와 저장은 서로 거꾸로

위에서 저장한 JSON 파일을 열어 보면 이렇게 생겼다.

```json
{"algol":{"Jerry":"A","Riah":"A+","Paul":"B"}, "basic":{...}, "c++":{...}}
```

5장에서 JSON을 읽을 때와 같은 규칙이다. 바깥 키 = 열 이름, 안쪽 키 = 행 이름.
저장한 파일을 `pd.read_json()`으로 다시 읽으면 원래 표로 돌아온다.

### 7-5. 엑셀 파일 하나에 시트 여러 장 저장하기: ExcelWriter

`to_excel("파일명")`을 두 번 하면 두 번째가 첫 번째를 **덮어써서** 시트가 한 장만 남는다.
표 여러 개를 한 파일의 서로 다른 시트에 넣으려면 `ExcelWriter`를 쓴다.

```python
with pd.ExcelWriter("./output/df_excelwriter.xlsx") as writer:
    df1.to_excel(writer, sheet_name="sheet1")   # sheet1 에 df1
    df2.to_excel(writer, sheet_name="sheet2")   # sheet2 에 df2
```

**공책에 비유하면:**

1. `pd.ExcelWriter(...)` : 공책(엑셀 파일)을 편다
2. `to_excel(writer, sheet_name=...)` : 1쪽, 2쪽에 각각 표를 쓴다
3. `with` 블록이 끝나면 : 공책을 덮는다 (저장 완료)

`with`를 안 쓰고 이렇게 할 수도 있다.

```python
writer = pd.ExcelWriter("./output/df_excelwriter.xlsx")
df1.to_excel(writer, sheet_name="sheet1")
df2.to_excel(writer, sheet_name="sheet2")
writer.close()   # 공책 덮기. 이걸 빼먹으면 파일이 저장되지 않는다!
```

`close()`를 깜빡하기 쉬우니까, 알아서 닫아 주는 `with` 방식이 더 안전하다.

---

## 8. 읽어 온 데이터 살펴보기

처음 보는 데이터를 받으면 바로 분석하지 말고, 먼저 **어떻게 생겼는지** 훑어봐야 한다.
자동차 398대의 연비, 무게 등이 담긴 `auto-mpg.csv`로 연습했다.

### 8-1. 열 이름 붙여서 읽기

이 파일은 첫 줄부터 바로 데이터(`18.0,8,307.0,130.0,...`)라서 열 이름 줄이 없다.
그래서 `header=None`으로 읽고, 열 이름을 직접 붙여 준다.

```python
df = pd.read_csv('./Data-Pandas/auto-mpg.csv', header=None)
df.columns = ['mpg', 'cylinders', 'displacement', 'horsepower', 'weight',
              'acceleration', 'model year', 'origin', 'name']
```

| 열 | 뜻 |
|---|---|
| mpg | 연비 (1갤런으로 몇 마일 가는지, 클수록 좋음) |
| cylinders | 실린더(엔진 통) 개수 |
| displacement | 배기량 |
| horsepower | 마력 |
| weight | 무게 |
| acceleration | 가속 성능 |
| model year | 출시 연도 (70 = 1970년) |
| origin | 생산 지역 (1 = 미국, 2 = 유럽, 3 = 일본) |
| name | 자동차 이름 |

### 8-2. 살펴보기 도구 한눈에

| 코드 | 알려 주는 것 | 책에 비유하면 |
|---|---|---|
| `df.head()` | 앞 5행 | 첫 페이지 훑기 |
| `df.tail()` | 뒤 5행 | 마지막 페이지 훑기 |
| `df.shape` | (행 수, 열 수) | 책 두께 |
| `df.dtypes` | 열마다 자료형 | 각 칸에 숫자가 들었나 글자가 들었나 |
| `df.info()` | 위 정보 + 빈칸 개수 | 목차 |
| `df.describe()` | 숫자 열의 통계 요약 | 성적표 요약 |
| `df.count()` | 열마다 빈칸 아닌 값 개수 | 출석 인원 |
| `df['열'].value_counts()` | 값마다 몇 번 나왔는지 | 투표 개표 |

⚠️ **괄호 규칙**: `shape`, `dtypes`는 괄호가 없다. 이미 갖고 있는 **정보**를 보여 주기만 하니까.
`head()`, `info()`처럼 괄호가 있는 건 뭔가를 계산하거나 출력하는 **동작**이다.

### 8-3. `shape` : 표의 크기

```python
print(df.shape)   # (398, 9) → 398행, 9열
```

### 8-4. `dtypes` : 열마다 자료형

```
mpg             float64    ← 소수
cylinders         int64    ← 정수
horsepower          str    ← ❗ 마력인데 글자?
name                str    ← 자동차 이름이니 글자가 맞음
```

마력은 숫자여야 하는데 글자(`str`)로 나왔다.
파일을 뒤져 보면 마력 칸 몇 군데(6곳)에 숫자 대신 `?`가 들어 있다.
**열 안에 글자가 하나라도 섞이면 pandas는 그 열 전체를 글자로 읽는다.**

그래서 생기는 문제:

- `describe()`가 숫자 열만 계산하기 때문에 horsepower가 결과에서 빠진다.
- `df['horsepower'] + 1` 같은 계산이 안 된다.

이건 다음 단원 "데이터 전처리"에서 해결한다. 미리 보면, `pd.read_csv(..., na_values='?')`로 읽으면 `?`가 처음부터 빈칸(NaN)으로 바뀐다.

💡 **숫자여야 할 열이 `str`로 나오면, 이상한 글자가 섞여 있다는 신호다.**

### 8-5. `info()` : 표의 목차

```python
df.info()
```

```
RangeIndex: 398 entries, 0 to 397     ← 행 398개, 행 이름은 0~397
 #   Column        Non-Null Count  Dtype
 0   mpg           398 non-null    float64
 3   horsepower    398 non-null    str
...
```

- `Non-Null Count`: 빈칸이 아닌 값의 개수. 398이면 빈칸이 하나도 없다는 뜻.
- **함정**: horsepower에는 `?`가 있는데도 398로 나온다. `?`는 빈칸이 아니라 "물음표라는 글자"이기 때문이다. 빈칸이 없다고 데이터가 깨끗한 건 아니다.

💡 `print(df.info())`라고 쓰면 끝에 `None`이 한 줄 더 찍힌다.
`info()`는 스스로 화면에 출력하고, 돌려주는 값은 없기(`None`) 때문이다. 그냥 `df.info()`만 쓰면 된다.

### 8-6. `describe()` : 숫자 열의 통계 요약

```
              mpg   cylinders  ...
count  398.000000  398.000000      ← 값 개수
mean    23.514573    5.454774      ← 평균
std      7.815984    1.701004      ← 표준편차
min      9.000000    3.000000      ← 최솟값
25%     17.500000    4.000000      ← 1/4 지점
50%     23.000000    4.000000      ← 가운데 (중앙값)
75%     29.000000    8.000000      ← 3/4 지점
max     46.600000    8.000000      ← 최댓값
```

어려워 보이는 항목만 풀어 보면:

- **std (표준편차)**: 값들이 평균에서 얼마나 퍼져 있는지. 작으면 다들 비슷하고, 크면 들쭉날쭉하다.
- **25%, 50%, 75%**: 값을 작은 순서로 줄 세웠을 때 1/4, 절반, 3/4 지점에 있는 값.
  50%는 딱 가운데 값이라 **중앙값**이라고 부른다.

**읽는 예**: "자동차 연비는 평균 23.5. 가장 나쁜 차는 9, 가장 좋은 차는 46.6. 절반의 차는 연비가 23 이하다."

### 8-7. `describe(include='all')` : 글자 열도 포함

```python
print(df.describe(include='all'))
```

글자 열은 평균을 낼 수 없으니 대신 이런 정보가 나온다.

| 항목 | 뜻 | horsepower 결과 |
|---|---|---|
| unique | 서로 다른 값이 몇 종류인지 | 94종류 |
| top | 가장 많이 나온 값 | `'150.0'` |
| freq | top이 나온 횟수 | 22번 |

숫자 열의 unique/top/freq 칸과 글자 열의 mean/std 칸은 NaN으로 나온다. "해당 없음"이라는 뜻이다.

### 8-8. `count()` : 열마다 값 개수

```python
print(df.count())         # 모든 열 398
print(type(df.count()))   # <class 'pandas.Series'>
```

결과는 시리즈다. 행 이름이 열 이름이고, 값이 개수다.

### 8-9. `value_counts()` : 값마다 개수 세기

```python
origin_counts = df['origin'].value_counts()
print(origin_counts)
```

```
origin
1    249     ← 미국 차 249대
3     79     ← 일본 차 79대
2     70     ← 유럽 차 70대
```

- 어떤 값이 몇 번 나왔는지 센다. 투표 개표와 같다.
- 많은 순서로 자동 정렬된다.
- 개수 말고 어떤 값들이 있는지만 보고 싶으면 `df['origin'].unique()` → `[1, 3, 2]`

---

## 9. 만날 수 있는 에러

| 에러 | 원인 | 해결 |
|---|---|---|
| `FileNotFoundError` | 경로 오타, 또는 노트북 위치가 달라서 `./`가 다른 곳을 가리킴 | 경로 확인 |
| `ImportError: ... openpyxl` | 엑셀 도구 미설치 | `pip install openpyxl` |
| `ImportError: lxml not found` | HTML 도구 미설치 | `pip install lxml` |
| `OSError: Cannot save file into a non-existent directory` | 저장할 폴더가 없음 | 폴더 먼저 만들기 |
| `NameError: name 'pd' is not defined` | `import pandas as pd` 셀을 실행 안 함 | 위 셀부터 실행 |
| `NameError: name 'df1' ...` | df1을 만드는 셀을 실행 안 함 | 위 셀부터 실행 |
| `PermissionError` (엑셀 저장 시) | 그 엑셀 파일이 엑셀 프로그램에 열려 있음 | 엑셀 창 닫고 다시 실행 |
| 결과가 `version https://git-lfs...` 3줄 | 진짜 데이터가 아니라 "파일이 다른 곳에 있다"는 메모만 받은 것 (Git LFS) | 원본 파일 다시 받기 |

---

## 10. 한눈에 요약

- **읽기** = `pd.read_csv / read_excel / read_json / read_html`
  **저장** = `df.to_csv / to_excel / to_json`
- `header` = 몇 번째 줄이 열 이름인지 (기본 `0`, 없으면 `None`)
- `index_col` = 어떤 열을 행 이름으로 쓸지 (읽은 뒤에 하려면 `set_index`)
- **JSON**: 바깥 키 → 열, 안쪽 키 → 행
- **read_html**은 표들의 리스트를 준다 → `tables[0]`으로 꺼내기
- 파일 내용은 함수가 정한다. 확장자는 이름표일 뿐이다.
- 시트 여러 장 = `ExcelWriter` + `sheet_name`, `with`로 감싸면 알아서 저장
- 처음 보는 데이터는 `head → shape → info → describe → value_counts` 순서로 훑어보기
- 숫자 열이 `str`로 나오면 이상한 글자(`?` 등)가 섞여 있다는 신호
