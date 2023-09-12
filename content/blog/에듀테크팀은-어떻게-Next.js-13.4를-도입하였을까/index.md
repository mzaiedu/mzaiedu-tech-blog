---
title: 쉽게 풀어 쓴 Next.js 13.4 도입기 
date: "2023-06-06T23:46:37.121Z"
---

**[ 로키토키 ]에 Next.js v13 끼얹기**
by Dongyeon

지난 10월, Next.js 컨버런스에서 13버전이 발표되었습니다. 핵심 문법과 레이아웃 등 굵직한 변화가 있었는데요, <로키토키> 홈페이지에서의 니즈와 일치하는 부분이 있어 발빠르게 적용해보았습니다. 이번 글에서는 Next.js가 무엇인지, <로키토키> 홈페이지에서 Next.js 13을 어떻게 채택하게 되었는지, 또 어떻게 사용하였는지 간단하게 알아보도록 하겠습니다.

<br/>

# Next.js는 무엇인가요?

프런트엔드 개발에서 가장 핵심적인 것 중의 하나는 “얼마나 빠르게” 사용자가 페이지에 접근할 수 있느냐의 문제입니다. 페이지의 로딩 속도는 사용자의 이탈에 큰 영향을 주는 요소이기 때문에 복잡한 페이지를 최대한 가볍고 유연하게 유지하는 것이 핵심이라고 할 수 있습니다. 

클라이언트 사이드 렌더링 전략에서는 모든 js 파일을 서버로부터 전달받은 후 화면을 렌더링하기 시작합니다. 전달받은 리소스들을 해석하고, 화면에 알맞게 그려주는 과정이 클라이언트에서 전적으로 진행됩니다. 따라서 애플리케이션이 복잡하면 복잡할수록 js 파일 크기가 커지기 때문에, 사용자는 서비스의 첫 화면을 보기 위해 더 많은 시간을 대기할 수 밖에 없습니다. 큰 파일을 다운로드 받고, 코드를 해석하고, 화면에 그리는 데에 상대적으로 많은 시간과 비용이 발생하기 때문입니다.

이러한 로딩 속도 이슈를 해결하기 위해 등장한 것이 “서버사이드 렌더링”이라는 렌더링 전략입니다. 사실 서버라이드 렌더링은 전혀 새로운 전략은 아닙니다. php와 jsp 등에서는 이미 서버사이드 렌더링으로 화면을 그려내고 있었죠. 

서버사이드 렌더링 전략은 클라이언트 사이드 렌더링 전략과는 반대의 전략을 사용합니다. 즉, 서버에서 클라이언트 측에 전달할 화면을 미리 그려내는 것입니다. 화면을 미리 준비해놓았기 때문에 클라이언트에서는 전달받은 화면을 단순히 보여주기만 하면 됩니다. 서버사이드 렌더링 전략은 첫 페이지의 빠른 로딩, 보안, 오래된 브라우저에서의 호환성, 검색 엔진 최적화 등의 영역에서 클라이언트 사이드 렌더링 전략에 비해 강점이 있습니다.

<img src="https://velog.velcdn.com/images/ksr20612/post/785bbb82-46a7-4200-88f2-225beccfa1f6/image.png" width="50%" height="50%">
<br/>

리액트 진영에서도 당연히 서버사이드 렌더링이 가능합니다. Next.js는 React 라이브러리를 감싼 프레임워크로 리액트의 서버사이드 렌더링을 손쉽게 구현할 수 있도록 많은 기능들을 제공하고 있습니다. 심지어는 스크립트를 빌드하는 시점에 미리 페이지를 렌더링하는 정적 사이트 생성(SSG) 방법도 제공하므로, 구현하고자 하는 서비스의 속성에 적합한 렌더링 전략을 선택할 수 있다는 강점이 있습니다.

<br/> 

# 12와 13 사이에서
[로키토키] 홈페이지의 목적을 고려할 때, Next.js를 이용하는 것이 적합하고 판단한 이유는 다음과 같습니다.

- 잠재적 고객을 만나는 첫 지점이기 때문에 좋은 성능, 즉 빠릿빠릿한 로딩 속도를 보여주어야 합니다.
- 검색엔진(구글과 네이버 등)에서 상단에 노출될 수 있도록 웹크롤러에게서 좋은 점수를 획득하여야 합니다.
- 많은 이미지를 포함하기 때문에 이미지 처리를 최적화할 수 있어야 합니다.

