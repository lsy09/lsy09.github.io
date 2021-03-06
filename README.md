# YounLee.


### Jekyll 이용 GitHub 블로그 생성

> 사전준비 사항
1. Repository 생성 {git계정}.github.io  **<sup>반드시 git계정명으로 생성해야 함</sup>**
2. Ruby 설치  **<sup>macOS 에는 이미 Ruby 가 설치되어 있음</sup>**

<br>

> Jekyll 설치
```
$ sudo gem install jekyll
$ sudo gem install jekyll bundler

$ bundle install
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

### Jekyll Directory
- _config.yml   
    - 환경설정은 `YAML 형식`사용 **<sup>전역 환경설정</sup>**

- _includes
    - header, footer 와 같은 공통의 파일을 같은 포스트와 페이지의 테마 빌더 

- _post
    - 파일 이름의 형식은 날짜-제목 형식, 확장자는 .md로 작성
    - 예) 2018-01-31-Jekyll-이용-GitHub-블로그-생성.md <br>
      '-' 의 경우 자동적으로 띄어쓰기로 변경됨
      
- _site
    - 컴파일된 내용은 _site에 생성