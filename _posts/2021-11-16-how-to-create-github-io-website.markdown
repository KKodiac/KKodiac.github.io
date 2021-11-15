---
layout: post
title: How to create github.io blog site
date: 2021-11-16 05:38:00 +0900
description: How to create a github blog # Add post description (optional)
img: github-pages.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Github Pages, How-To]
---
기존에 노션으로 정리 하기 시작했던 정보들을 github pages 를 통해 관리를 할 수 있는데, 예전에 한 템플릿을 대체로 새로 시작 해볼까 해서 다시 정리하려고 합니다. Github 계정만 있으면 자신만의 웹 사이트를 가지고 꾸밀 수 있다는게 상당히 끌리는 부분이라서 좋은 것 같아요.

## How to Create a Repository

Github 에서 저장소를 만드는 방법부터 소개하겠습니다. 

![Step One]({{site.baseurl}}/assets/img/how-to-create-a-repo.png)

우측에 초록색으로 `New` 라고 되어 있는 버튼을 클릭하면 새로운 저장소를 생성 할 수 있습니다. 

해당 버튼을 누르게 되면 아래처럼 생성하는 페이지가 나오게 됩니다. 

![Step Two]({{site.baseurl}}/assets/img/how-to-create-a-repo-2.png)

새로운 저장소의 이름을 작성해야 하는데, `{github 계정}.github.io` 으로 이름을 지어주어야 해당 url 이 생성 될 수 있습니다.

저는 이미 생성한 저장소가 있어서 불가 하다고 뜨지만 처음 진행하는 것이면 문제 없이 진행할 수 있습니다.


![Step Three]({{site.baseurl}}/assets/img/how-to-create-a-repo-3.png)

``` bash
git clone https://github.com/{github 계정}/{github 계정}.github.io.git
```

이렇게 생성된 리모트 저장소를 같이 로컬에 클론 해주면 됩니다. 


## How to choose a template

저는 구글링으로 `https://jekyllthemes.io/github-pages-themes` 사이트에 있는 무료 템플릿을 사용했습니다.
마음에 드는 템플릿을 선택한 후 해당 템플릿의 깃허브 저장소를 포크한 후 로컬에 클론하면 됩니다. 

![Step Four]({{site.baseurl}}/assets/img/fork-template.png)

우측 상단에 있는 `Fork` 버튼을 누르고 자신의 깃허브로 옮긴 후,


``` bash
git clone https://github.com/{github 계정}/{템플릿 저장소 이름}.git
```

처럼 로컬에 저장해 줍니다.

그 후 해당 요소들을 이전 로컬에 저장했던 `{github 계정}.github.io` 에 옮기면 됩니다.

```
cd {템플릿 저장소 이름} 
cp -r * ../{github 계정}.github.io
cd ../{github 계정}.github.io
```

이렇게 모든 요소들을 옮긴 후 `ls` 명령어로 확인을 하면 

![ls result]({{site.baseurl}}/assets/img/ls-result.png)

템플릿 활용에 필요한 파일들이 github.io 디렉토리로 이동한 것을 확인할 수 있습니다.

## How to Build the Template

이제 해당 내용을 Github 에 푸쉬하고 url 이 활성화 된 것을 확인하면 됩니다.

템플릿들은 이제 Ruby 언어로 개발된 Jekyll 이라는 정적 사이트 생성기로 빌드를 해야됩니다. 

따라서 Ruby 및 추가적인 리소스가 필요합니다. 

[이 링크](https://jekyllrb.com/docs/installation/#requirements)에 있는 스텝을 따라서 사용하는 운영체제에 따라 Ruby 와 필요한 Dependency 를 설치 후,

[이 링크](https://jekyllrb.com/docs/) 에 나와 있는대로 빌드를 하면 됩니다.

빌드가 성공한 것을 확인하면 리모트에 푸쉬 해줍니다. 

```
git add *
git commit -m "Adding Template to github.io"
git push -u origin master
```

`https://{github 계정}.github.io` 로 들어가서 확인하면 내용이 반영된 것을 확인 할 수 있습니다.\

## How to Update Your Blog

링크까지 작동하는 것을 확인 했으면, 이제 내용을 추가할 스텝입니다.

다시 로컬로 돌아와서 `_config.yml` 파일을 확인해 줍니다. 

해당 파일이 jekyll 에서 빌드 시 필요한 정보를 얻는 곳 입니다. 
[이 링크](https://jekyllrb.com/docs/configuration/) 를 참고하면 자세한 사항을 알 수 있습니다.


![config file]({{site.baseurl}}/assets/img/config-file.png)

여기서 참고 할 내용은 `title`, `baseurl` 입니다. 

`title`은 해당 웹페이지의 타이틀의 내용을 수정할 수 있는 태그 입니다.

`baseurl`은 해당 웹페이지의 리소스가 로딩될 시 Github Page 가 참조하는 url 입니다. `""` 로 두면 됩니다.

새로운 블로그 포스트는 디렉토리 중 `_posts` 디렉토리에 마크다운 파일을 생성하여 추가 해주면 됩니다. 

만약 이미지 파일 등 마크다운 파일에서 참조해야 할 리소스가 있는 경우 `assets` 디렉토리 안에 `img` 디렉토리 안에 이미지를 삽입해서 사용하면 됩니다. 


