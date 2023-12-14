# Firebase

# 1. 소개

Firebase 는 구글이 제공하는 웹 애플리케이션 개발 플랫폼이다. 

만약 쇼핑몰 웹 애플리케이션을 만들고 있다면, 서버 개발자가 할 일은 다음과 같다.

- 사용자 인증
- 데이터베이스 관리
- 클라우드 저장소
- 실시간 알림
- REST API

이것을 직접 구현하는 것은 매우 어렵고 힘든 일이다. Firebase 는 웹 애플리케이션에서 서버 개발에 기본적으로 요구되는 사항을 대부분 이미 구현해두었고, 개발자는 해당 기능을 선택해 사용하기만 하면 된다.

Firebase 는 여러 서비스로 구성되어 있지만, 핵심 서비스는 Firestore 이다. 

Firestore : SQL 을 사용하지 않는 문서형 데이터베이스

 

문서형 데이터베이스는 MySQL 로 대표되는 관계형 데이터베이스와 큰 차이점을 가진다.

게시판을 만든다고 가정해보자. 데이터는 총 세 개의 테이블을 사용해 다음과 같이 저장될 것이다.

**member**

| member_id | name | avatar | description |
| --- | --- | --- | --- |
| 1 | David | david.jpg | 저는 개발자입니다. |
| 2 | Alice | alice.jpg | 저는 디자이너입니다. |

**article**

| article_id | date | title | content | like_count | member_id |
| --- | --- | --- | --- | --- | --- |
| 1 | 2023-12-14 | 안녕하세요! | 반갑습니다 여러분 | 10 | 1 |
| 2 | 2023-12-15 | 오늘의 디자인 | 어떤지 확인 부탁드립니다. | 5 | 2 |

**comment**

| comment_id | date | content | like_count | member_id | article_id |
| --- | --- | --- | --- | --- | --- |
| 1 | 2023-12-14 | 안녕하세요! | 3 | 2 | 1 |
| 2 | 2023-12-15 | 반갑습니다! | 1 | 1 | 1 |

위 테이블에서 게시글 2번을 가져오고 싶다면, 다음 REST API 를 만들어야 한다.

```
GET /articles/:id
```

SQL 로 구현한다면 다음과 같을 것이다.

```sql
SELECT * FROM article WHERE article_id = id;
```

중요한 것은 이를 구현하는 과정이다.

1. 프로젝트를 생성한다.
2. DB 연결이 정상인지 테스트한다.
3. 예외처리를 고려하여 서비스 로직을 작성한다.
4. HTTP 요청이 왔을 때 서비스를 호출한다.
5. 코드 작성 과정 중 유닛 테스트를 실시한다.
6. 빌드 후, 서버에 업로드한다.
7. 유지보수

위는 가장 기본적인 과정만 나열한 것이며, 인증, 관리자권한, 로그분석 등을 포함하면 과정은 더 복잡해진다.

Firestore 의 과정과 비교해보자. 문서형 데이터베이스이므로, 다음 형태로 저장된다.

```json
{
  "members": {
    "advagwefSDFAEdca": {
      "name": "David",
      "avatar": "david.jpg",
      "description": "저는 개발자입니다."
    },
    "qeqrDSdfEEFQF": {
      "name": "Alice",
      "avatar": "alice.jpg",
      "description": "저는 디자이너입니다."
    }
  },
  "articles": {
    "DCdfqerfbqRQF": {
      "date": "2023-12-14",
      "title": "안녕하세요!"
      "content": "반갑습니다 여러분",
      "like_count": 10,
      "member_id": "1",
      "comments": {
        "DCdffqceqerdD": {
          "date": "2023-12-14",
          "content": "안녕하세요!",
          "like_count": 3,
          "member_id": "qeqrDSdfEEFQF",
        },
        "GRasdeqdcqaqxca": {
          "date": "2023-12-15",
          "content": "반갑습니다!",
          "like_count": 1,
          "member_id": "qeqrDSdfEEFQF",
        }
      }
    },
    "cWQCxcqewrqcc": {
      "date": "2023-12-15",
      "title": "오늘의 디자인"
      "content": "어떤지 확인 부탁드립니다.",
      "like_count": 5,
      "member_id": "qeqrDSdfEEFQF",
      "comments": {}
    }
  }
}
```

