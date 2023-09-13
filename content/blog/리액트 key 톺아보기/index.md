---
title: 리액트 key 톺아보기
date: "2023-09-11T17:34:37.121Z"
---

written by **ryong**

### **React Key란?**

배열 요소들을 `map()`을 통해 리액트 컴포넌트로 렌더링할 때 주로 마주치게 되는 `key` 속성은 사실 리액트 컴포넌트에 있어 꽤나 중요한 녀석입니다.

리액트에서 ‘`key` 속성이 왜 필요한 지’에 대해 생각해보도록 하죠.

여러분의 컴퓨터에 이름을 갖고 있지 않은 파일들이 한 폴더에 여러 개 있다고 가정해봅시다. 이름이 없는 파일들은 각 파일들이 배치된 순서로 서로서로를 구분하고 있습니다.

만약 여러분이 그 파일 중 하나를 삭제하게 된다면, 상황은 복잡해질 것입니다. 두 번째 파일이 첫 번째 파일이 되고, 세 번째 파일은 두 번째로, 이러한 과정이 반복되어 변할테니까요.

그리고 각 파일이 어떤 파일이었는지 내용을 직접 들어가서 확인을 해 봐야 알 수 있는 불편한 상황이 펼쳐질 겁니다.

폴더에 있는 파일의 이름과 배열에서의 JSX `key` 속성은 비슷한 역할을 합니다.

`key` 속성은 리액트 컴포넌트가 렌더링 되는 동안 형제 컴포넌트 간 고유한 식별을 할 수 있게 도와주기 때문이죠.

이에 따라 배열의 순서가 변경되면서 위치가 변경되더라도, key 속성을 통해 생명주기 동안 컴포넌트를 식별하게 됩니다.

### **Key를 사용하는 방법**

**서버에서 받는 DB 데이터**

보통 서버에서 전달받는 데이터 중에서 여러 항목들이 배열로 감싸져 전달되는 경우에는 각 항목에 고유한 key 값들이 포함되어 있습니다.

**직접 key 값 추가하기**

직접 리액트에서 `key` 값을 다룰 때에는 아래와 같이 몇 가지 규칙이 수반됩니다.

### **Key 설정에 필요한 규칙**

-   `key` 값은 렌더링이 되는 동안 생성되면 안 되고, 미리 데이터에 만들어져 포함되어야 합니다.
    
    incrementing counter, uuid, crypto.randomUUID() 등..
    
-   `key` 값은 리렌더링이 되더라도 변하지 않는 고유한 값이어야 합니다.
    
    따라서 `<Input key={Math.random()} />` 와 같이 변하는 값을 넣는 것은 반드시 지양해야 합니다.
    
-   `index`로 `key` 값을 사용하는 경우에는 두 가지의 조건을 모두 만족하는 경우에만 사용해야 합니다.
    
    1.  리스트에 수정, 삭제, 삽입을 할 일이 없을 때
    2.  리스트의 순서가 변하지 않을 때
    
    [변하는 리스트에서 key를 배열의 index로 사용할 경우 일어나는 현상을 소개한 글](https://velog.io/@ssoon-m/react-key-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%8B%A4%EB%A3%A8%EA%B8%B0#%EB%B3%80%ED%95%98%EB%8A%94-%EB%A6%AC%EC%8A%A4%ED%8A%B8%EC%97%90%EC%84%9C-key%EB%A5%BC-%EB%B0%B0%EC%97%B4%EC%9D%98-index%EB%A1%9C-%EC%82%AC%EC%9A%A9%ED%95%9C%EB%8B%A4%EB%A9%B4)을 링크에 첨부합니다.
    
-   `key` 속성은 리액트 자체에서 컴포넌트를 구분하기 위한 힌트로 사용되고, 하위 컴포넌트의 속성으로는 전달되지 않습니다.
    
-   `<></>` 에 key 속성을 추가하기 위해서는 아래와 같은 조치가 필요합니다.
    
    ```tsx
    import { Fragment } from 'react';
    
    // ...
    
    const listItems = people.map(person =>
      <Fragment key={person.id}>
        <h1>{person.name}</h1>
        <p>{person.bio}</p>
      </Fragment>
    );
    
    ```
    

### **Key를 응용하여 문제 해결하기**

```tsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}

```

`ProfilePage` 컴포넌트는 외부로부터 `userId` 속성을 넘겨받고 있는 것을 알 수 있습니다.

이 컴포넌트에는 코맨트를 입력하는 기능이 포함되어 있으며, 해당 코맨트는 `comment` 변수로 상태가 관리되고 있습니다.

만약 `userId`를 `1`로 전달받아 1번 유저에 대한 `ProfilePage`를 보여주다가, `userId`가 `2`로 변경된 페이지로 navigate 한다면 어떤 현상이 일어날까요?

변경된 2번 유저에 대한 `ProfilePage`를 보여주겠지만, `comment` 값은 `ProfilePage`가 처음 렌더링 됐을 때처럼 `''`로 초기화되지 않고, 이전에 사용되던 `comment` 상태 값을 그대로 보여주게 됩니다.

이러한 이슈를 해결하기 위해서 컴포넌트가 렌더링 됐을 때 상태값을 초기화 시키는 위 코드와 같은 `useEffect`를 사용하는 방식은 복잡성을 증가시킬 뿐 효율적인 방법이 아닙니다.

`key` 속성을 사용하면 userId가 달라졌을 때, 리액트에게 완전히 새로운 컴포넌트로 렌더링하도록 설정할 수 있습니다.

하나로 뭉쳐있는 코드를 분리하여 아래와 같이 리팩토링 해봅시다.

```typescript
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState('');
  // ...
}

```

`userId`가 달라짐에 따라 컴포넌트에 `key` 속성을 다르게 설정하면,

리액트는 완전히 다른 새로운 컴포넌트로 인식하여 `Profile` 컴포넌트가 처음 렌더링됐을 때 상태값을 초기화하는 로직을 진행하게 됩니다.

이처럼 리액트 `key` 속성을 사용하면 원치 않는 렌더링 이슈를 잡을 수 있습니다.