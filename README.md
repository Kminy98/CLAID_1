# 🎵CLAID🎵

![대지 1-100](https://github.com/madonghwi/CLAID_1/assets/97779949/4557236b-7deb-49cc-ad6e-047561de2ee3)

### **누구든 가수가 될 수 있다!**
1. **내 목소리를 AI에게 학습하는 방법을 공유**
2. **원하는 노래에 적용!**
3. **고음, 저음 상관없이 능숙하게 잘 부르는 내 목소리를 감상!**

## ✨Front-End
[Front-End Link](https://github.com/madonghwi/CLAID_front)

------
## 📑목차
- [🔧설치방법](#설치방법)
- [🛠트러블 슈팅](#트러블-슈팅)
- [✋역할](#역할)
- [🖥기능](#기능)
- [🔴시스템 아키텍처](#시스템-아키텍처)
- [🟠ERD](#erd)
- [🟡API 명세](#api-명세)
- [🟢와이어 프레임](#와이어-프레임)
- [🔵역할분담 + 전체 S.A.](#역할분담--전체-sa)
- [🟣프로젝트 타임라인](#프로젝트-타임라인)

-----

## 🔧설치방법
1. **리포지토리 클론**:
   ```
   git clone https://github.com/yourusername/your-repo-name.git
   cd your-repo-name
   ```
2. 가상 환경 생성 및 활성화:
    ```
    python -m venv venv
    source venv/bin/activate  # Unix/macOS
    venv/Scripts/activate   # Windows
    ```
3. 필요한 패키지 설치:
    ```
    pip install -r requirements.txt
    pip install spleeter
    ```
4. 환경변수설정:.env 파일 생성
    ```
    SECRET_KEY='[]'
    EMAIL_HOST_USER='[]'
    EMAIL_HOST_PASSWORD=[]
    API_KEY=[]
    ```
5. 마이그레이션:
    ```
    python manage.py makemigrations
    python manage.py migrate
    ```
6. 서버 실행:
    ```
    python manage.py runserver
    ```
-----
## 🛠트러블 슈팅
### 🔘ffmpeg binary not found
- 설명
    - 음원파일 추출하려고 하면 에러발생, pip install ffmpeg를 했는데도 계속 같은 에러가 발생함
- 문제원인
    - 처음에는 ffmpeg-python과 ffmpeg가 동시에 설치되어 있어서 생긴 에러라고 파악했음(그래서 해결과정의 1~3번 시도)
    - ffmpeg 실행 파일이 프로젝트 해당 드라이브에 설치되어있지 않아 생긴 에러였음(해결과정5~8시도 후 에러 해결!)
- 해결 과정
    1. pip uninstall ffmpeg
    2. pip uninstall ffmpeg-python
    3. pip install ffmpeg-python
    4. pip list에 보면 ffmpeg-python이 잘 들어가 있지만 음원 추출을 시도하면 같은 에러가 발생
    5. https://www.gyan.dev/ffmpeg/builds/ 사이트에서 ffmpeg 실행 파일([ffmpeg-git-full.7z](https://www.gyan.dev/ffmpeg/builds/ffmpeg-git-full.7z))을 설치함
    6. 시스템환경변수에서 path에 ffmpeg파일 경로를 추가함 ((ex)C:\Program Files\FFmpeg\bin)
    7. +그래도 계속 같은 에러가 발생함
    8. D드라이브에 있던 프로젝트 파일과 ffmpeg파일을 C드라이브로 이동하고 path 경로도 수정해줌

 ### 🔘user 회원가입 이메일 인증 개선
- 설명
    - 회원가입 시 이메일 인증하기 버튼을 누르면 약 10~30초간 화면에 아무런 반응이 없어 사용자가 불편함을 느낄 것이라고 생각됨
- 문제원인
    - 이메일이 전송되는 동안 메시지 창 띄우기와 같은 다른 작업을 수행할 수 없어서  이메일이 실제로 보내질 때 까지 화면에는 아무런 반응이 없는 것
- 해결 과정
    1. 이메일이 전송되는 동안에도 다른 작업을 수행 할 수 있도록 이메일 전송을 비동기 처리 하기로 함
    2. 셀러리랑 threading 둘 중 내장 라이브러리인 threading를 선택 (이메일 발송 작업을 별도의 스레드에서 실행 할 수 있음)
    3. UserSerializer > create함수에 있던 이메일 전송 코드를 send_email함수를 추가해 따로 분리함 
    4. User Serializers.py에 `from threading import Thread` 추가
    5. create함수에  send_email함수를 새로운 스레드에서 실행할 코드 작성

       ```email_thread = Thread(target=self.send_email, args=(user,))```
       ```email_thread.start()```

### 🔘일반 로그인 E-mail 인증
- 설명
    - Email 인증 기능을 통해 email로 인증을 받아서 회원을 활성화 하려고 하는데 일반 회원 E-mail 인증시 같은 코드인데도 사람에 따라서 작동이 되지 않음
- 문제원인
    - 보안 인증이 되지 않음
- 해결과정
    1. db를 지우고 마이그레이션을 다시 생성 해봤으나 문제 해결이 되지않음
    2. env 코드를 받아오는지 확인하기 위해 print로 찍어보니 출력이 되지않음
    3. 이메일 인증을 위해 필요한 정보를 .env 파일에서 받아오지 않고 settings.py에 직접 넣어주니 작동함
    4. 다른 .env 정보들은 받아 올 수 있는 것을 확인 (이메일 인증만 x)
    5. 구글 보안설정에서 2단계 인증을 사용안함에서 사용으로 설정 변경했더니 해결 되었음
------

## ✋역할
- 일반 회원가입, 로그인, 마이페이지:
    - 일반 회원가입 시 이메일 인증해야 가입완료

- 포인트
    - 게시글, 댓글 작성시 포인트 지급
    - 게시글, 댓글 24시 지나기 전 삭제시 포인트 차감
    - 로그인한 사용자 포인트 조회
    - 관리자 아이디 포인트 list 가져오기
    - 슈퍼계정이 일반계정에게 포인트 지급, 차감
    - 전체 포인트 히스토리 조회

- 게시글
    - 게시글 CREATE
    - 게시글 READ
    - 게시글 UPDATE
    - 게시글 DELETE
------

## 🖥기능
### 회원기능 : Jwt Token, Oauth 2.0 사용

1. 회원가입 `POST`
    - 일반 회원가입
    - SNS 회원가입
        - SNS 사이트에 이동하여 사용자 인증 후 앱 연결 동의 후 로그인으로 연결
            - KAKAO
            - GOOGLE
2. 이메일 인증 `POST`
    - uid64를 base64로 디코딩하여 사용자 식별 값을 생성 후 이메일 인증
3. 로그인 `POST`
    - 일반 로그인
    - SNS 로그인
        - SNS 사이트에 이동하여 사용자 인증 후 JWT Token 발급
            - KAKAO
            - GOOGLE
4. 로그아웃
    - 사용자의 웹 브라우저에서 JWT Token 삭제
5. 프로필 `GET`
    - 프로필 이미지, 닉네임

### 포인트
- 회원 가입 시 1000p 지급
- 게시글 작성 시 1000p 지급
- 댓글 작성시 500p 지급
- 게시글 작성 후 24시간이내 삭제 시 1000p 차감

### 음원 분리 (separator)
1. 음악 업로드 `POST`
    - 사용자가 음악을 업로드하면 머신러닝 Spleeter가 분석
    - 보컬과 악기로 나누어줌.
2. 보컬 / 악기 `GET`
    - 목록
        - 음원 분리한 목록들을 PageNumberPagination을 Custom해서 5개씩 나열해줌.
        - 음원들을 wave로 표현
    - 상세페이지
        - 음원들을 wave로 표현
        - 보컬과 악기 오디오를 함께 들으며 테스트
        - 다운로드
        - 볼륨 조절

### 자랑 (article)
1. 게시글 CREATE `POST`
    - 사용자가 제목, 내용, 이미지, 음원파일을 입력하고 작성완료 버튼 클릭 시 게시글 리스트에 생성되어 공유.
2. 게시글 READ `GET`
    - 목록
        - 홈, index.html 게시글 목록
        - 사진과 오디오로 목록화, 누르면 해당 게시글 상세 페이지
    - 상세 페이지
        - 해당 게시글의 작성시간, 작성자 등을 포함한 상세 페이지
        - 댓글 작성 가능
        - 해당 게시글에 달린 댓글 확인 가능

3. 게시글 UPDATE `PATCH`
    - 현재 로그인 되어있는 사용자와 해당 글 작성자가 동일할 경우에만 수정 버튼 생성되며 수정 가능

4. 게시글 DELETE `DELETE`
    - 현재 로그인 되어있는 사용자와 해당 글 작성자가 동일할 경우에만 삭제 버튼 생성되며 삭제 가능
    
5. 댓글 목록 `GET`
    - 댓글을 단 사람의 프로필이미지와 이름을 보여줌
    - 유저네임, 프로필이미지, 좋아요, 댓글내용을 가져옴
    
6. 댓글 작성 `POST`
    - 로그인 한 상태로 내용을 입력해서 댓글을 작성가능 
    - 내용이 없는 댓글 작성불가

7. 댓글 수정 `PATCH`
    - 댓글 작성자만 수정이 가능함
    - 댓글 수정중 취소할 수 있음

8. 댓글 삭제 `DELETE`
    - 댓글 작성자만 가능

9. 좋아요 `POST`
    - 로그인한 사용자만 가능
    - 다시 한번 누르면 좋아요를 취소 할 수 있음

10. 조회수 `GET`
    - IP당 하루에 한 번 오르게 설정
    
### FAQ (VocalNotice)
  - VocalNotice -> 질문 / 공지
1. 게시글 CREATE `POST`
    - 사용자가 제목과 내용을 입력하고 작성완료 버튼 클릭 시 게시글 리스트에 생성되어 공유.
    
2. 게시글 READ `GET`
    - 목록
        - 홈, 게시글 목록을 누르면 해당 게시글 상세 페이지 로드
        - 페이지네이션을 통해 게시글이 많아졌을 때 한번에 10개의 게시글만 보이도록 목록 분리
        -
    - 상세 페이지
        - 해당 게시글의 작성시간, 작성자 등을 포함한 상세 페이지
        - 게시글 좋아요
        - 해당 게시글에 달린 댓글 확인 가능
        - 댓글 작성 가능

3. 게시글 UPDATE `PATCH`
    - 해당 글 작성자만 수정 가능

4. 게시글 DELETE `DELETE`
    - 해당 글 작성자만 삭제 가능

5. 댓글 목록 `GET`
    - 댓글을 단 사람의 프로필이미지와 이름을 보여줌
    - 유저네임, 프로필이미지, 좋아요, 댓글내용을 가져옴

6. 댓글 작성 `POST`
    - 로그인 한 상태로 내용을 입력해서 댓글을 작성가능 
    - 내용이 없는 댓글 작성불가

7. 댓글 수정 `PATCH`
    - 댓글 작성자만 수정이 가능함
    - 댓글 수정중 취소할 수 있음

8. 댓글 삭제 `DELETE`
    - 댓글 작성자만 가능

9. 좋아요 `POST`
    - 로그인한 사용자만 가능
    - 다시 한번 누르면 좋아요를 취소 할 수 있음

10. 조회수 `GET`
    - IP당 하루에 한 번 오르게 설정

### 검색 (Search)
1. artice, vocalnotice 검색 `POST`
    - 각 모델에서 정보를 필터링하여 가져옴

2. 검색결과 `GET`
    - 가져온 결과를 보여줌 내용(contents)에는 제한을 두어 일정 글자수 이상은 ...으로 표기됨 
***

## 🔴시스템 아키텍처
![Untitled](https://github.com/madonghwi/CLAID_1/assets/97779949/6cb9b9b8-b526-40d3-8d87-704317b254ae)

## 🟠ERD
![b7-claid-erd](https://github.com/madonghwi/CLAID_1/assets/97779949/91a7a329-0bf8-49c4-b9da-d85b04d0fd90)

## 🟡API 명세
[API 명세](https://juicy-bow-33d.notion.site/4f68e45033084ac8be3d7a4ba7e76ac9?v=62276268370f48a9be9dd75b7880ae87&pvs=4)

## 🟢와이어 프레임
[와이어 프레임](https://www.figma.com/design/OH25J6tB2MTR1oRDNXh4yv/CLAID-Wireframe?node-id=0-1&t=1UDQlZ6y5r4Of4PF-1)

## 🔵역할분담 + 전체 S.A.
[역할분담 + SA](https://juicy-bow-33d.notion.site/S-A-4e0242adf881433d9273057cca09496a?pvs=4)

## 🟣프로젝트 타임라인
[프로젝트 타임라인](https://juicy-bow-33d.notion.site/04d66670f2424788843bb1c5170dc8c8?v=f865c6b085ad49b28e3fa0b408aab2de&pvs=4)