테이블을 사용하는 SQL 데이터베이스와 비교했을 때의 장점은, 클라이언트에서 요청 즉시 데이터를 받아올 수 있다는 것이다. 즉, 별도로 REST API 를 구현할 필요가 없다.

특이사항으로, 아이디 항목이 이상하다. 이는 Firestore 에서 무작위의 문자열로 아이디를 생성하는것이 기본이기 때문이다.

특정 아이디에 해당하는 게시글을 가져오고 싶으면 다음의 요청을 보낸다.

```jsx
const docSnapshot = await getDoc(doc(db, "articles", id));
```

HTTP 요청 실행 시, 다음 결과를 받아올 수 있다.

```jsx
{
  article_id: "cWQCxcqewrqcc",
  date: "2023-12-15",
  title: "오늘의 디자인",
  content: "어떤지 확인 부탁드립니다."
  like_count: 5,
  member_id: "2",
  comments: [],
}
```

즉, Firestore 를 사용하면 SQL 데이터베이스를 사용하는 일반적인 REST API 개발의 복잡한 절차 없이 즉시 서버 운영이 가능하다.

# 2. 프로젝트 만들기

Firebase 사이트에 접속하자. 구글 계정이 필요하다.

[https://firebase.google.com/?hl=ko](https://firebase.google.com/?hl=ko)

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled.png)

시작하기 버튼을 누른다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%201.png)

프로젝트 만들기 버튼을 누른다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%202.png)

프로젝트 이름: sample

체크박스 체크 후 계속 버튼을 누른다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%203.png)

구글 애널리틱스 사용 여부를 묻는 화면이다. 계속 버튼을 클릭한다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%204.png)

대한민국을 선택하고, 프로젝트 만들기 버튼을 클릭한다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%205.png)

계속 버튼을 누른다.

# 3. 요금제

무료와 유료 요금제가 있다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%206.png)

인증과 Firestore 서비스를 함께 사용한다고 가정했을 때, 주요 차이점은 다음과 같다.

|  | Spark (무료) | Blaze (유료) |
| --- | --- | --- |
| 인증 | SMS 10개/일 | 종량제 |
| Storage | 1GiB | 종량제 |
| Read | 50,000/일 | 종량제 |
| Write | 20,000/일 | 종량제 |
| Delete | 20,000/일 | 종량제 |
| Google Cloud 연동 | X | O |

즉, 학습하는 용도로는 유료 버전을 사용할 이유가 없다.

# 4. Firestore 생성

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%207.png)

왼쪽 사이드바 - 빌드 - Firestore Database 를 선택한다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%208.png)

프로젝트 바로가기 항목에 Firestore Database 가 등록되었다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%209.png)

데이터베이스 만들기 버튼을 클릭한다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2010.png)

위치는 Seoul 로 설정한 후, 다음 버튼을 클릭한다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2011.png)

"테스트 모드에서 시작" 을 선택한다. 해당 보안 규칙은 추후 변경 가능하며, 30일 이후에 프로덕션 모드로 자동 변경된다.

이후, 사용 설정 버튼을 클릭한다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2012.png)

위 화면과 같은 작업 창을 확인할 수 있다.

# 5. 컬렉션

컬렉션 시작 버튼을 클릭하자.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2013.png)

컬렉션은 관계형 데이터베이스의 테이블 역할을 한다. 복수로 네이밍한다.

`articles` 로 만들고, 다음 버튼을 클릭하자.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2014.png)

문서는 관계형 데이터베이스의 row 역할을 한다.

