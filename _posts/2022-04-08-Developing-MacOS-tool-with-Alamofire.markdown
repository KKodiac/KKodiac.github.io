---
layout: post
title: Developing a MacOS tool with Alamofire
date: 2022-04-08 05:38:00 +0900
description: Alamofire 라이브러리를 활용해서 노래 가사 보여주는 CLI 도구 만들기 # Add post description (optional)
img: alamofire.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Swift, MacOS, Popover, Alamofire]
---

# 개요

전에 파이썬으로 만들었던 노래 가사를 보여주는 [CLI 도구](https://github.com/KKodiac/lyricli)를 활용할 수 있는 무언가를 만들기 위해 생각해 보았다. 파이썬으로는 `requests`와 `beautifulsoup4` 라이브러리를 사용해서 정말 쉽게 멜론을 크롤링하고 파싱해서 만들 수 있었다. Swift 언어를 공부해가면서 iOS 기반인 어플리케이션 말고 다른 일반적인 것도 만들어 보고 싶다는 생각에 진행하게 되었다. 그래서 MacOS의 메뉴바에서 현재 재생 중인 노래의 가사를 볼 수 있는 어플리케이션을 만들어 보기로 했다.


## Table of Contents
 - [Prerequisites](#Prerequisites)
 - [진행 중 막힌 부분](#Challenges-During-Project)
 - [Reference](#referece)
 - [마치며](#마치며)
 

## Prerequisites
[Lyricli](https://github.com/KKodiac/lyricli)의 API 버전인 [Lyricapi](https://github.com/KKodiac/lyricli/lyricapi)가 실행 중이어야 한다. 추가적인 내용은 해당 링크의 README.md 를 참고하면 된다.

또한 Swift의 HTTP Networking library 인 Alamofire 라이브러리를  Swift Package Manager에 추가해 줘야한다. 

### Installing Alamofire

Swift 에서는 크게 세 가지의 패키지 매니저가 존재한다. `Carthage`, `Cocoapods`, 그리고 애플에서 발표한 `Swift Package Manager(SPM)`이 세가지다. 해당 패키지 매니저들의 차이점은 다음 시간에 알아보기로 하고, [Alamofire](https://github.com/Alamofire/Alamofire) 깃허브 저장소에서 SPM으로 패키지를 프로젝트에 추가해주기로 한다.


## Challenges-During-Project

진행하면서 막혔던 부분은 크게 네 가지가 있었다.
- 현재 재생 중인 음악을 어떻게 가져와야 하지?
- 메뉴 바를 어떻게 생성을 해야하는 거지?
- 기본적으로 Alamofire을 어떻게 사용해야 하는 거지?
- Popover을 생성하면 화살표 모양의 Anchor가 생성되는데 이걸 어떻게 지우고, Human Interface Guideline에 위배 되지는 않나?

> ### 현재 재생 중인 음악을 어떻게 가져와야 하지?
우선 가장 첫 번째 였던 재생 중인 음악 정보를 가져오는 것은 스택 오버플로를 검색 하던 중 그대로 정답이 있었다. 아래에 링크를 정리해 놓았으니 참고하자. 

> ### 메뉴 바를 어떻게 생성을 해야하는 거지?
두번째 메뉴바는 구글링 도중 `NSPopover`을 사용한 예시를 보았다. 그대로 따라 해보던 도중, `AppDelegate` 을 추가해야 됐는데, 이것을 기반으로  어플리케이션 라이프사이클을 만들었어야 하는 예시였다. 하지만 일반적인 `SwiftUI` 만을 사용해 보았기에, 생성한 `AppDelegate` 클라스를 기존 앱에 추가를 어떻게 해야하는 지 몰랐다. 해당 솔루션은:

```swift
@NSApplicationDelegateAdaptor(AppDelegate.self) var delegate
```

을 struct App 에 추가해 주면 된다. 

```swift
@main
struct lyricsbarApp: App {
    @NSApplicationDelegateAdaptor(AppDelegate.self) var delegate
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

> ### 기본적으로 Alamofire을 어떻게 사용해야 하는 거지?
Alamofire 공식 [문서](https://alamofire.github.io/Alamofire)를 참고했다. 

> ### Popover을 생성하면 화살표 모양의 Anchor가 생성되는데 이걸 어떻게 지우고, Human Interface Guideline에 위배 되지는 않나?
해당 의문점에 대해서 똑같은 질문을 스택오버플로에서 찾을 수 있었다. 링크는 아래 [참조](#reference) 에 걸어 놓았다.


## 마치며
강의를 따라하면서 진행하는 것도 배울 점이 많지만, 내가 원하는 어플리케이션을 구상하면서 제작하며 직접 몸으로 느껴가면서 배우는 것도 유의미한 것 같다. 아직 일부분만 제작이 완료 되었지만, 꾸준히 기능을 추가하고 싶다. 해당 프로젝트의 깃허브는 [링크](https://github.com/KKodiac/menu_bar_lyrics) 를 참고하면 된다.

## Reference 
[Alamofire](https://github.com/Alamofire/Alamofire)

[Popover의 Ancho Arrow 없애는 방법](https://stackoverflow.com/questions/4801860/can-i-remove-the-arrow-in-the-popover-view)

[맥OS에서 현재 플레이 중인 노래 정보 얻는 방법](https://stackoverflow.com/questions/61003379/how-to-get-currently-playing-song-on-mac-swift)