---
layout: single
title: "Windows 10 환경에서 파이썬 패키지 배포 (Distribution of own python package in windows 10)"
author_profile: false
tag: 
- python
- windows
- programming-techniques
categories: 
- python
---

Curvelib 이라는 퀀트트레이딩 툴을 만들었는데, 사용하다보니 몇 가지 불편함이 느껴졌다. 

> (1) 워크스테이션 노트북에서 코드를 수정하여 github에 올리는데, 다른 컴퓨터에서 사용하려면 매번 github에서 다운 받아야 한다.    
> (2) 다른 사람이 사용하기에는 다운로드, 설치, 개발환경설정 등이 너무 복잡하다  
 
이 문제를 해결하려면 내가 만든 패키지를 PyPi에 오픈소스로 배포해야 했다.  
 
이번 기회를 통해서 파이썬 패키지 배포하는 법을 알게 되었다.  
혹시나 나중에 배포하게 될 일이 또 있을 것 같아 스스로 정리하는 차원에서 글을 써본다.  
 
참고로, 나의 개발환경은 Windows 10, Python 3.6.8 버전, Pycharm IDLE 이다. (아나콘다는 쓰지 않는다)  
배포하면서 다른 기술 블로그들을 참고하였는데, 대부분의 개발자 분들은 리눅스나 맥 환경에서의 구현을 보여주고 계셔서 약간의 어려움을 극복해야 했다.
(물론 그냥 내가 리눅스나 맥을 쓰면 해결될 문제이긴하다...)

 
## Step1. 패키지 만들기 ##    
 
Pycharm 환경에서 보통 개발할때 C:\Users\사용자명\PycharmProjects 에 프로젝트 폴더를 만들고 그 안에 가상환경을 구축하여 사용한다. 
가상 환경은 보통 프로젝트 폴더 안에 venv라는 폴더로 되어있다. 그림으로 보면 아래와 같다. 설명을 위해 'TestPack'이라는 패키지를 만들었다고 가정하겠다.

![image](https://user-images.githubusercontent.com/34860302/57270806-d40dfc80-70c7-11e9-86f5-86d4b123dc5a.png)  
 
##### (Figure 1) PycharmProjects 폴더 안에 패키지 폴더를 만듦 #####   
 
 
![image](https://user-images.githubusercontent.com/34860302/57270751-927d5180-70c7-11e9-8db2-65f8d0ad8e5d.png)  
 
##### (Figure 2) 패키지 폴더 안에 venv를 구축하여 사용함 #####   
 
 
이 상태에서 'TestPack'을 배포해 보겠다.
 
## Step2. 패키지 배포에 필요한 파일 작성 ##  
 
패키지 배포에 필요한 파일들을 작성해야한다.  
 
* __setup.py__  
* __setup.cfg__  
* __README.md__  
* __MANIFEST.in__  
 
작성은 패키지 폴더가 들어있는 루트 디렉토리인 PycharmProjects 폴더에 한다.  
 
![image](https://user-images.githubusercontent.com/34860302/57270860-facc3300-70c7-11e9-9f72-d400be634ac5.png)  
 
각 파일의 내용 작성법은 Reference에 참조한 다른 기술블로그들을 참조하는 것을 추천한다. (나는 각 항목들의 자세한 내막은 잘 모르기에...)  

위의 4개의 파일 중 반드시 필요한 것은 __setup.py__ 하나이다. 나머지는 필요하면 넣고 아니면 빼도 상관 없다.
 
#### (1) setup.py ####
 
TestPack 기준으로 작성한 __setup.py__ 내용이다

```
from setuptools import setup, find_packages

setup(
    name             = 'TestPack',
    version          = '1.0.0',
    description      = 'Test package for distribution',
    author           = 'Onew',
    author_email     = 'seunghwan0216@gmail.com',
    url              = 'https://github.com/onewquant/TestPack',
    download_url     = 'https://github.com/Onewquant/TestPack/archive/master.zip',
    install_requires = ['bs4','pandas<=0.22.0','urllib3'],
    packages         = find_packages(exclude = ['tests*']),
    keywords         = ['TestPack', 'testpack'],
    python_requires  = '>=3',
    package_data     =  {
        'TestPack' : [
            'testpack_configs.txt',
    ]},
    zip_safe=False,
    classifiers      = [
        'Programming Language :: Python :: 3.5',
        'Programming Language :: Python :: 3.6'
    ]
) 
```  
 
대부분의 항목들은 직관적으로 어떤 내용을 쓰면 될지 알 수 있다.  
헷갈리는 부분만 짚어보면,  
* __install_requires__ 는 TestPack 내에서 import해서 사용하는 다른 라이브러리 패키지를 써놓는 곳이다. 여기에 써 놓으면 나중에 pip를 통해 TestPack을 다른 컴퓨터에 설치할때, 써 놓은 패키지들은 자동으로 같이 설치가 된다.  
* __packages__ 는 만약 TestPack을 여러 개의 패키지 폴더 묶음으로 배포하고 싶다면 사용하는 옵션인데, 현재 여기서는 TestPack이 그런 상황은 아니므로 패스한다. 위에 써 놓은 내용은, 폴더 이름에 tests라는 문자가 포함된 폴더는 빼고 빌드 하겠다는 내용이다.  
* __package_data__ 는 패키지 안에 .py 파일이 아닌 다른 파일을 더 포함시키고 싶을때 쓰면 된다. TestPack 내에 일부러 testpack_configs.txt라는 파일을 포함시켜보도록 하겠다.
* __zip_safe__ 정확히 무슨 역할을 하는지는 잘 모르겠으나 package_data에 추가한 내용이 있으면 False로 설정해야한다고 한다...  
* __classifiers__ 파이썬 3.5와 3.6에서만 된다고 명시했다. (그냥 명시만 될 뿐 실제 에러가 따로 뜨거나 설치가 안되거나 하는 일은 없는 듯 하다.

## Reference ##    
(OS) Windows 10 Pro  
(Programming Language) Python 3.6.8   
(IDLE) Pycharm   
(Figure 1,2,3) 자체 제작   
(1) <https://code.tutsplus.com/ko/tutorials/how-to-write-your-own-python-packages--cms-26076>
(2) <https://rampart81.github.io/post/python_package_publish/>

  