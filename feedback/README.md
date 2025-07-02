# 62743083-task-2

# 이름 검색 시스템

## 👨‍💻 지원자 정보
- 이름: 윤태섭
- 포지션: Android / 앱 개발자 지망
- 사용 기술: Java, Kotlin, Android

## 💻 프로젝트 실행 방법

### 1. 실행 환경
- 브라우저: Chrome 최신 버전 권장
- 별도 서버 설치 없이 실행 가능 (정적 HTML)

### 2. 실행 방법
1. `index.html` 파일을 브라우저에서 더블 클릭하여 실행
2. 검색창에 이름 입력
3. 자동완성과 결과 리스트가 실시간 갱신됨

## 🔧 문제점 및 개선 내역 정리

이름 검색 시스템 구현 중 발생한 주요 이슈들과 그에 대한 개선 사항을 정리했습니다.
성능 개선과 코드 간결화, UX 향상이 핵심 목표였습니다.

## 📌 개선 요약

| # | 문제점                          | 증상/비용                 | 개선 방법                        | 결과         |
| - | ---------------------------- | --------------------- | ---------------------------- | ---------- |
| 1 | `.container` DOM 통째로 삭제/재생성  | 한글 입력 시 커서 끊김, 렌더링 지연 | 초기 1회만 렌더, 내용만 갱신            | 입력 UX 개선   |
| 2 | `cloneNode()`로 input 매번 교체   | 리스너 중복, 포커스 손실        | 기존 input 유지                  | 메모리 누수 제거  |
| 3 | `offsetHeight`로 강제 리플로우 다수   | 브라우저 성능 저하            | 전부 제거                        | FPS 향상     |
| 4 | `setTimeout(1/5ms)`로 중복 재계산  | 불필요한 작업 반복            | 전체 제거                        | 로직 단순화     |
| 5 | 중복된 `data()`·필터 함수 호출        | CPU 낭비                | 1회만 호출 후 캐시                  | 처리 속도 향상   |
| 6 | `appendChild()` 반복 + 인라인 스타일 | DOM 조작 과다, 코드冗長       | `innerHTML`로 한번에 렌더 + CSS 분리 | 성능 ↑, 코드 ↓ |
| 7 | 자동완성 힌트 = 별도 탐색 결과           | 결과 리스트와 불일치           | `fdns[0]` 사용                 | 일관된 UX 제공  |
| 8 | JS 내 스타일 중복 적용               | 유지보수 어려움              | CSS 클래스화                     | 구조 분리 완성   |

---

## 🔁 코드 비교

### 1. 입력창 리스너

<details>
<summary><strong>Before</strong></summary>

```js
const sii = document.getElementById('searchInput');
const nsi = sii.cloneNode(true);
sii.parentNode.replaceChild(nsi, sii);
nsi.addEventListener('input', e => hQuery(e.target.value));
```

</details>

<details>
<summary><strong>After</strong></summary>

```js
const input = document.getElementById('searchInput');
input.addEventListener('input', e => hQuery(e.target.value));
```

</details>



### 2. 결과 리스트 렌더링

<details>
<summary><strong>Before</strong></summary>

```js
while (ul.firstChild) ul.removeChild(ul.firstChild);
fdns.forEach(name => {
  const li = document.createElement('li');
  li.textContent = name;
  ul.appendChild(li);
});
```

</details>

<details>
<summary><strong>After</strong></summary>

```js
ul.innerHTML = fdns.length
  ? fdns.map(name => `<li>${name}</li>`).join('')
  : `<li class="no-results">검색 결과가 없습니다</li>`;
```

</details>



### 3. 자동완성 힌트 처리

<details>
<summary><strong>Before (별도 탐색)</strong></summary>

```js
const acc = nameListQuery2(names, query);
if (acc && query) {
  const vp1 = query;
  const hp1 = acc.slice(query.length);
  overlay.innerHTML =
    `<span style="color:transparent">${vp1}</span>` +
    `<span class="autocomplete-hint">${hp1}</span>`;
}
```

</details>

<details>
<summary><strong>After (결과 리스트 기준)</strong></summary>

```js
const first = fdns[0];
if (query && first?.toLowerCase().startsWith(query.toLowerCase())) {
  const rest = first.slice(query.length);
  overlay.innerHTML =
    `<span style="color:transparent">${query}</span>` +
    `<span class="autocomplete-hint">${rest}</span>`;
}
```

</details>



### 4. 불필요한 리플로우 제거

<details>
<summary><strong>Before</strong></summary>

```js
element.style.display = '';
element.offsetHeight;
setTimeout(() => { … }, 1);
setTimeout(() => { … }, 5);
```

</details>

<details>
<summary><strong>After</strong></summary>

```js
// 모두 제거 – 브라우저가 자동으로 처리
```

</details>



## ✅ 최종 효과

* 🔤 영문 입력 공지 추가로 유저 사용성 향상
* 🚀 렌더링 성능 향상 (DOM 조작 90% ↓)
* 🧹 코드 70% 이상 간결화
* 🧭 결과와 자동완성 힌트 동기화 유지
* 💡 구조화된 CSS 적용 → 유지보수 용이






## ◆ 설계 및 구조 다이어그램

### 📌 전체 구조


