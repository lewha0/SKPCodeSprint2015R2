# SK planet Code Sprint 2015 Round 2

## 실행방법
* ../rsc/ 아래에 round2_itemInfo.tsv, round2_purchaseRecord.tsv 가 존재한다고 가정한다.
* 그런 상황에서 python solution.py 를 실행하면 predict.csv가 생성된다.

## 실행 환경 만들기
* Python 2.7.6 환경에서 작업하였다.
* <s>oth/requirements.txt를 참고하면 어느 패키지가 필요한지 확인할 수 있다.</s> 
* 특별한 패키지를 사용하진 않았으며, 아마도 `sklearn`과 `numpy` 정도면 충분하리라 생각한다.

## 기타
* oth/solution.ipynb 안에 작업시 사용하였던 ipython notebook 파일이 포함되어 있다.
* 해당 파일 내에 주석과 설명이 포함되어 있으므로 이를 참고할 수 있다.
* 아래는 풀이에 대한 간략한 설명입니다. 해당 설명은 ipynb 파일에도 포함되어 있습니다.

# SK planet Code Sprint 2015 Round 2 풀이

## 들어가기

* 이 내용의 작성자는 Jinhyun "lewha0" Kim이다(gmail 계정 lewha0).
* 아래 내용은 SK planet Code Sprint 2015 Round 2에 대한 풀이를 담고 있다.
* 설명된 내용과 실제 코드가 약간 다를 수 있다.
 * 이는 해당 내용을 구현할 때 구현상의 편의를 위하여 설명된 내용을 약간 변형한 것일 수도 있고, 역으로 이해를 돕기 위하여 구현된 내용을 단순화하여 설명하였을 수도 있다.
 * 또한 문서 작성 시점 이후에도 점수 개선을 위하여 코드를 약간 변형하였을 수 있다.
 * 혹은, 코드를 간결하게 바꾸는 과정에서 tie-breaking rule에 미묘한 차이가 생기면서 결과가 아주 약간 달라지는 경우도 있음을 발견하였다.
 * 그러나 큰 틀 안에서는 설명 내용은 계속 유효할 것이며, 큰 틀이 바뀌게 되면 설명 내용도 바뀔 것이다.
* 설명된 내용은 수 차례의 실험을 통해 개선하였던 방법에 대한 것이며, 따라서 '왜 이와 같이 하였는가?'란 물음에 대한 답은 대개는 '실험해보니 좋아서'가 될 것이다. 가급적이면 내용을 설명할 때 사고 과정이 드러날 수 있도록 작성하였다.
* **저는 python을 제대로 공부해 본 적이 없으며, 따라서 잘 쓸줄 모르고, 그렇기 때문에 코드가 못생겼을 것입니다. 이 점 너그러이 이해하고 봐 주실 것을 부탁드립니다. 이를테면, 저는 이번 기회를 통해서 zip이나 enumerate 같은 것들을 처음 사용해 보았는데, 아직도 이들이 어떠한 작업을 수행해 주는지는 알지 못합니다. 죄송합니다.**
* 채점 기준이 대회 중간에 수정되었으나, 이로 인하여 의욕이 많이 상실되었기 때문에 대대적인 수정을 가하지는 못하였고, 일부만 수정하여 최종적으로 제출하게 되었다.

## 기본 아이디어

* 컨텐츠 사이의 유사도 sim(cx, cy)를 계산한다. 유사도는 0에서 1 사이의 값을 갖도록 한다.
* 사용자 u가 cx라는 컨텐츠를 구매하였다면, 그 사용자의 cx라는 컨텐츠에 대한 점수 score(u, cx)를 sim(cx, cy) 만큼 증가시킨다. 이를 u가 구매한 모든 컨텐츠에 대해 적용한다.
* 사용자 u에 대해서 가장 점수가 높은 영화 컨텐츠, 즉 score(u, c)를 최대로 하는 c(단 c는 영화)를 선택한다. 그리고 그 사용자에 대해서 c를 **100개** 출력한다!