여기서, ID 를 자동 설정하는 옵션이 있는데, 무작위 문자열을 설정하는 권장되는 방법이지만 원활한 실습을 위해 우선 숫자로 ID 를 명시하도록 하겠다.

문서 ID : 1

필드1: date (timestamp) = 2023년 12월 14일 오전 11:36:09

필드2 : title (string) = 안녕하세요!

필드3: content (string) = 반갑습니다 여러분!

저장 버튼을 클릭하자.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2015.png)

위 화면처럼 작성된 문서를 확인할 수 있다.

문서를 추가하자.

문서 ID : 2

필드1: date (timestamp) = 2023년 12월 14일 오전 11:50:26

필드2 : title (string) = 안녕하세요!

필드3: content (string) = 반갑습니다 여러분!

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2016.png)

두 개의 문서가 작성되었다.

# 6. firebaseConfig 객체

데이터 패치를 하기 전, firebaseConfig 객체를 가져와야 한다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2017.png)

왼쪽 사이드바 - 프로젝트 개요 오른쪽 톱니바퀴 - 프로젝트 설정

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2018.png)

아랫쪽 "내 앱" 카드에서 </> 버튼을 클릭하자.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2019.png)

앱 닉네임을 "app" 으로 정하고, 호스팅은 체크하지 않은 채로 앱 등록 버튼 클릭

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2020.png)

HTML 로 간단하게 테스트할 것이기 때문에, <script> 태그 사용을 선택.

일단, 아래 출력되는 코드를 복사하지말고 버튼을 눌러 프로젝트 설정으로 돌아간다.

이후, 프로젝트 설정 "내 앱" 항목에서 CDN 을 선택하고, 전체 코드를 복사하지 말고 `firebaseConfig` 부분만 복사한다.

```jsx
const firebaseConfig = {
  apiKey: "",
  authDomain: "",
  projectId: "",
  storageBucket: "",
  messagingSenderId: "",
  appId: "",
  measurementId: ""
};
```

# 7. Firestore JavaScript API

저장된 데이터를 가져와보자.

먼저, 세팅 코드이다. `index.html` 을 하나 만들자.

Firebase 사용 시엔 일반적으론 Fetch API 또는 Axios 를 사용하지 않고, Firestore JavaScript API 를 사용한다. 아래는 `index.html` 에 Firestore JavaScript API 를 세팅하는 코드다.

`firebaseConfig` 는 각각의 앱마다 다르므로, 직접 복사 붙여넣기하자.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <h1>데이터 패치</h1>

    <script type="module">
      import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
      import {
        getFirestore,
        collection,
        getDocs,
      } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

      // 이 부분을 교체하세요
      const firebaseConfig = {};

      const app = initializeApp(firebaseConfig);
      const db = getFirestore(app);
    </script>
  </body>
</html>
```

# 8. CRUD

## 8-1. Read All

먼저, 전체 가져오기는 다음과 같이 작성한다.

```jsx
// 전체 가져오기
async function fetchArticles() {
  try {
    const querySnapshot = await getDocs(collection(db, "articles"));
    const articles = querySnapshot.docs.map((doc) => ({
      articleId: doc.id,
      ...doc.data(),
    }));
    console.log(articles);
  } catch (error) {
    console.error(error);
  }
}

