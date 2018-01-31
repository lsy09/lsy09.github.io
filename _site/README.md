# YounLee.


### Jekyll 이용 GitHub 블로그 생성

> 사전준비 사항
1. Repository 생성 {git계정}.github.io <b><sup>반드시 git계정명으로 생성해야 함</sup></b>
2. Ruby 설치 <b><sup>macOS 에는 이미 Ruby 가 설치되어 있음</sup></b>

> Jekyll 설치
```
$ sudo gem install jekyll
$ sudo gem install jekyll bundler
```

> Jekyll 로컬 테스트
```
$ bundle exec jekyll serve
```
- http://127.0.0.1:4000

![](http://lsy09.github.io/static/assets/img/landing/blogImg.png)

> Jekyll Themes 
1. [Themes](http://jekyllthemes.org/) 선
2. Download
3. {git계정}.github.io 안에 압축 해제 
4. 자동으로 인식하기 때문에 새로고침 하면 변경된 Theme 확인 가능

> Theme 변경 후 에러 발생시 
```
$ gem install bundler
$ bundle install

$ bundle exec jekyll serve
```

