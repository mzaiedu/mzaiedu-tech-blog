---
title: 프런트엔드 개발자의 Figma Dev Mode 리뷰
date: "2023-08-29T20:02:11.337Z"
---

![](https://velog.velcdn.com/images/ksr20612/post/18099f7b-48d2-47dd-8e6a-4595b0b7ee4e/image.png){: width="100%" height="100%"}


지난 2023년 6월, 🔗[Figma](https://www.figma.com/)에 Dev Mode가 추가되었습니다. 감사하게도 팀의 디자이너분께서 빠르게 Dev Mode와 호환 작업을 진행해주셨고, Dev Mode를 사용해볼 수 있었습니다. 1개월 조금 넘는 기간 동안 Dev Mode를 사용해보면서 좋았던 점과 아쉬웠던 점을 정리해보았습니다.

## 👍 추가되어 좋았던 점


### 💡 사용성 향상

지난 2년 간 개발자로서 피그마를 사용하면서 가장 답답했던 사용성 이슈가 이번 Dev Mode에서 해결되었습니다. 기존 피그마에서는 선택 모드와 화면 이동 모드를 구분했고, 사용자는 버튼을 클릭하여 모드를 변경해주어야 했습니다. 

![](https://velog.velcdn.com/images/ksr20612/post/05413a29-e8c4-4dd4-a32b-df4d7339b60b/image.png){: width="100%" height="100%"}


화면을 이동하며 빠르게 요소를 선택해 속성을 확인해야 하는 개발자에게는 여간 신경쓰이는 부분이 아닐 수 없었는데요, 이번에 두 모드가 하나의 방식으로 통합되었습니다. 이제는 쉽고 빠르게 **화면을 이동하며 컴포넌트를 선택**할 수 있게 되었습니다. 😭

![](https://velog.velcdn.com/images/ksr20612/post/7a127595-5623-4f65-a6c2-99557e006e51/image.gif){: width="100%" height="100%"}

뿐만 아니라 **클릭을 중첩하여 하위 요소를 선택**하는 것도 가능해졌습니다. 컴포넌트가 중첩되어 있는 경우 좌측의 설계 패널에서 컴포넌트 트리를 확인해야 하는 것이 불편했었는데요, 이번 Dev Mode에서 이러한 불편함도 해결해주었습니다.

이처럼 피그마는 이번 Dev Mode에서 **다양한 작업을 마우스 동작 하나만으로 가능**하도록 사용 규칙을 변경하였습니다. 아주 영리한 개선이었다는 생각이 듭니다. 사용자의 멘탈 모델을 화면에 한정시킴으로써 사용자의 몰입을 해치지 않았기 때문이죠. 각 작업들과 대응되는 마우스 동작도 꽤나 직관적이라서 쉽게 적응할 수 있었습니다.

### 💡 프레임 히스토리

다음으로 좋았던 점 중 하나는 **프레임의 히스토리를 쉽게 파악할 수 있다**는 점이었습니다. Dev Mode의 우측 패널에서는 프레임의 수정 기록(*Edited 5 days ago*)과 함께 무엇이 어떻게 수정되었는지 확인할 수 있는 기능(*Compare changes*)을 제공하였는데요, 이를 통해 수정된 항목을 빠르게 파악할 수 있었습니다.

![](https://velog.velcdn.com/images/ksr20612/post/5f8dc80f-12be-4f0c-a6f1-6c1b35b0379a/image.png){: width="100%" height="100%"}


css 속성(Property)가 변경된 경우에는 이전 값과 현재 값이 모두 표시되어 빠르게 해당 부분을 찾아 수정할 수 있었습니다. 

![](https://velog.velcdn.com/images/ksr20612/post/465099ee-a43c-48c1-8f17-cb2d6f01f48b/image.png){: width="100%" height="100%"}


기존에는 피그마 커멘트나 Jira 이슈 카드를 통해 변경 사항을 알렸는데요, 양측의 프레임 패널을 이용하여 변경점을 쉽게 파악할 수 있어 좋았습니다. 

![](https://velog.velcdn.com/images/ksr20612/post/30c95580-fcc9-4b38-b2b9-f4c4bd7fcd16/image.png){: width="100%" height="100%"}


### 💡 프레임 별로 구분된 설계 패널

프레임 히스토리를 파악하기 위해서는 좌측의 설계 패널로 적절하게 활용할 필요가 있었는데요, 이번 Dev Mode에서는 **설계 패널 디자인이 변경되어 프레임을 쉽게 파악**할 수 있었습니다.

![](https://velog.velcdn.com/images/ksr20612/post/5d96a421-f2d8-471f-9c56-4a077a5387e4/image.png){: width="100%" height="100%"}


### 💡 개발 친화적인 Inspect 패널

많은 컴포넌트의 스타일링에는 피그마에 명시된 css 속성들을 그대로 사용하게 되는데, **css 속성들을 코드 스타일로 확인할 수 있어 편리**했습니다. 기존 모드에서는 불가능했던 rem / rgb 등으로의 단위 변경 기능을 덤으로 제공해주어 더 빠른 개발이 가능했습니다.

![](https://velog.velcdn.com/images/ksr20612/post/d249a609-f124-4ab7-a84b-6887cce7ac5a/image.gif){: width="100%" height="100%"}

### 💡 플레이그라운드 모드

Dev Mode에서는 컴포넌트 플레이그라운드 기능도 제공하고 있었습니다. **리액트 컴포넌트를 개발할 때 상태값을 선정하는 데 소요되는 비용이 크게 감축되어 편리**했습니다.

![](https://velog.velcdn.com/images/ksr20612/post/571a0274-e359-4f44-88be-870684697f56/image.gif){: width="100%" height="100%"}

현재 컴포넌트의 문서화와 디자인 시스템 구축을 위해 🔗[StoryBook](https://storybook.js.org/) 도입을 검토하고 있는데, 피그마의 플레이그라운드를 적절하게 사용하면 큰 도움이 될 것 같습니다. 

## 😓 그럼에도 아쉬웠던 점


### 💣 파악이 어려운 변경된 프레임

양측의 설계 패널을 이용하면 쉽게 변경된 사항을 알 수는 있지만, 다소 번거로운 작업이긴 합니다. 좌측의 패널에서 변경된 프레임을 파악하고, 우측 패널에서 상세한 변경점을 파악해야 하니까요. 

**History Mode를 추가해 전체 화면에서 변경된 프레임을 쉽게 확인하고, 프레임을 클릭하는 것으로 상세한 변경점을 알 수 있으면 좋을 것 같아요.**

변경된 프레임을 외곽선으로 표시해주면 한 눈에 파악이 쉬울 것 같아요. 이렇게요.

![](https://velog.velcdn.com/images/ksr20612/post/54ad0a36-ef68-4979-9052-66752ebe2b9c/image.png){: width="100%" height="100%"}


🔗[Beta Feedback](https://form.asana.com/?k=wksnkyJe5TlKwleZgXZHng&d=10497086658021) 에도 제출하였으니 내년 업데이트에 반영되었으면 좋겠습니다. 😊

## 마무리


개발하는 서비스 [ Rockie Talkie ] 에서는 기능명세서를 피그마상에서 작업하고 있습니다. 

![](https://velog.velcdn.com/images/ksr20612/post/e5ee083a-bee0-4931-9e8b-82d67a44ea49/image.png){: width="100%" height="100%"}


기능과 동작 정의를 화면 상으로 끌고 들어온 것인데요, 피그마의 지향점도 이와 유사하다고 생각합니다. 하나의 리소스(피그마 파일)를 통해 **개발자가 필요로 하는 것들을 쉽게 얻을 수 있도록 배려**하는 것이죠. 

> ***Today, we’re excited to introduce Dev Mode, a new workspace in Figma that’s designed to get developers what they need, when they need it, harnessing the tools they use every day.***
> 

생산성 향상이라는 목적 속에는 항상 서로 **배려**하려는 모습이 담겨 있는 것 같습니다. 프론트엔드 개발자로서 디자인 팀과의 소통에 부족한 점은 없었는지 돌아보며 글을 마칩니다.

---

### 출처

- [https://www.figma.com/blog/introducing-dev-mode/](https://www.figma.com/blog/introducing-dev-mode/)