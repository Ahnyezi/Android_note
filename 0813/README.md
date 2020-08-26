# playdata

8/13 (목)
# 1. adapter

![image](https://user-images.githubusercontent.com/62331803/90098108-75c3d500-dd72-11ea-96cf-1323fec97d4f.png)
![image](https://user-images.githubusercontent.com/62331803/90098128-7fe5d380-dd72-11ea-941c-bd58b2d3f324.png)

- 예제
![image](https://user-images.githubusercontent.com/62331803/90098152-8bd19580-dd72-11ea-830a-f325dc0117f4.png)
![image](https://user-images.githubusercontent.com/62331803/90098162-8ecc8600-dd72-11ea-94ce-61fef7477f69.png)

- 연습문제: 전화번호부
*내부 model 패키지에 member(dto) class 생성
![image](https://user-images.githubusercontent.com/62331803/90098192-a0ae2900-dd72-11ea-99f1-024409bcfa38.png)
![image](https://user-images.githubusercontent.com/62331803/90098197-a3a91980-dd72-11ea-853e-87e2c0e154a9.png)
* overriding 단축키 ⇒ alt + insert
![image](https://user-images.githubusercontent.com/62331803/90098209-ab68be00-dd72-11ea-9932-f5c19541c0bc.png)

# 2. menu
2-a. options menu
![image](https://user-images.githubusercontent.com/62331803/90098234-bd4a6100-dd72-11ea-9642-0d5a53f4572e.png)
![image](https://user-images.githubusercontent.com/62331803/90098238-bf142480-dd72-11ea-94c1-5653c31714cb.png)
![image](https://user-images.githubusercontent.com/62331803/90098252-c3404200-dd72-11ea-8c01-1ea437cc8ff1.png)
![image](https://user-images.githubusercontent.com/62331803/90098257-c50a0580-dd72-11ea-9b19-3a43745d216a.png)

2-b. context menu
![image](https://user-images.githubusercontent.com/62331803/90098281-d3f0b800-dd72-11ea-9f64-4c57b33a2c7a.png)
![image](https://user-images.githubusercontent.com/62331803/90098286-d5ba7b80-dd72-11ea-9201-a4e61ca1aec5.png)
![image](https://user-images.githubusercontent.com/62331803/90098290-d7843f00-dd72-11ea-8a9f-4c8df59ab04d.png)

# 3. listview 라인별로 사용할 레이아웃 생성
![image](https://user-images.githubusercontent.com/62331803/90098311-e7038800-dd72-11ea-9137-fd6572d989dc.png)

- phoneAdapter.java 생성
![image](https://user-images.githubusercontent.com/62331803/90098324-eec32c80-dd72-11ea-8b52-0cf5ed0a0223.png)
![image](https://user-images.githubusercontent.com/62331803/90098343-f7b3fe00-dd72-11ea-98fc-8b8690524162.png)
![image](https://user-images.githubusercontent.com/62331803/90098347-faaeee80-dd72-11ea-8efa-01b7da96a956.png)
- MainActivity3.java 수정
![image](https://user-images.githubusercontent.com/62331803/90098409-27fb9c80-dd73-11ea-8d01-54f56fa2a485.png)
![image](https://user-images.githubusercontent.com/62331803/90098418-2d58e700-dd73-11ea-9ee9-ce02129abf66.png)

*cf1) SMS,CALL 버튼 클릭 불가
<해결방법>
a. 각 버튼의 focusable => false 지정
![image](https://user-images.githubusercontent.com/62331803/90098452-41044d80-dd73-11ea-9699-1cae2f48c8b5.png)
b. 전체 linear layout의 descendantFocusable => block 지정
![image](https://user-images.githubusercontent.com/62331803/90098468-4a8db580-dd73-11ea-807d-a53c08e879fa.png)
*cf2) ACTION_DIAL 사용 permission 등록
![image](https://user-images.githubusercontent.com/62331803/90098491-57120e00-dd73-11ea-9e40-989b9b617ddb.png)

