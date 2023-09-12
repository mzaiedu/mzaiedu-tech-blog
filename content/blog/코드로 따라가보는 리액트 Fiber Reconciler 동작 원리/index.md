---
title: 코드로 따라가보는 리액트 Fiber Reconciler 동작 원리 
date: "2023-09-13T08:00:00.121Z"
---

**답답해서 facebook/React를 열어보았습니다.**
by Dongyeon

렌더링 단계에서 만들어지는 리액트 엘리먼트(요소)들, 이전 스냅샷(native DOM)과 파이버 트리를 비교하는 Reconciling... 두리뭉실하게 잡힌 개념, 이참에 리액트 코드를 따라가며 다시 ***Fiber Reconciler***를 정리해보았습니다.

***👇 오늘 해체할 코드 👇***
[🔗 facebook/react](https://github.com/facebook/react)


## 💁 리액트의 렌더링 세 단계

React는 다음의 세 단계를 거쳐 브라우저에 새로운 DOM 엘리먼트를 그립니다.

***1. Render***
&nbsp; &nbsp; &nbsp; 트랜스파일된 jsx(createElement())를 통해 리액트 요소를 생성합니다.
***2. Reconcile***
&nbsp; &nbsp; &nbsp; 이전 스냅샷과 새로 렌더링할 트리를 비교하여 변경점을 확인합니다.
***3. Commit***
&nbsp; &nbsp; &nbsp; 새로 렌더링할 트리를 브라우저에 그립니다.

> 💥 ***토막 기록***
Render 단계에 대해서 잘못 알고 있던 필자가 잘못 알고 있던 한 가지는 Render 단게에서 모든 리액트 요소의 ``render``함수가 트리의 끝까지 재귀적으로 호출된다는 것이었습니다. Render 단계에서는 최초 지점(root)의 리액트 요소만 생성하며, 리액트 요소를 파이버로 변환하고 렌더 함수를 실행시키는 것은 ***Reconcile*** 단계입니다.

리액트 요소는 ```type```, ```key```, ```ref```, ```props```로 구성된 일반 객체이며, ```props```에는 ```children```이 담겨 있어 ***트리 구조***의 형태를 가집니다. 좁은 의미의 렌더 단계(일반적으로 말하는 '렌더링'과는 달리 리액트의 Reconcile 직전 단계)에서는 리액트 요소를 생성하는 것으로 단계를 종료됩니다.

## 📱 Reconcile 알고리즘으로써 파이버

두번째 Reconcile 단계에 진입하면, 최초 리액트 요소의 ```children```부터 탐색을 시작하여 파이버 노드를 생성합니다. 이어 파이버 노드를 통해 해당 컴포넌트의 렌더 함수를 호출하며 컴포넌트 인스턴스가 생성됩니다. 차례차례 다음 파이버 노드를 생성하면서 새로운 파이버 노드 트리를 생성하는 것이죠. 생성한 노드를 이전의 스냅샷과 비교하는 과정도 자연스럽게 포함됩니다. 코드를 따라 Fiber Reconciler의 내부 동작을 이해해보도록 하겠습니다. 

> 💥 ***토막 기록***
사실 공식적으로 렌더링 단계같은 건 없어요. 다만 이해하기 편하게 공통적인 로직을 묶어놓은 것 뿐이랍니다.


### 1. PRE-Reconcile: 최초 루트 파이버를 생성해요

```javascript
const root = ReactDOM.createRoot(document.getElementById('root') as HTMLElement);
root.render(<>...</>);
```

```createRoot()```를 호출하는 것으로 렌더링이 시작됩니다. ```createRoot()```는 내부적으로 ```createFiberRoot()```를 호출하는데요, ```createFiberRoot```는 다시 ```createHostRootFiber```를 호출해 최초의 호스트 루트를 생성합니다. 

```javascript
// ReactFiberReconciler.js
export function createContainer(
	...
): OpaqueRoot {
	...
  return createFiberRoot(
    containerInfo,
    tag,
    hydrate,
    initialChildren,
    hydrationCallbacks,
    isStrictMode,
    concurrentUpdatesByDefaultOverride,
    identifierPrefix,
    onRecoverableError,
    transitionCallbacks,
  );
}

// ReactFiberRoot.js
export function createFiberRoot(
	...
): FiberRoot {
  // 파이버 루트 생성
  const root: FiberRoot = (new FiberRootNode(
	...
  ): any);
	...
  // 호스트 루트 파이버 생성
  const uninitializedFiber = createHostRootFiber(
    tag,
    isStrictMode,
    concurrentUpdatesByDefaultOverride,
  );
  
  // 서로를 참조(순환참조)할 수 있도록 연결해줍니다.
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;

	...

  return root;
}
```

여기서 ```root```는 정적인 상태의 노드를, ```uninitiazliedFiber```는 서비스의 다양한 상태들이 변경되며 화면이 변하는 것을 파악하기 위해 사용되는 파이버 노드입니다. Reconcile 작업에서는 호스트 루트인 ```uninitializedFiber```를 사용하게 됩니다. 
즉, 호스트 루트는 Reconciler가 비교하는 두 스냅샷 중 한 축을 맡게 됩니다(이전의 화면 혹은 변경될 화면).


### 2. workLooping: 현재 처리 중인 노드(workInProgress) 무한 반복해요

```javascript
// ReactFiberWorkLoop.js
function workLoopConcurrent() {
  // workInProgress(작업 대상 파이버)가 있는 경우 작업에 착수
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

모든 Reconcile 작업은 ```workLoopConcurrent(sync)``` while문에서 처리됩니다. 작업할 Fiber가 있는 경우 ```performUnitOfWork()```를 호출하여 작업에 착수하는 것이죠. ```workInProgress``` 가 null만 아닌 경우에는 다음 파어버(작업 단위)를 순서대로 작업하는 것입니다.

```javascript
// ReactFiberWorkLoop.js
function performUnitOfWork(unitOfWork: Fiber): void {

  // 기존 노드에 대응되는 alternate가 작업 대상
  const current = unitOfWork.alternate;
  ...

  let next;
  if (enableProfilerTimer && (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);
    // 작업 시작!
    next = beginWork(current, unitOfWork, renderLanes);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork(current, unitOfWork, renderLanes);
  }

  ...
  
  if (next === null) {
    // 다음 작업이 없으면 작업 종료
    completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }

  ReactCurrentOwner.current = null;
}
```

호출된 ```performOfUnitFiber```를 살펴보면 크게 ```beginWork```와 ```completeUnitOfWork```로 구성되어 있습니다. 말 그대로 Reconcile 작업을 본격적으로 시작, 종료하는 것입니다. 

시작에 앞서 ```const current = unitOfWork.alternate;```를 통해 현재 작업할 노드를 설정해줍니다. 즉, 이전 스냅샷의 노드(unitOfWork)에 대응되는 새로운 노드를 새로운 작업 대상(current)으로 설정하는 것입니다. 앞으로의 작업은 모두 ```current```로 부터 시작됩니다.

> 💥 ***토막 기록***
위에서 ```HostRoot```가 Reconcile 비교 작업의 한 축을 담당한다고 했던 의미가 여기에 있어요. ```alternate```를 통해 가져온 나머지 한 축을 탐색하면서 Reconcile 작업을 완료하고, 나머지 한 축에 변경된 사항을 반영해요. 두 축이 각각 native DOM(이전 스냅샷)과 React Dom(변경될 스냅샷)을 대표하는 것이죠.


### 3. beginWork(): 파이버 노드 생성을 시작해요

```javascript
// ReactFiberBeginWork.js
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
	...
  switch (workInProgress.tag) {
    case IndeterminateComponent: {
      // 컴포넌트 렌더링(Component()를 호출하여 리액트 요소를 전달받아요)
      return mountIndeterminateComponent(
        current,
        workInProgress,
        workInProgress.type,
        renderLanes,
      );
    }
	...
    case FunctionComponent: {
		...
      return updateFunctionComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderLanes,
      );
    }
		...
    case HostRoot:
      return updateHostRoot(current, workInProgress, renderLanes);
	...
    case Fragment:
      return updateFragment(current, workInProgress, renderLanes);
    ...
  }

}
```

```workInProgress```의 ```tag```값에 따라 reconcile 상세 작업이 달라지게 됩니다. 여기서 ```tag```는 생성할 컴포넌트의 인스턴스의 유형을 의미합니다. ```switch```문 안쪽에서 명시된 것처럼 ```FunctionComponent```, ```HostRoot```, ```Fragment``` 등의 값을 가질 수 있어요.

최초의 Reconcile일 경우에는 `workInProgress`는 ```HostRoot```이므로 `updateHostRoot()`를 호출합니다. 호출되는 함수는 서로 다르지만, 궁극적으로는 파이버 노드를 생성하고 비교점을 업데이트(파이버의 effect)한다는 점에서는 동일합니다.


### 4. reconcileChildren(): 자식 노드를 타고 들어가 비교를 시작해요

자, 그럼 먼저 `updateHostRoot()`를 타고 들어가보겠습니다.

```javascript
// ReactFiberBeginWork.js
function updateHostRoot(
  current: null | Fiber,
  workInProgress: Fiber,
  renderLanes: Lanes,
) {
	...
  } else {
    resetHydrationState();
    // 이전 파이버와 다음 파이버가 같다면(변경점이 없다면), 더이상 비교 안함
    if (nextChildren === prevChildren) {
      return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
    }
    // 변경이 발생했다면, 비교를 시작
    reconcileChildren(current, workInProgress, nextChildren, renderLanes);
  }
  return workInProgress.child;
}
```

생략되었지만 `updateHostRoot`의 초반부에서 다음 작업 단위(파이버 노드)를 `nextChildren` 에 할당합니다. 해당 값을 이전 파이버 노드와 ***단순 객체 비교***하여 같다면 더이상 작업을 수행하지 않고, 결과를 반환합니다. 변경이 발생했다면, ```reconcileChildren()``` 를 호출합니다.

```javascript
// ReactFiberBeginWork.js
function updateFragment(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
) {
  const nextChildren = workInProgress.pendingProps;
  // 여기서도 reconcileChildren 진입!
  reconcileChildren(current, workInProgress, nextChildren, renderLanes);
  return workInProgress.child;
}
```

`switch`문의 다른 함수들도 결국 내부적으로는 `reconcileChildren()`을 호출합니다. 

```javascript
// ReactFiberBeginWork.js
export function reconcileChildren(
	...
) {
  if(current === null) {
    ...
  } else {
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderLanes,
    );
  }
}
```

`reconcileChildren()`은 다시 `reconcileChildFibers()`를, `reconcileChildFibers()`는 `createFiberFromElement()`를 호출하는데요, 최종적으로 `createFiberFromElement()`에서 컴포넌트의 파이버 노드를 생성합니다.


### 5. createFiberFromFragment(): 파이버 노드를 생성해요

```javascript
// ReactFiber.js
export function createFiberFromFragment(
  elements: ReactFragment,
  mode: TypeOfMode,
  lanes: Lanes,
  key: null | string,
): Fiber {
  const fiber = createFiber(Fragment, elements, key, mode);
  fiber.lanes = lanes;
  return fiber;
}
```

드디어 `workInProgress`(작업 중인 파이버)의 ***첫번째 child***에 대한 <span style="background: #C0FFFF; color: black">🎊***파이버 노드가 생성***🎊</span>되었습니다. 생성한 파이버 노드를 리턴합니다.

`reconcileChildren()`과 `updateFragment()` 내부 로직을 기억하시나요(바로 위에 있어요)? `reconcileChildren()` 내부에서 `workInProgress.child`에 막 생성한 파이버 노드를 할당했죠. `updateFrament()`에서는 이 `workInProgress.child`를 리턴했구요. 리턴된, 새로 생성된 자식 파이버 노드는 결국 어디에 도달하게 될까요? 

> 💥 ***토막 기록***
우리가 짠 ```컴포넌트()``` 는 언제 호출될까요? 언제 호출되어 React 요소(엘리먼트)로 변경될까요? 맨 처음 루트 컴포넌트가 React 요소로 변경되어야 줄줄이 파이버 노드를 생성할 수 있을텐데 말이죠... 이 작업은 ```renderWithHooks()```가 담당합니다. ```beginWork()```의 스위치문 안쪽에 있는 ```mountIndeterminateComponent()```에서 ```renderWithHooks()```를 호출해요.


### 6. completeUnitOfWork(): 다음 노드가 없으면 다시 역으로 올라와요

```javascript
function performUnitOfWork(unitOfWork: Fiber): void {

  // 기존 노드에 대응되는 alternate가 작업 대상
  const current = unitOfWork.alternate;
  ...

  let next;
  if (enableProfilerTimer && (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);
    // 작업 시작!
    next = beginWork(current, unitOfWork, renderLanes);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork(current, unitOfWork, renderLanes);
  }

  ...
  
  if (next === null) {
    // 다음 작업이 없으면 작업 종료
    completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }

  ReactCurrentOwner.current = null;
}
```

`performUnitOfWork()`를 다시 살펴보겠습니다. ```beginWork()```의 결과로 다음 작업할 파이버 노드가 ```next``` 변수에 할당되었습니다. 다음 작업할 파이버 노드가 null, 즉 없다면? ```completeUnitOfWork()```로 진입합니다. 다음 작업할 파이버 노드가 뭐였냐구요? workInProgress 파이버의 child이자, 첫 번째 자식 파이버 노드였죠!

```javascript
// ReactFiberWorkLoop.js
function completeUnitOfWork(unitOfWork: Fiber): void {
	...
      next = completeWork(current, completedWork, renderLanes);
    } else {
     	...
    }
		...
    if (next !== null) {
      // completeWork 했는데 다음 작업이 생겨버린 경우엔 다시 작업
      workInProgress = next;
      return;
    }
	
    // 형제 노드가 있는 경우엔 다음 형제 노드가 작업 단위가 됨
    const siblingFiber = completedWork.sibling;
    if (siblingFiber !== null) {
      // If there is more work to do in this returnFiber, do that next.
      workInProgress = siblingFiber;
      return;
    }
    // Otherwise, return to the parent
    // $FlowFixMe[incompatible-type] we bail out when we get a null
    completedWork = returnFiber;
    // Update the next thing we're working on in case something throws.
    workInProgress = completedWork;
  } while (completedWork !== null);

  // 루트까지 다 타고 올라갔다면 끝!
  if (workInProgressRootExitStatus === RootInProgress) {
    workInProgressRootExitStatus = RootCompleted;
  }
}
```

`completeUnitOfWork()`는 내부적으로 `completeWork()`를 호출합니다. 만약 `completeWork()`에서 다음 작업이 생긴다면? `workInProgress`에 다음 작업(파이버)을 할당해서 ```workLoop```가 다시 작업을 처리하도록 합니다. 

***첫번째 child를 파이버 노드로 만드는 것이 이상했다구요?*** 형제 노드가 있는 경우에는 형제 노드를 `workInProgress`에 할당하여 다시 `workLoop`가 돌아가게 처리합니다. 역시나 다 계획이 있던 거죠.

![너는 계획이 다 있구나](https://velog.velcdn.com/images/ksr20612/post/99c2b2c9-36e3-4a86-8b23-0dbb4e66b441/image.png)


### 7. completeWork(): 파이버 노드로부터 DOM 인스턴스를 생성(업데이트)해요

`completeWork()` 내부를 살펴보겠습니다. `completeWork()`는 너무 길어서 필요한 부분만 적가져오겠습니다(지금까지도 그랬으면서!)

```javascript
// ReactFiberCompleteWork.js
function completeWork() {
  ...
  // 이미 하이드레이트되었다면, 즉 이미 렌더링 되었다면 업데이트만
  const wasHydrated = popHydrationState(workInProgress);
  if (wasHydrated) {
    // TODO: Move this and createInstance step into the beginPhase
    // to consolidate.
    if (
      prepareToHydrateHostInstance(workInProgress, currentHostContext)
    ) {
      markUpdate(workInProgress);
    }
  } else {
    const rootContainerInstance = getRootHostContainer();
    // 첫 렌더링이라면 DOM 요소 생성
    const instance = createInstance(
      type,
      newProps,
      rootContainerInstance,
      currentHostContext,
      workInProgress,
    );
    appendAllChildren(instance, workInProgress, false, false);
    workInProgress.stateNode = instance;
  ...
}
```

이미 렌더링되었다면 단순히 DOM 인스턴스를 업데이트합니다(`markUpdate()`). 첫 렌더링이라면? `createInstance()`를 통해 DOM 인스턴스를 생성합니다. 

> 💥 ***토막 기록***
개발진이 적어놓은 TODO 주석이 보이네요. 코드를 하나하나 들여다보면서 많은 TODO를 발견했습니다. 개발진 대단해요 정말.

### 8. commitWork(): 루트까지 거슬러 올라왔으면 커밋해요(그려줘요)

루트까지 업데이트가 끝났다면, `finishConcurrentRender()`에 의해 `commitWork()`가 실행됩니다. `ReactFiberRootScheduler`가 루트까지 업데이트 되었는지 모니터링하기 때문에 커밋 시점을 알 수 있습니다. `finishConcurrentRender()`를 호출하는 `performConcurrentWorkOnRoot()`를 스케줄러가 관리하거든요.

```javascript
export function performConcurrentWorkOnRoot(
  root: FiberRoot,
  didTimeout: boolean,
): RenderTaskFn | null {
  ...
  commitRoot(
    root,
    workInProgressRootRecoverableErrors,
    workInProgressTransitions,
  );
  ...
}
```

`HostRoot`와 `HostRoot.alternate`가 비교의 두 축을 담당한다고 했던 말, 생각나시나요? (***##2 [토막 기록] 참고***) 커밋 작업을 수행하면서 `HostRoot.alternate`에 변경된 dom인스턴스가 커밋되고, 이제는 이 트리가 이전 스냅샷이 됩니다(native DOM에 반영된 것과 동일한 트리를 가지고 있음). 다음 변경이 발생하면 `HostRoot`에서 Reconcile 단계를 시작합니다. `root.current`를 `HostRoot.alternate`로 바꿔주기 때문에 가능합니다! `root`에 `hostRoot`까지 생성했던 이유, ***재활용***하기 위함이었네요.
 
## 🌈 파이버 노드 덕분에 증분 렌더링

> 💥 ***토막 기록***
파이버는 재조정 알고리즘을 가리키기도, 렌더링 작업 단위(가상 렌더링 스택 위의)를 가라키기도 해요. 다양한 개념을 의미하기 때문에 문맥을 통해 그 의미를 잘 파악해야 해요. 필자는 아키텍처(엔진)으로서의 파이버를 ```Fiber reconciler```, 렌더링 작업 단위로서의 파이버를 ```Fiber Node``` 로 부르고 있어요.

```Reconcile```단계를 통해 파이버 노드를 생성하고, 파이버 노드를 통해 이전/이후의 스냅샷을 비교합니다. 그렇다면 파이버 노드는 과연 무엇일까요? 파이버 노드가 무엇이길래 렌더링의 작업 단위가 되고, 변경 사항을 비교할 수 있는 걸까요? 파이버 노드의 구조를 간략하게 살펴보겠습니다.

[ 로키토키 ]의 아무 Dom 요소를 찍어 ```__reactFiber```로 시작하는 속성을 찍어보면 파이버 노드의 구조를 쉽게 확인할 수 있습니다(리액트로 만든 서비스라면 다 가능해요).

![파이버 구조 샘플](https://velog.velcdn.com/images/ksr20612/post/f7f4b143-9590-48e7-9b92-67e178a9b94c/image.png)


파이버 노드는 많은 속성들을 가지고 있지만, 기능 별로 묶어 크게 정리하면 다음 세 요소로 구분될 수 있습니다.

1. 리액트 컴포넌트와 관련된 요소
2. 작업 단위 및 트리와 관련된 요소
3. 변경 사항과 관련된 요소

```javascript
// 중요한 속성만 정리한 파이버
{
  tag, key, type, // 생성할 리액트 컴포넌트와 관련된 요소
  return, child, sibling, // 작업 단위 및 트리와 관련된 요소 
  nextEffect, firstEffect, lastEffect // 변경 사항과 관련된 요소(정보)
}
```

파이버 노드가 위 세 가지 정보를 모두 가지고 있기 때문에 렌더링의 작업 단위로서 기능할 수 있습니다. 즉, 생성할 컴포넌트의 인풋과 형태, 다음 작업에 대한 정보, 작업을 차례차례 호출하면서 변경된 사항들에 대한 정보를 파이버 노드가 ***홀로*** 들고 있기 때문에 파이버 노드는 그 자체로 완벽한 최소 작업 단위라고 할 수 있습니다.

이러한 구조의 파이버가 갖는 함의점은 무엇일까요? 바로 <span style="background: #C0FFFF; color: black">***증분 렌더링***</span>이 가능하다는 것입니다.

증분 렌더링이란 동시성있는 렌더링 작업을 의미하는데요, 쉽게 말하면 분할된 렌더링 작업 단위에 우선 순위를 매겨 순서대로 처리하며, 우선도가 높은 작업 단위가 갑자기 끼어들었을 때 진행 중이던 작업을 중지하고 먼저 그 작업을 처리하고 돌아올 수 있는 유연한 형태의 렌더링을 의미합니다. 

즉, 파이버는 <span style="background: #E6E6FA; color: black">과거(변경 사항과 관련된 정보), 현재(리액트 컴포넌트와 관련된 요소), 미래(다음 작업 단위 및 트리와 관련된 요소)에 대한 정보를 모두 들고 있기 때문</span>에 증분 렌더링을 실현케 하는 중요한 개념이라고 할 수 있겠습니다. 


## 🤩 마무리

리액트 코드를 따라가며 React의 Fiber Reconcile 아키텍처를 살펴보았습니다. 코드를 해석하며 다시금 훌륭한 개발자분들께 경의를 표합니다. 

![rEaCt!](https://velog.velcdn.com/images/ksr20612/post/d3718018-97e1-424d-9076-ab4910c74294/image.png)



<br/> <br/>

---
🙇 도움이 된 글들
- [React 파이버 아키텍처 분석](https://d2.naver.com/helloworld/2690975)
- [React Fiber Architecture](https://immigration9.github.io/react/2021/05/29/react-fiber-architecture.html)
- [Virtual DOM과 Internals](https://ko.legacy.reactjs.org/docs/faq-internals.html)
- [React Fiber 얕게 알아보기](https://velog.io/@jangws/React-Fiber)