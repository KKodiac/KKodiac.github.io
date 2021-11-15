---
layout: post
title: My Ubuntu Setup
date: 2021-11-16 00:00:00 +0900
description: 개인적으로 우분투 설정할 때 깔아주는 것들을 정리해 봤습니다. # Add post description (optional)
img: ubuntu.jpg # Add image post (optional)
tags: [Productivity, Ubuntu] # add tag
---

Windows를 데스크톱으로 사용하면서 개발 공부는 WSL을 활용했지만, 이런 저런 것들이 불편하기도 하고, Windows 로 사용하다가 게임에 한눈 판적이 한두번이 아니기에 아에 이번에 Ubuntu Distro 로 듀얼 부팅 세업으로 옮겼다. 

확실히 데스크톱을 껐다 키는 것이 귀찮아서 그런지 게임하는 것도 줄고 한눈 파는 일이 적어진 것 같은데, 이번 이렇게 옮긴 후 Ubuntu 설정하면서 깔아준 것들을 정리해보려고 한다. 

쓸모 없는 클러터가 깔아지는 것을 방지하기 위해서 minimal install 을 해서 깔끔하게 유지할 순 있지만 필요한 것들을 깔아야되서 귀찮다. 

그래서 혹시 모를 사태를 대비해서 미리 정리 해두려고 한다.


## ICloud Linux
 - [iCloud Snap Store](https://snapcraft.io/install/icloud-for-linux/ubuntu)
    - 맥북이랑 생각보다 괜찮게 된다

## Visual Studio Code
 - [VS Code](https://code.visualstudio.com/)
    - Snap 으로 깔아도 되지만 한글 지원이 안되는 오류가 있는걸로 알고 있어서 공홈(?) 에서 다운로드 받아서 `dpkg -i` 로 깔아주는게 낫다.

## Terminal Theme
 - [Dracula](https://draculatheme.com/terminal)

## Software for Studies
 - [Anaconda](https://www.anaconda.com/products/individual)
 - [Docker](https://docs.docker.com/engine/install/ubuntu/)
 - NASM
 - Golang
 - Android Studio
 - Flutter
 - MongoDB
 등등

## Oh-My-Bash
 - [oh-my-bash](https://github.com/ohmybash/oh-my-bash)
    - 개인적으로 `agnoster` 테마가 맞는거 같다.

## Vim
 - [Vimrc](https://github.com/KKodiac/vimrc/tree/master)

## Apt Packages
 - bat(batcat)
    - `cat` 명령어의 상위 호환
 - neofetch
    - 그냥 시스템 확인 용도 `lolcat` 이랑 사용하면 예쁘다 ㅎ
 - nvm
    - npm 버전 매니저, NodeJS 는 언젠가 공부할 거 같아서 미리 깔아뒀다.
 - dpkg
    - `wget`으로 다운로드 받은 파일들을 `dpkg` 로 설치하렸는데 minimal installation으로 하면 같이 안딸려온다
 - net-tools
    - `ifconfig` 쓰려고 설치, 다른 명령어는 모름
 - htop 
    - `top` 상위호환
 - vim
    - 그냥 `nano` 보다 많이 써서 터미널에서 편해짐
 - openssh-server
    - 맥북프로에서 `ssh` 접속 필요 시 