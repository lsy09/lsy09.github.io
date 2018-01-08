---
layout: post
title:  "JSON Fragment Edit 기능"
date:   2017-12-26
excerpt: ""
tag:
- Intellij
- JSON
---
    


## Intellij - JSON Fragment Edit 기능


#### Intellij 사용시 JSON을 편집 할 수 있는 **JSON Fragment Edit 기능** 제공 

1. JSON문자열을 넣을 String변수 선언. 

![스크린샷 2017-12-06 오후 4.12.47](https://i.imgur.com/wk0vezt.png)<br> 

2. 변수의 값이 되는 **""** 사이에 커서를 두고, **alt+enter**를 누르면 **Inject language or reference 선택** 시 아래와 같이 선택 리스트 생성 됨. 

![스크린샷 2017-12-06 오후 4.13.03](https://i.imgur.com/sG7P3fA.png)<br> 

3. JSON 선택 시 아래와 같은 팝업 생성됨, 한번 더 **alt+enter**. 

![스크린샷 2017-12-06 오후 5.12.57](https://i.imgur.com/enPlkQb.png)<br> 

4. **//language=JSON** 라는 주석이 생성. 

![스크린샷 2017-12-06 오후 5.27.01](https://i.imgur.com/eFvy2OL.png)<br> 

5. 다시 **""** 사이에 커서를 두고, **alt+enter**를 누르면 메뉴가 나오는데 이때 **Edit JSON Fragment** 선택. 

![스크린샷 2017-12-06 오후 5.28.04](https://i.imgur.com/bAuqTdp.png)<br> 

6. 아래 이미지와 같은 JSON편집 화면 생성 됨. 

![스크린샷 2017-12-06 오후 4.47.18](https://i.imgur.com/6xJcaHo.png)<br> 

7. Fragment 편집 화면에서 JSON을 직접 편집하면 편집한 내용이 자동으로 변수에 값 변환이 되어서 적용 됨. 

![스크린샷 2017-12-06 오후 4.46.07](https://i.imgur.com/GE7HdRF.png)<br> 

---

- Inject Ianguage or reference 가 보이지 않을 때 

 ![스크린샷 2017-12-06 오후 5.42.30](https://i.imgur.com/z7AdPAj.png)<br> 

1. command+enter -> Editor -> Intentions 선택. 

![스크린샷 2017-12-06 오후 4.48.35](https://i.imgur.com/eMCqj1S.png)<br> 

2. Language Injection 리스트 내에 있는 Inject language or reference 클릭 저장. 

![스크린샷 2017-12-06 오후 5.43.28](https://i.imgur.com/Z76ivKi.png)<br> 

3. Inject language or reference 선택 팝업 정상 생성. 

![스크린샷 2017-12-06 오후 4.12.56](https://i.imgur.com/4VFVHhp.png)<br>