## 컨텐츠 사이의 유사도 계산

* 각각의 컨텐츠에 대한 정보를 하나의 문자열로 만든다.
 
 * 먼저 텍스트 형태로 되어있는 정보를 추려내어 하나의 문자열을 만든다. *제목, 시놉시스, 감독명 배열, 출연자명 배열, 세부 장르, 제작 국가*가 이에 해당한다.
  > 비엔나 호텔의 야간 배달부 1957년의 비엔나. 과거 칼펜부르노 (하략)
  
 * 문자열에 포함된 내용 중, 학습에 방해가 될 것 같은 부분 문자열을 공백으로 치환한다. 예를 들어 괄호, 문장 부호, 하이픈 등이 있으면 이를 하나의 공백으로 치환한다.
 
 * 텍스트 정보가 더 잘 드러날 수 있도록, 원본 문자열의 모든 bigram(2-gram)을 문자열에 덧붙인다.
  > 비엔나 호텔의 야간 배달부 1957년의 비엔나. 과거 칼펜부르노 (중략) 비엔 엔나 나호 호텔 텔의 (하략)

 * 원본 문자열의 모든 3-gram, 4-gram도 문자열에 덧붙인다.
  > 비엔나 호텔의 야간 배달부 1957년의 비엔나. 과거 칼펜부르노 (중략) 비엔 엔나 나호 호텔 텔의 (중략) 비엔나 엔나호 나호텔 (중략) 비엔나호 엔나호텔 나호텔의 (하략)
  
 * 이 문자열에 텍스트 형태가 아닌 정보를 이어붙인다. 이 때 각각의 정보가 어느 필드에 해당하는지를 표현할 수 있도록 정보 앞에 필드 번호를 붙인다. 이는 해당 정보가 텍스트 정보 안에 포함되어 있는 것과 구별하기 위함이다. 예를 들어, 영화가 1974년도에 나왔다면 시놉시스에도 1974가 등장할 수 있으므로 이 둘이 구별되도록 하기 위함이다. *필드 번호는 0부터 시작한다*.
 > 비엔나 호텔의 (중략) 2_FL 3_MOV 6_1974 7_19971108 13_18 14_14/07/11

 * 해당 컨텐츠를 구매한 사용자들의 목록도 이어붙인다. 만일 한 사용자가 여러 번 구매했다면 그만큼 여러 번 이어붙인다. 마찬가지로 해당 내용이 구매 사용자의 번호임을 알 수 있도록 하는 정보를 덧붙여 표현한다. *사용자 번호도 0부터 시작한다*.
 > 비엔나 호텔의 (중략) 2_FL 3_MOV 6_1974 7_19971108 13_18 14_14/07/11 (중략) user_0 user_0 user_1986 user_2222

* 문자열을 벡터로 바꾼다.

 * TF-IDF를 사용한다.
 
 * 텍스트 형태가 아닌 정보, 혹은 구매한 사용자 정보 등은 어떻게 보면 해당 컨텐츠를 표현하고 있는 문서의 일부라기보다는 독자적인 정보일 것이다. 따라서 이를 한꺼번에 뭉뚱그려 TF-IDF를 계산하는 것이 일견 비합리적으로 보일 수도 있으며, 해당 정보에 대해서는 빈도수를 세는 등의 방식이 좋아보이기도 한다. 그러나 실험해보니 그냥 이렇게 TF-IDF를 사용하는게 제일 괜찮았다.
 
 * 이제 각각의 컨텐츠가 하나의 벡터로 표현되었다.
 
* 컨텐츠 사이의 유사도를 계산한다.

 * 두 컨텐츠에 대한 벡터 두 개의 cosine similarity를 사용한다. TF-IDF 벡터는 sparse하게 표현될 것이므로 이를 빠르게 계산할 수 있기 때문이다.
 
 * 실제로 활용되는 것은 임의의 컨텐츠 cx와 영화 컨텐츠 cy 사이의 유사도이다. 따라서 유사도를 계산할 때에는 한 쪽은 모든 컨텐츠, 다른 한 쪽은 영화 컨텐츠로 하였다.
 
 * 자기 자신에 대한 유사도는 1로 표현될 것이다. **이를 따로 보정하지 않고 그대로 놔 둔다**. 여러 방법으로 보정을 해 보았는데 별로였다. **재구매 만세!**.
 
