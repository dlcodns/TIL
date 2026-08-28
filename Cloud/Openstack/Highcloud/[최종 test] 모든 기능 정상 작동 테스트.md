# Highcloud 테스트 계획 및 시나리오

```
source ~/highcloud-admin-openrc.sh
```

## 계획

1. 플젝 생성 시
   1. 새 유저(리더)에게 초기 랜덤 비번 발송 [그 랜덤 비번으로 sdk main.py 수정]
   2. 유효성 검사 실패 시 실패 이유를 메일 발송 [메일 정규식 안 맞으면 걍 무시]
2. 유저 생성 시
   1. 새 유저(멤버)에게 초기 랜덤 비번 발송 [그 랜덤 비번으로 sdk main.py 수정]
   2. 유효성 검사 실패 시 실패 이유를 메일 발송 [메일 정규식 안 맞으면 걍 무시]
3. 유저 삭제 시
   1. 멤버 비번이 실제로 맞으면 삭제하고 삭제 완료 메일 발송 [현재 비번이 검증 구현 안되어 있음]
   2. 멤버 비번이 틀리면 틀리다고 메일 발송
4. 플젝 삭제 시
   1. 리더 정보가 아니면 리더만 삭제 가능 메일 발송
   2. 리더 비번이 틀리면 비번 다름 메일 발송
   3. 성공하면 성공했다 메일 발송

---

## Phase 1 - 프로젝트 생성: 검증 실패 분기 (생성 안 됨)

<details>
<summary><b>1-0 전체 결과 시트</b></summary>

![alt text](/assets/Openstack/test1.png)

</details>

<details>
<summary><b>1-1 개인정보 미동의</b></summary>

- 슬랙
![alt text](/assets/Openstack/test2.png)

- 이메일
![alt text](/assets/Openstack/test3.png)

> 결과 -

</details>

<details>
<summary><b>1-2 성함 한글 아님</b></summary>

- 슬랙
![alt text](/assets/Openstack/test4.png)

- 이메일
![alt text](/assets/Openstack/test5.png)



</details>

<details>
<summary><b>1-3 이메일 정규식 틀림</b></summary>

- 슬랙
![alt text](/assets/Openstack/test6.png)

- 이메일 발송 X


</details>

<details>
<summary><b>1-4 플젝명이 영문이 아님</b></summary>

- 슬랙
![alt text](/assets/Openstack/test7.png)

- 이메일
![alt text](/assets/Openstack/test8.png)


</details>

<details>
<summary><b>1-5 id가 영문과 숫자가 아님</b></summary>

- 슬랙
![alt text](/assets/Openstack/test9.png)

- 이메일
![alt text](/assets/Openstack/test10.png)



</details>

<details>
<summary><b>1-6 반납일이 내일 이후가 아님</b></summary>

- 슬랙
![alt text](/assets/Openstack/test11.png)

- 이메일
![alt text](/assets/Openstack/test12.png)

</details>

---

## Phase 2 - 프로젝트 생성: 대기 / 거절 / 승인 / 오류

<details>
<summary><b>2-1 신청 후 생성 대기 상태</b></summary>

- 슬랙
![alt text](/assets/Openstack/test13.png)

- 시트
![alt text](/assets/Openstack/test14.png)


</details>

<details>
<summary><b>2-2 [2-1]과 같은 내용으로 신청</b></summary>

- 슬랙
![alt text](/assets/Openstack/test15.png)

- 이메일
![alt text](/assets/Openstack/test16.png)

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>2-3 [2-1]과 같은 플젝명으로 다른 사람이 신청</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>[2-1]이 거절한 후 다시 다른 사람이 생성</b></summary>

- 슬랙

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>2-4 [2-1]을 슬랙에서 거절</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>2-5 [2-1]을 슬랙에서 승인</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>2-6 [2-1]의 유저 정보와 다르게 새로운 플젝 파기</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

---

## Phase 3 - 유저 추가: 검증 + 정상(신규/기존 가입)

<details>
<summary><b>3-1 개인정보 미동의</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>3-2 성함 한글 아님</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>3-3 이메일 정규식 틀림</b></summary>

- 슬랙
- 이메일 발송 X

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>3-4 id가 영문과 숫자가 아님</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>3-5 없는 프로젝트명으로 유저 추가</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>3-6 승인 대기중인 프로젝트로 유저 추가</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>3-7 유저 추가 성공</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>3-8 추가한 유저를 같은 플젝에 또 추가</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>3-9 가입된 유저를 다른 프로젝트에 유저 추가 - 비번 안내 X</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

---

## Phase 4 - 멤버 유저 삭제: 검증 + 정상

> 초기 상태

<details>
<summary><b>4-1 플젝에 포함되지 않은 유저를 삭제할 때</b></summary>

- 슬랙
- 이메일

> 결과 - 플젝에 존재하지 않는 id라고 떠야하는데 그렇지 않음

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>4-2 유저 개인 정보 불일치하게 삭제 신청할 때</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>4-3 [리더 방지] 리더를 삭제하려고 할 때</b></summary>

- 슬랙
- 이메일

> 리더는 플젝 삭제 때 같이 됨.

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>4-4 개인정보는 맞는데, 비번이 틀렸을 때</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>4-5 유저 삭제가 잘 됐을 때</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

---

## Phase 5 - 프로젝트 삭제: 검증 + 정상

<details>
<summary><b>5-1 멤버가 플젝을 삭제할 때 (안 되어야 함)</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>5-2 프로젝트명을 틀렸을 때</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>5-3 정말로 삭제하시겠습니까에 아니오 했을 때</b></summary>

- 슬랙
- 이메일 발송 X

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>5-4 리더의 정보가 불일치할 때</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>5-5 개인정보는 맞는데, 비번이 틀렸을 때</b></summary>

- 슬랙
- 이메일

> [명확한 test를 위해 skyline에서 새로운 비번으로 바꾸기!]

> 결과 -

<!-- 이미지 삽입 -->

</details>

<details>
<summary><b>5-6 플젝 삭제가 잘 됐을 때</b></summary>

- 슬랙
- 이메일

> 결과 -

<!-- 이미지 삽입 -->

</details>

**테스트 계정**

1. chaeun (Leader)
2. yuha

---

## Phase 6 - 자동 반납 검증

<details>
<summary><b>test-20260615가 20260616에 삭제될 예정임</b></summary>

> 결과 -

<!-- 이미지 삽입 -->

</details>