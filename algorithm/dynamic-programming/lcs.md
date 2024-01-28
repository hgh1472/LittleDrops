---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
---

# LCS

LCS(Longest Common Subsequence)란 최장 공통 부분 문자열이다.

여기서 Subsequence는 부분 수열으로 **문자열이 연속적일 필요는 없다**는 것이다.

즉, 두 문자열을 비교할 때 공통 부분 수열 중 길이가 가장 긴 부분 수열을 의미한다.

예시로 "ACAYKP", "CAPCAK"를 보면 두 문자의 LCS는 `ACAK`다.

이 답을 어떻게 구할 수 있을까?

해결은 DP를 이용해 해결한다. 먼저 DP를 이용한 테이블을 먼저 봐보자.

<table data-full-width="true"><thead><tr><th width="73">DP</th><th width="68"></th><th width="45">C</th><th width="48">A</th><th width="49">P</th><th width="52">C</th><th width="46">A</th><th>K</th></tr></thead><tbody><tr><td></td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr><tr><td>A</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr><tr><td>C</td><td>0</td><td>1</td><td>1</td><td>1</td><td>2</td><td>2</td><td>2</td></tr><tr><td>A</td><td>0</td><td>1</td><td>2</td><td>2</td><td>2</td><td>3</td><td>3</td></tr><tr><td>Y</td><td>0</td><td>1</td><td>2</td><td>2</td><td>2</td><td>3</td><td>3</td></tr><tr><td>K</td><td>0</td><td>1</td><td>2</td><td>2</td><td>2</td><td>3</td><td>4</td></tr><tr><td>P</td><td>0</td><td>1</td><td>2</td><td>3</td><td>3</td><td>3</td><td>4</td></tr></tbody></table>

여기서 DP\[3]\[5]는  ACAYKP의 3번째 문자 `A`, CAPCAK의 5번째 문자 `A`까지 비교한다.  DP\[3]을 예시로 들어보자.

DP\[3]\[0]부터 DP\[3]\[6]까지 가면서 비교를 하게된다. 만약 비교하게 되는 문자가 같지 않다면, 각 문자를 고려하기 전의 최댓값으로 반영한다.&#x20;

`DP[i][j] = max(DP[i-1][j], DP[i][j-1]`

왜냐하면  문자가 같지 않다면,  현재 있는 부분 수열에서 가장 큰 값을 그대로 받기 때문이다.

여기서 각 문자라는 것은 두 문자열의 비교되는 문자이다. DP\[3]\[5]라면 `ACAYKP`의 `A`와 `CAPCAK`의 `A`를 말한다.&#x20;

DP\[3]\[5]처럼 비교되는 문자가 같다면,  `DP[i][j] = DP[i-1][j-1] + 1` 이다. 두 문자가 같기 때문에 두 문자를 비교하기 전 최댓값에 +1을 해주는 것이다.