## 실험

* 주어진 데이터를 training data와 validation data로 나눠서 사용하였다.
 
* 데이터를 나눌 때에는 가장 단순한 방법을 사용했는데, 1월까지의 데이터로 학습을 하여 2월 데이터에 대해 평가를 수행하였다.
 
* 이와 같이 하게 되면 결과 수치가 조금 작게 나올 수 있는데, 이는 2월에 영화를 하나도 구매하지 않은 사용자가 있기 때문이다. 이러한 사용자는 애초에 없다고 생각하고 점수를 계산하였다. 즉, N을 줄여서 사용하였다.
 
* Validation set에 대한 점수와 실제 제출하여 받는 점수 사이에은 **매우 유의미한** 상관관계가 있었다.
 
* 최종적으로 제출할 때에는 주어진 전체 데이터를 사용한다.
 
## 결과 출력

* 과거 SK planet Code Sprint의 경험상, 점수 계산 방식만 잘 활용하여도 높은 추가점을 획들할 수 있는 경우가 몇 번 있었다. 이번에도 그렇지 않을까 하는 생각에 이런저런 방법을 시도해보게 되었고, 뒤늦게나마 한 방법을 찾았다.
 
* MAP을 이리저리 분석해 보니 해당 사용자가 구매하였을 확률이 가장 높은 영화 하나만 100개 출력하는게 좋아 보였다. 실제로도 그러했다.
 
* 이는 아직도 그 이유를 정확히 모르겠는데, 대략 두 가지 이유 때문인 것으로 판단된다.
 
 * 사용자는 실제로 그리 많은 영화를 구매하지 않는다. 91,223명의 사용자는 6개월간 5,978,953건의 컨텐츠를 구매하였다. 한 사람이 한 달에 평균 10번 컨텐츠를 구매하는 것이다. 영화는 더 적을 것이다. 이 때, 해당 사용자가 구매한 영화 중 하나만 제대로 맞힐 수 있다면, 그 사용자에 대해서 100/n 점을 얻을 수 있다. 실험을 해보니 이와 같이 영화 하나를 제대로 맞히는 것도 제법 수월한 편이었던 것으로 보이고, 각 사용자의 n이 작아서 점수가 뻥튀기되는 정도도 컸던 것으로 생각된다. 개인적인 견해로는, 이처럼 실제로 구매한 컨텐츠 수에 비해서 예측해야 하는 컨텐츠 수가 많은 경우라면 MAP이 부적절한 것 같다. 1만 번 영화를 구매하는 사람들에 대해서 100개를 예측하게 하는 것은 의미가 있겠으나, 이처럼 영화를 10번도 구매하지 않는 사용자라면 MAP이 다소 엉뚱한 결과를 내는 것은 아닌가 하는 생각이 든다.
   
 * 사용자는 생각보다 자주 컨텐츠를 재구매한다. 본인의 Btv 사용 경험에 따르면(주어진 데이터가 Btv와 연관이 있을지는 모르겠으나), 무료 영화가 많이 올라와 있으며 따라서 생각날 때마다 이를 구매하여 시청할 수 있다. 꼭 그런 이유가 아니더라도 컨텐츠를 재구매하는 사용자도 제법 많았고, 재구매율이 높은 컨텐츠도 꽤 많았다. 그 덕분에 각 사용자가 구매할 영화 중 하나를 맞히는 것이 생각보다 수월했던 것으로 생각된다.
 
## 최종 버전

* 최종적으로는 중복 예측이 불허되었기 때문에 이 버전을 수정하여 사용하였다.

* score가 제일 높은 100개의 영화를 출력한다.