다만, 현재 Next.js는 격변기를 겪고 있습니다. 지난 10월 13버전이 발표된 이후, 현재까지도 꾸준히 마이너 업데이트가 계속되고 있습니다(한달도 안된 5월 5일, 13.4 버전이 업데이트되었습니다). 문제는 버전이 업데이트될 때마다 굉장히 많은 것이 바뀌고 있다는 점입니다. 불과 한 달 전에 작성된 블로그 글들이 현재 버전에는 맞지 않아 이미 유물로 전락해버리고 말았습니다.

<p align="center" style="color:gray">
<img src="https://velog.velcdn.com/images/ksr20612/post/3d00e399-1fbd-43ac-9087-720d7c5d110c/image.png" width="50%" >
</p>
<br/>

레퍼런스의 빈곤과 불완전함 사이에서도 Next.js 13버전을 선택한 이유는 13버전에서 변화한 것들이 앞으로 Next.js이 궁극적으로 변화하려는 방향의 시작점이기 때문입니다. 앞으로의 변화는 13버전의 구조 위에 쌓아올라갈 것이므로, 과감하게 Next.js 13을 도입하기로 결정하였습니다(물론 홈페이지의 초안이 단일 페이지로 복잡하지 않아 도입이 쉽기도 했습니다).

<br/>

# 이러한 것들을 적용해보았습니다

[로키토키]에 적용한 next.js의 단편을 몇 가지 소개해보겠습니다.

### 1. @next/font를 통한 웹 폰트 최적화
```typescript
  // layout.tsx
  import localFont from 'next/font/local';
  const ssurround = localFont({ src: '../public/fonts/Ssurround.ttf', variable: '--font-ssurround' });
  const nanumSquare = localFont({ src: '../public/fonts/NanumSquareRound.ttf', variable: '--font-nanumsquare' });

  // tailwind.config.js
  theme: {
      extend: {
        fontFamily: {
          sans: ['var(--font-inter)'],
          surround: ['var(--font-ssurround)'],
          square: ['var(--font-nanumsquare)'],
        },
     }
  }
```
Next.js 13버전에는 @next/font라는 패키지가 새롭게 추가되었습니다. @next/font 패키지는 폰트의 사전 로딩, 레이아웃 변동(cumulative layout shift)를 방지하기 위한 adjustFallbackFont 기능 등이 포함되어서 뛰어난 사용자 경험을 제공하는 데 초점을 두고 있습니다. 

[로키토키]에서는 layout.tsx에서 커스텀 폰트를 빠르게 로딩하여 css의 전역변수로 설정하였고, tailwind.config.js에서 변수를 확장하여 html 엘리먼트에서 해당 변수들을 클래스명으로 사용하였습니다. 구글 폰트를 사용하지는 않았지만, 구글 폰트를 사용하였다면 엄청난 성능 향상을 경험할 수 있었을 겁니다. Next.js 13버전은 빌드 시점에 구글 폰트를 미리 호스팅해놓기 때문입니다. 폰트 파일도 일종의 리소스이기 때문에 외부에서 다운받는 데 추가적인 시간과 비용이 들기 마련입니다. 하지만 @next/font 패키지에서는 구글 폰트 파일을 빌드 시점에 파일을 미리 다운로드 받아 저장해놓고 있기 때문에 폰트 파일의 빠른 로드가 가능합니다. 즉, 서로 다른 도메인으로 리소스를 주고받는 데 발생하는 비용이 최소화되기 때문에 비교적 빠른 속도로 폰트 파일을 다운로드 할 수 있습니다.
<br/> 

### 2.@next/image를 이용한 이미지 최적화
```
  import Image from 'next/image'
  import Launch from '../../public/jpgs/rockie_launch@2x.jpg';

  <Image 
      src={Launch}
    alt={'Rockie with a spaceship'} 
    width={400}
    height={400}
    placeholder='blur'
  />
```
Next.js에서는 또한 @next/image 패키지를 제공하기 때문에 이미지도 쉽게 최적화할 수 있습니다. Next.js의 이미지 최적화 전략은 이미지가 많은 [로키토키] 홈페이지에 안성맞춤이었습니다. 

@next/image 패키지는 Image 컴포넌트로 생성한 모든 이미지 객체에 지연 로딩(화면 안에 있는 이미지만 로드하는 기능)을 적용하기 때문에 불필요한 대역폭 낭비를 줄이고, 화면에 보여줄 이미지만 빠르게 로드할 수 있습니다. 자바스크립트에서 지연 로딩을 구현하려면 스크롤 이벤트나 Intersection Observer를 이용하여 이미지 객체가 화면에 들어오는 시점을 포착해주어야 하는데, 이러한 복잡한 로직을 자동으로 해주니 얼마나 좋은 패키지입니까.

