---
layout: post
title: Foaas for IOS
date: 2022-04-25 05:38:00 +0900
description: Alamofire 라이브러리를 활용해서 간단한 IOS 앱 만들기 # Add post description (optional)
img: iOS.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Swift, Alamofire, IOS]
---

# 개요

Github 에 있는 [awesome-apis](https://github.com/toddmotto/public-apis) 저장소를 둘러보던 중 FOAAS(Fuck off as a service) 라는 오픈 API를 보게 되었다. 시험기간에 전공 과목은 아에 듣지 않고 있어서 iOS 공부를 겸하면서 간단하게 프로젝트를 진행하고 싶던 차에 Auth 및 SSL이 딱히 없는 좋은 실험 대상을 알게 되어서 iOS 어플로 해당 API 를 제공해보기로 했다. [한글 버전](https://github.com/KKodiac/foaas-kr)도 기존 레포를 포크해서 심심할 때 마다 내용을 추가해 줄 생각이다. 일단 [기본 틀](https://foaas-kr.herokuapp.com)만 헤로쿠에 배포 해놓았다.


## Table of Contents
 - [진행 중 막힌 부분](#Challenges-During-Project)
 - [Reference](#referece)
 - [마치며](#마치며)
 

![foaas-demo]({{site.baseurl}}/assets/img/demo_foaas_ios.jpeg)


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