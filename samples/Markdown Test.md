---
title: Markdown Test
created: 2025-11-18 08:11:49+09:00
updated: 2025-11-18 08:11:49+09:00
tags:
  - Markdown
status: in progress
alias:
  - Markdown Test
---

## 📝 Markdown 종합 테스트 파일 (Sample.md)

이 파일은 앞서 제공된 **기본** 및 **고급/확장** Markdown 문법 요소를 모두 포함하고 있습니다. `.md` 파일로 저장하여 테스트에 사용해 보세요.

---

### **1. 텍스트 및 제목 (Text & Headings)**

가장 큰 제목입니다.

# 제목 1

두 번째로 큰 제목입니다.

## 제목 2

세 번째로 큰 제목입니다.

### 제목 3

---

### **2. 텍스트 스타일 (Text Styles)**

* **굵게:** **굵은 텍스트**
* *기울임:* *기울어진 텍스트*
* ***굵고 기울임:*** ***굵고 기울어진 텍스트***
* ~~취소선:~~ ~~취소선 텍스트~~

---

### **3. 목록 (Lists)**

#### **순서 없는 목록 (Unordered List)**

* 항목 A
* 항목 B
    * 들여쓰기 항목 B-1
    * 들여쓰기 항목 B-2
* 항목 C

#### **순서 있는 목록 (Ordered List)**

1.  첫 번째 단계
2.  두 번째 단계
3.  세 번째 단계

---

### **4. 링크 및 미디어 (Links & Media)**

* **링크:** [Google 검색 엔진](https://www.google.com)
* **이미지:** ![Markdown 로고 예시](https://fastly.picsum.photos/id/665/200/300.jpg?hmac=N-XMC7MU7lBNaQh5wZrn0uyEcJkpIGQP2s1vL_A8kF8)


---

### **5. 인용문 및 코드 (Blockquotes & Code)**

#### **인용문 (Blockquotes)**

> 이것은 인용문입니다.
>
> > 중첩된 인용문도 가능합니다.

#### **인라인 코드 (Inline Code)**

문장 내에서 `let x = 10;`처럼 짧은 코드를 표시합니다.

#### **코드 블록 (Code Block)**

```javascript
// JavaScript 코드 블록 예시
function calculateSum(a, b) {
    return a + b;
}

console.log(calculateSum(5, 7));
```

# 1. GFM 확장 문법 테스트 (GFM Extensions)

## 1.1 테이블 (Tables)
| 헤더 1 | 헤더 2 (중앙 정렬) | 헤더 3 (우측 정렬) |
| :--- | :---: | ---: |
| 행 1 셀 1 | 행 1 셀 2 | 행 1 셀 3 |
| 행 2 셀 1 | 행 2 셀 2 | 행 2 셀 3 |

## 1.2 태스크 리스트 (Task Lists)
* [x] 완료된 할 일
* [ ] 미완료된 할 일
* [ ] 긴 설명이 필요한 할 일

## 1.3 취소선 (Strikethrough)
~~이 텍스트는 취소되어야 합니다.~~

## 1.4 자동 URL 링크 (Autolinks)
이것은 자동으로 링크가 됩니다: https://github.com/

# 2. 표준 마크다운 문법 테스트 (Standard Markdown)

## 2.1 제목 (Headings)
### 2.1.1 Sub-Heading (H3)
#### 2.1.2 Deeper Heading (H4)

## 2.2 텍스트 스타일 (Emphasis)
일반 텍스트. **강조된 텍스트 (Bold)**와 *기울임꼴 (Italic)*.
두 가지 모두 사용: ***매우 중요함***.

## 2.3 목록 (Lists)

### 순서 없는 목록 (Unordered)
* 항목 A
* 항목 B
    * 중첩된 항목 1
    * 중첩된 항목 2

### 순서 있는 목록 (Ordered)
1. 첫 번째 단계
2. 두 번째 단계
   - 중첩된 목록도 가능

## 2.4 코드 블록 (Code Blocks)

### 인라인 코드 (Inline Code)
`const example = "Hello GFM";`

### 펜스 코드 블록 (Fenced Code Block)
```javascript
function greet(name) {
  console.log(`Hello, ${name}!`);
}
greet("GitHub");
```
# 3. GFM Alerts (특수 블록 요소) 테스트

## 3.1 Note (참고)
> [!NOTE]
> 이 내용은 중요한 **참고 사항**을 담고 있습니다.
>
> 여러 줄로 이어질 수 있으며, 일반적인 마크다운 문법(예: **굵게**)도 내부에서 사용할 수 있습니다.

## 3.2 Tip (팁)

> [!TIP]
> 더 나은 결과를 위한 **유용한 팁**입니다.
> 
> * 이렇게 목록도 넣을 수 있습니다.
> * 이 기능은 꼭 사용해 보세요.

## 3.3 Important (중요)
> [!IMPORTANT]
> 이 단계를 건너뛰면 **문제가 발생할 수 있는 중요한 내용**입니다.
> 
> **경고**만큼 심각하지는 않지만, 꼭 지켜야 할 사항입니다.

## 3.4 Warning (경고)
> [!WARNING]
> 예상치 못한 동작이나 **잠재적인 위험**을 경고하는 메시지입니다.
> 
> 조심해서 사용해야 합니다.

## 3.5 Caution (주의 / 위험)
> [!CAUTION]
> 이는 **가장 심각한 경고**입니다. 데이터 손실이나 시스템 손상으로 이어질 수 있는 조치에 대한 **최종 주의**입니다.


# 4. 언어 지정 코드 블록 (Syntax Highlighting)

## 4.1 python

```python
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b
```


## 4.2 코드 블록 파일명
```javascript pathfile=src/components/MyComponent.jsx
import React from 'react';

const MyComponent = () => {
  return <h1>Hello, Component!</h1>;
};

export default MyComponent;
```

# 5 링크 생성

## 5.1 ### Issue/PR 자동 링크

이 버그는 #42 에서 논의되었습니다.

## 5.2 사용자 및 저장소 Mention
이 PR을 @kyoulee 님이 검토해 주시면 좋겠습니다.
이 기능은 kyoulee/github-blog-obsidian 저장소에 추가됩니다.

Some references:
## test

*   Commit: f8083175fe890cbf14f41d0a06e7aa35d4989587
*   Commit (fork): foo@f8083175fe890cbf14f41d0a06e7aa35d4989587
*   Commit (repo): remarkjs/remark@e1aa9f6c02de18b9459b7d269712bcb50183ce89
*   Issue or PR (`#`): #1
*   Issue or PR (`GH-`): GH-1
*   Issue or PR (fork): foo#1
*   Issue or PR (project): remarkjs/remark#1
*   Mention: @wooorm

Some links:

*   Commit: <https://github.com/remarkjs/remark/commit/e1aa9f6c02de18b9459b7d269712bcb50183ce89>
*   Commit comment: <https://github.com/remarkjs/remark/commit/ac63bc3abacf14cf08ca5e2d8f1f8e88a7b9015c#commitcomment-16372693>
*   Issue or PR: <https://github.com/remarkjs/remark/issues/182>
*   Issue or PR comment: <https://github.com/remarkjs/remark-github/issues/3#issue-151160339>
*   Mention: <https://github.com/ben-eb>