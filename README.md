# YounLee.


### Jekyll 이용 GitHub 블로그 생성

> 사전준비 사항
1. Repository 생성 {git계정}.github.io <font color = "C62737"><b><sup>반드시 git계정명으로 생성해야 함</sup></b></font>
2. Ruby 설치 <b><sup>macOS 에는 이미 Ruby 가 설치되어 있음</sup></b>

<br>

> Jekyll 설치
```
$ sudo gem install jekyll
$ sudo gem install jekyll bundler
```

<br>

> Jekyll 로컬 테스트
```
$ bundle exec jekyll serve
```
- http://127.0.0.1:4000

![](https://lsy09.github.io/static/assets/img/landing/blogImg.png)

<br>

> Jekyll Themes 
1. [Themes](http://jekyllthemes.org/) 선택
2. Download
3. {git계정}.github.io 안에 압축 해제 
4. 자동으로 인식하기 때문에 새로고침 하면 변경된 Theme 확인 가능

<br>

> Theme 변경 후 에러 발생시 
```
$ gem install bundler
$ bundle install

$ bundle exec jekyll serve
```

### Jekyll Directory 구조

- _config.yml
- _layouts
- _post
    - 파일 이름의 형식은 날짜-제목 형식, 확장자는 .md로 작성
    - 예) 2017-04-08-자바스크립트-패턴-1주.md <br>
      '-' 의 경우 자동적으로 띄어쓰기로 변경 됩니다.