fetchArticles();
```

하나하나 살펴보자.

```jsx
const querySnapshot = await getDocs(collection(db, "articles"));
```

`getDocs` 는 Firestore 에서 컬렉션을 가져올 때 사용한다.

`collection` 의 두번째 매개변수가 컬렉션 이름이다.

```jsx
const articles = querySnapshot.docs.map((doc) => ({
  articleId: doc.id,
  ...doc.data(),
}));
console.log(articles);
```

다음, `articles` 를 가져와서 JavaScript 에서 쓰이는 객체 배열로 만드는 코드이다. 데이터는 `docs` 안에 있으며, `map` 을 사용해 `doc.id` 를 가져온 후, `articleId` 라는 키를 만들어 배정한다. 그리고 spread 문법을 사용해 `doc.data()` 의 실행결과를 풀어낸다.

만들어진 `index.html` 파일을 더블클릭하고, 개발자도구(F12) 를 연 후, Console 탭에 들어가 출력 결과를 확인하자.

이후, 다음과 같은 결과가 나온다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2021.png)

`date` 의 경우, Timestamp 타입이기 때문에 seconds 와 nanoseconds 로 나뉘어져있다. 이것을 실제 날짜 시간으로 바꾸는 코드는 다음과 같다. Day.js 라이브러리를 활용했다.

```html
<script src="https://unpkg.com/dayjs"></script>
<script module>
// ...
	const date = new Date(date.seconds * 1000);
	const formattedDate = dayjs(date).format(
	  "YYYY년 MM월 DD일 HH시 mm분 ss초"
	);
// ...
</script>
```

이를 종합적으로 활용해, 접속과 동시에 화면에 데이터를 출력해보도록 하겠다. 로딩 아이콘도 포함했다. `firebaseConfig` 부분을 교체하는것을 잊지말자!

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <link
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css"
      rel="stylesheet"
    />
  </head>
  <body>
    <div class="container py-5">
      <h1 class="mb-5">게시판</h1>
      <div id="articles" class="row row-cols-1 row-cols-md-3 g-4"></div>
      <div
        id="loading"
        class="position-fixed top-50 start-50 translate-middle"
        style="display: none"
      >
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
      </div>
    </div>

    <script src="https://unpkg.com/dayjs"></script>
    <script type="module">
      import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
      import {
        getFirestore,
        collection,
        getDocs,
      } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

      // 이 부분을 교체하세요
      const firebaseConfig = {};

      const app = initializeApp(firebaseConfig);
      const db = getFirestore(app);

      // 전체 가져오기
      async function fetchArticles() {
        try {
          document.getElementById("loading").style.display = "block";
          const querySnapshot = await getDocs(collection(db, "articles"));
          const articles = querySnapshot.docs.map((doc) => {
            const data = doc.data();
            const date = new Date(data.date.seconds * 1000);
            const formattedDate = dayjs(date).format(
              "YYYY년 MM월 DD일 HH시 mm분 ss초"
            );
            return {
              articleId: doc.id,
              title: data.title,
              date: formattedDate,
              content: data.content,
            };
          });
          displayArticles(articles);
        } catch (error) {
          console.error(error);
        } finally {
          document.getElementById("loading").style.display = "none";
        }
      }

      function displayArticles(articles) {
        const articlesDiv = document.getElementById("articles");
        articles.forEach((article) => {
          const articleDiv = document.createElement("div");
          articleDiv.classList.add("col");
          articleDiv.innerHTML = `
            <div class="card h-100">
              <div class="card-body">
                <h5 class="card-title">${article.title}</h5>
                <p class="card-text">${article.content}</p>
              </div>
              <div class="card-footer">
                <small class="text-muted">${article.date}</small>
              </div>
            </div>
          `;
          articlesDiv.appendChild(articleDiv);
        });
      }

      fetchArticles();
    </script>
  </body>
</html>
```

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2022.png)

## 8-2. Read By ID

먼저 코드를 원래대로 돌려놓자. `firebaseConfig` 를 수정하는것을 잊지말자!

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <h1>데이터 패치</h1>

    <script type="module">
      import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
      import {
        getFirestore,
        collection,
        getDocs,
      } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

      // 이 부분을 교체하세요
      const firebaseConfig = {};

      const app = initializeApp(firebaseConfig);
      const db = getFirestore(app);
    </script>
  </body>
</html>
```

아이디 기준 가져오기는 다음과 같다.

```jsx
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import {
  getFirestore,
  doc,
  getDoc,
} from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

// ...