또한 @next/image 패키지는 이미지의 포맷과 사이즈도 자동으로 브라우저에 맞춰 최적화합니다. 모바일로 [로키토키]에 접속한다면, 굳이 큰 용량과 사이즈의 jpg 파일을 다운로드 받을 필요 없이, 모바일 화면에 맞는 크기의 이미지를 다운받는 편이 로딩에 유리합니다. 이를 구현하려면 디바이스 크기와 이미지의 포맷을 고려하여 srcSet 태그 안쪽에서 순차적으로 이미지를 지정해주어야 하는데요, 지정할 포맷(webp, jpeg 등)과 사이즈 별 이미지 각각을 서버에서 보유하고 있어야 합니다(혹은 변환하는 일련의 로직이 필요합니다). @next/image 패키지는 이를 알잘딱깔센하게 처리해줍니다.

![](https://velog.velcdn.com/images/ksr20612/post/64af5607-c2d1-4b58-a889-cd3cb5b7683d/image.png)
<p align="center" style="color:gray"></p>

추가적으로 [로키토키] 홈페이지에서는 이미지 컴포넌트에 placeholder 속성을 추가하여 이미지 로드에 따른 레이아웃 변동(layout shift)를 막고, 뛰어난 사용자 경험을 제공하려 blur 값을 설정하여 이미지의 로딩 전에 빈 공간을 보여주는 대신 불투명한 이미지를 보여주도록 설정하였습니다.
<br/>

### 3. 메타태그로 검색엔진 최적화하기

```javascript
  export const metadata = {
	metadataBase:  new  URL('https://eng.rockie-talkie.com'),
	title:  'RockieTalkie | 로키토키',
	description:  '영어를 마법처럼 배우는 우주 여행이 시작됩니다',
	keyword:  'rockie-talkie, RKTK, english, book, reading',
	author:  'mediazen edutech',
	openGraph: {
		title:  'RockieTalkie | 로키토키',
		description:  '영어를 마법처럼 배우는 우주 여행이 시작됩니다',
		url:  'https://rockie-talkie.com',
		type:  'website',
		locale:  'ko_KR',
		images:  '/jpgs/rktk-logo.jpg',
		site_name:  'Rockie Talkie',
	},
	twitter: {
		title:  'RockieTalkie | 로키토키',
		description:  '영어를 마법처럼 배우는 우주 여행이 시작됩니다',
		url:  'https://rockie-talkie.com',
		type:  'website',
		locale:  'ko_KR',
		images:  '/jpgs/rktk-logo.jpg',
		site_name:  'Rockie Talkie',
	},
  }
```
Next.js 13버전에서는 기존 <head> 내에서 정의했던 메타 태그들을 metadata라는 객체 형태로 선언하도록 변경되었습니다. 메타 태그는 검색 엔진 최적화를 위해 사용되는 태그들로, SEO태그라고도 불립니다. 검색 엔진의 웹 크롤러는 웹사이트를 미리 방문하여 웹페이지가 어떤 컨텐츠를 제공하고 어떤 구조를 가지고 있는지를 수집합니다. 수집된 정보는 검색 엔진의 검색 결과나 랭크에 영향을 줄 수 있기 때문에 좋은 메타 태그를 설정하는 것은 검색 엔진 최적화에 큰 도움이 됩니다.

[로키토키] 홈페이지에서는 검색 엔진 최적화를 달성하기 위해 상기 코드처럼 메타 태그들을 설정해주었습니다. 추가적으로 컨텐츠 구조에 맞는 시맨틱 태그를 사용해 마크업하여 검색 엔진 최적화를 보완하고, 홈페이지의 접근성을 향상시켰습니다.
  

# 마치며
 
지금까지 Next.js를 간략하게 살펴보았습니다. 

이틀간 개발을 진행하면서 초기 세팅에 조금 애를 먹긴 했지만, Next.js 덕분에 최적화에 대한 부담을 덜어내고 빠르게 개발을 마무리할 수 있었습니다. 추후 지속적인 홈페이지 확장이 예상되는데요, 좋은 성능의 좋은 서비스를 지속적으로 제공하기 위해서 노력해야 하겠습니다.

  <p align="center" style="color:gray">
<img src="https://velog.velcdn.com/images/ksr20612/post/75723720-fb42-47f1-80d0-67ac5c99061b/image.png" width="50%" >
</p>
<br/>

Next.js v13에서는 tailwindcss의 사용을 권장하고 있어 [로키토키] 홈페이지에서도 이를 사용하였는데요, tailwindcss 관련해서도 추후에 글을 이어가 보도록 하겠습니다.

긴 글 읽어주셔서 감사합니다. 🤗
  
  
 
[👉로키토키 홈페이지 구경가기👈](https://rockie-talkie.com)