// 아이디로 가져오기
async function fetchArticle(id) {
  try {
    const docSnapshot = await getDoc(doc(db, "articles", id));
    if (docSnapshot.exists()) {
      const article = {
        articleId: docSnapshot.id,
        ...docSnapshot.data(),
      };
      console.log(article);
    } else {
      console.log("No such document!");
    }
  } catch (error) {
    console.error(error);
  }
}

fetchArticle("1");
```

`doc` 과 `getDoc` 을 사용하므로, `import` 구문을 수정해주자.

articles 컬렉션에서 1번 아이디를 가진 문서를 가져오게 된다. 단, `id` 는 Firestore 에 전달될 때 반드시 string 이어야한다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2023.png)

## 8-3. Create

새로운 문서를 추가할 때는 `addDoc` 을 사용한다.

```jsx
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import {
  getFirestore,
  collection,
  addDoc,
  Timestamp,
} from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

// ...

// 새로운 문서 추가하기
async function createArticle(article) {
  const date = new Date();
  const timestamp = Timestamp.fromDate(date);
  const newArticle = { ...article, date: timestamp };
  try {
    const docRef = await addDoc(collection(db, "articles"), newArticle);
    console.log("Document written with ID: ", docRef.id);
  } catch (error) {
    console.error("Error writing document: ", error);
  }
}

const article = {
  title: "Firebase",
  content: "즐거운 Firebase 시간!",
};

createArticle(article);
```

Firestore 의 Timestamp 타입이 필요하므로, `import` 하도록 한다. 

콘솔에 작성된 결과는 다음과 같다.

```
Document written with ID:  6RRNTyi7eeJx9UN2J8kr
```

즉, 새로 만들어진 문서의 아이디는 1, 2, 3 처럼 순차적으로 생성되는 것이 아니라, 무작위의 문자열로 생성된다.

- NoSQL Database 에선 이런 방식이 표준적이다. MySQL 테이블에서의 Auto Increment 방식을 억지로 적용하려 할 경우, 성능 하락이 발생할 수 있다고 한다.

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2024.png)

새로 생긴 문서가 확인되며, 데이터도 잘 저장되었다.

## 8-4. Delete By ID

문서를 삭제할 땐 `deleteDoc` 을 사용한다.

```jsx
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import {
  getFirestore,
  doc,
  deleteDoc,
} from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

// ...

// 아이디로 문서 삭제하기
async function deleteArticle(id) {
  try {
    await deleteDoc(doc(db, "articles", id));
    console.log("Document successfully deleted!");
  } catch (error) {
    console.error("Error deleting document: ", error);
  }
}

deleteArticle("1");
```

콘솔에 출력된 메세지는 다음과 같다.

```
Document successfully deleted!
```

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2025.png)

1번 아이디를 가진 문서가 잘 삭제되었음을 알 수 있다.

## 8-5. Update By ID

문서를 수정할 땐 `updateDoc` 을 사용한다.

2번 아이디를 가진 문서를 수정해보자.

```jsx
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import {
  getFirestore,
  doc,
  updateDoc,
  Timestamp,
} from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

// ...

// 아이디로 문서 업데이트하기
async function updateArticle(article) {
  const date = new Date();
  const timestamp = Timestamp.fromDate(date);
  const updatedArticle = { ...article, date: timestamp };
  try {
    await updateDoc(doc(db, "articles", article.articleId), updatedArticle);
    console.log("Document successfully updated!");
  } catch (error) {
    console.error("Error updating document: ", error);
  }
}

const article = {
  articleId: "2",
  title: "이건 수정된 title",
  content: "이건 수정된 content",
};

updateArticle(article);
```

콘솔에 출력된 메세지는 다음과 같다.

```
Document successfully updated!
```

![Untitled](Firebase%2086a877ded68e40d89d2722cb9328663e/Untitled%2026.png)

잘 수정된 것이 확인되며, `date` 도 현재 시간 기준으로 수정되었다.