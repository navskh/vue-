# firstweb

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).



(21.04.30~21.05.03)
리뷰 준비
https://fyzh99.tistory.com/34 참조

* 기능별 설명을 하고 시작하자.
  - 회원가입
  - 로그인(로그인 시 헤더에 이름 띄움)
  - 로그아웃 
  - 리스트
  - 조회
  - 등록 (에디터 : tiptap)
  - (등록한 글인 경우) 수정/삭제

0. 기본 환경 설명
- 프레임워크:  vue js
    vue cli로 설치 
    npm i -g @vue/cli 
    > vue 프로젝트 생성
    vue create
    > 기본 파일 시스템이 생기게 됨.
    - package.json 소개
    >> "axios": "^0.21.1",  : 서버와의 통신 (서버단은 node.js를 사용하였음. db: mssql)
    >> "tiptap": "^1.32.2",    : 에디터 
    >> "vue-router": "^3.5.1", : 페이지 이동 시 사용

    - src 디렉토리 안에 기본 소스들이 생성됨
    >> asset : 이미지 파일
    >> component : 컴포넌트 파일들.. 
    >> routes :  페이지 이동과 연관된 파일 (path/url 관리)
    >> App.vue : 최상위 컴포넌트
    >> main.js : 최초 실행되는 자바스크립트

    - main.js 
    : 서버와 통신할 때 사용할 axios.
    : 라우팅 기능 router import
    : new Vue vue를 시작한 후 
    >> App.vue의 <div id=app>가 있는 템플릿을 가져와서 mount 해줌.
  
- App.vue 
    <router-view name="header" />
    >> routes/index.js 파일로 가게 됨
    가장 위의 
    path:'/'
    ,components:{
      default:HelloWorld
      ,header:Header
    }
    components 디렉토리에
    default : HelloWorld
    header : Header.
    여기선 name을 header로 지정하였으므로 상단의
    import Header from '@/components/common/Header'; // 헤더 부분 수정할 때 사용
    부분을 참조하여 src/components/Header.vue 컴포넌트를 가져와 보여준다.
    
    참고로 그 아래 
    <div class="container"> <!--class 추가-->
    <router-view></router-view>
    </div>
    router-view에서 Defualt로 설정 되어 있는 HelloWorld.vue 컴포넌트를 가져온다.

- Header.vue
    4번~6번 line 에서 logo 를 클릭하면 router link인 첫 화면으로 이동하게 만들어줌.

    10~13번 줄 
    v-if 로 LoginFlag를 받아서 로그인 상태이면 현재 로그인 중인 유저의 이름과 로그아웃 버튼을 보여주도록 하였다.
    * 이때 header가 모든 페이지 이동 시 현재 Login 상태인지 아닌지를 알고 있어야 함.
    
    1) LoginFlag 값 변화흐름
    data() 안의 LoginFlag 의 초깃값은 'false'
    ,mounted => fnGetHeader() 를 호출
    get 방식으로 axios를 사용하여 서버단에 보냄
    
    2) firstserver/routes/api 안의 index.js 파일에 가보면 router.get('/',dao.header); 라는 부분이 있음.
    
    3) board 안의 dao.js 파일을 보면
    const db = require('../../../config/db');
    config 폴더 안의 db.js를 열어보면
    mssql에 접속할 수 있게 pool을 만들어 연결하고
    pool 다쓰면 지워주는 식으로 동작함.

    4) 다시 dao.js 로 돌아와서 
    여기에서 보면 header 라는 부분에 가서
    select문으로 현재 로그인 한 아이디로 검색하여 이름 정보를 가져온다.
    (여기서 uID는 서버단에서 선언한 전역변수이다. user 메서드에 가보면 로그인 시 uID에 아이디 정보를 넣어주는 것을 알 수 있다.)

    그 다음 171 line을 보면
    LoginFlag 값을 클라이언트 단으로부터 false로 받을 시 
    전역변수로 설정한 LoginFlag를 false 로 만들어준다.
    이는 로그아웃 상황에서 LoginFlag를 false로 입력하기 위해서 처리해주는 코드이다.

    이로써 흐름은 다음과 같다.
    로그인 시 User.vue => dao.js (user 메서드) 에서 LoginFlag true로 설정 => dao.js(header 메서드) 에서 Header.vue에 현재 로그인한 이름 정보와 LoginFlag 전달.

    5) 다시 Header.vue 를 마저 설명하면
    71번 line
    ,fnLogout() 
    에서는 LoginFlag를 false로 설정하여 axios로 서버단에 보내준다. 해당 설명은 위에서 하였다.
    
    그 외 fnViewList는 list를 보여주는 페이지로 이동하는 것이다.

- List.vue : "게시판 클릭" 시 리스트 가져오기.   
    List를 일단 불러오면..

    1) (리스트 불러오기 - 서버단에서의 동작) 
    91번줄 mounted() 로 이동한 후 
    this.fnGetList(2)를 실행함. (여기서 2로 준 이유는 기본 모드를 내림차순으로 설정하기 위함임.)

    95번째 줄 
    fnGetList(updown)
    일단 게시글에 대한 정보는 전부 Sql Server에 저장되어 있음.
    
    그러므로
    dao.js 39번 line
    where, sql 에 query 명령을 넣고
    totalCount(글의 총갯수, sql 결과인 data.recordset[0].cnt)를 사용하여
    total page, start, end page 를 구한 후 

    client로 부터 받은 updown이 1이면 오름차순
    2면 내림차순으로 구성하게 만들었다. (2인경우 끝에 order by i_num desc가 붙음)

    2) 
    서버단으로부터 통신 성공하면
    this.list 에 저장한 후
    해당 list를 v-for 문으로 돌려서 그 안의 데이터를 표안에 넣어서 출력해준다.

    3) fnAdd()
    등록 버튼 클릭할 시 동작
    /.write 로 이동함.

    4) fnSearch()
    검색 버튼 클릭할 시 동작
    page를 1로만들어 fnGetList 호출
    키워드를 가지고 서버단과 통신하면서 
    해당 키워드로 where 문을 사용하여 sql에서 검색

    5) fnPage(n)
    this.page = n 을 만들어줌으로, 해당 페이지로 이동하게 됨 
    맨 왼쪽에 있는 << 화살표를 클릭하면 1번페이지로 이동
    end page가 10보다 크면 < 화살표가 나오고 한칸 이동
    우측도 마찬가지

    6) fnView
    view 페이지로 이동

    7) fnAscend, fnDescend
    오름차순, 내림차순 정렬
    

    8) fnSelectViewMode()
    10개, 30개, 100개 모드를 결정하여 리스트 출력 

- Write.vue     
    
    0) 일단 기본 템플릿은
    제목 입력
    내용 입력 (tiptap 에디터 사용)

    *) tiptap에디터 사용법
    import    Editor.vue 에서 가져옴
    componet에 Editor 등록

    Editor.vue 안을 보면
    import { defaultExtensions } from '@tiptap/starter-kit'
    extension을 사용하여 설정값을 가져오고
    bubble 메뉴 : 드래그 시 해당 글의 폰트 변경이 가능함.
    floating 메뉴 : 커서 옮길 시 작성전 폰트 변경이 가능함.
    editor-content : 입력 공간

    1) mounted 가 되면
    if문으로 this.num이 있으면 (수정 조건으로 받아서 fnGetView를 호출 - 수정할 때는 이전의 글의 상태가 그대로 들어가 있어야 되기 때문임.)
    this.num 은 게시글 고유 숫자. 해당 글의 pk임. num 값으로 글의 정보들을 가져옴

    2) 
    fnGetView() 호출
    axios로 서버단에 보냄.
    this.body 에 묶어서 보냄 이 정보들은 
    View.vue 에서 body로 전달하는데 이 body는 List,vue에서 query로 전달한 값
    List.vue에서 fnView() 를 보면 body 안에 정보들이 다음과 같이 들어가 있음.
    this.body.num = num;
    this.body.updown = updown;
    this.body.LoginFlag = this.LoginFlag;
    this.body.userID = this.userID;

    이 정보들을 가지고 서버단에 보냄.
    dao.js (view 메서드 호출)
    select 문을 사용하여 해당 글의 num값을 통해 글의 제목, 내용등을 가져옴.

    그러면 해당 데이터를 그대로 수정할 수 있도록 띄워줌.
    v-model = cont 로 맵핑시켜주었음.

    3) 그외 버튼이 세가지가 있음.
    목록, 등록, 수정
    여기서 등록과 수정은 num을 받았을 땐 (수정모드이므로) 수정이 보이고
    num 이 없을 때는 등록이 보이게끔 만들어주었음.

    4) 목록 버튼 클릭 시 
    fnList() 호출
    여기서 List로 돌아갈 때는 무조건 내림차순을 보여주도록 돌아감. 왜냐면 가장 최신 리스트가 보여져야 하기 때문임.


    5) 등록 버튼 클릭 시 
    fnAddProc() 이 호출되고
    먼저 제목이 없으면 alert 창을 띄워줌.

    this.form 변수 안에 
    board_code : 기존에는 여러 카테고리로 나눠서 게시판을 구성하려 하였으나 하나의 카테고리만을 사용하므로 news로 통일
    subject : 제목
    cont : content 내용
    id : 작성자 id를 입력

    6) dao.js (add메서드 호출)
    insert 문으로 해당 정보들을 db에 insert 시킴

    7) 수정 호출 시 
    fnModProc() 함수 호출
    fnAddProc() 과 거의 비슷
    dao.js 에서 mod 메서드를 보면 add 메서드에서 insert를 update로 바꿔준것 외에는 동일

- View.vue : 게시판 상세 보기
    1) fnGetView() 호출
    제목, 내용, 작성자 정보 가져오고 그 정보를 페이지에 출력해줌
    여기서 내용 정보를  replace (../) 추가해주어서 한칸 띄는 것이 가능하게 해주었다.

    2) 페이지 상에서
    22번 line 작성자가 지금 접속한 user의 id가 같으면
    목록, 수정, 삭제 버튼 만들어줌.

    3) 목록 버튼 클릭 시 
    fnList() 호출 됨. updown 정보를 받아서 보내줌.
    리스트 > 뷰 > 리스트 updown 값을 유지.

    4) 수정 버튼 클릭 시 
    fnMod()  호출 > body에 num 값을 넣어서 write로 이동

    5) 삭제 버튼 
      fnDelete() 호출
      axios로 서버에 dao.js>delete 메서드 호출
      num 값을 받아서 delete 문 전달
      성공 하면 fnList() 호출하여 리스트 보여줌

- User.vue
    로그인 시 
    로그인/ 회원가입 버튼 

    1) 로그인 버튼 클릭 시
      axios로 서버단과 통신
      dao.js > user 메서드에서 해당 UserID로 검색하여 해당 user의 이름.  password 정보 가져옴
      만약 user 에 아이디가 있는 경우 LoginFlag를 true로 만들어줌 (LoginFlag가 false일 시 success:false로 설정)
      LoginFlag true인 경우 전역변수 uID에 저장하고(헤더에 이름 띄워주기 위함) 클라이언트단에 보내줌

    2) 서버로부터 응답받은 후
      success:false라면  존재하지 않는 아이디 식별
      success:true 인 경우 password 비교하여 일치하면 로그인 성공
      로그인 성공 시 LoginFlag를 true로 만들어 전달.
      리스트를 띄워줌.

    3) 회원가입 버튼 클릭 시 
      fnUserAdd() 호출, UserAdd.vue 로 이동        

- UserAdd.vue
    1) UI 보면
      이름, 아이디(중복체크), 비밀번호, 비밀번호 확인,  회원가입 완료
      v-model 로 data 바인딩 해줌

    2) 중복체크(fnCheck) 
      userId를 넣어서 서버측에 보냄.
      같은 id가 있을 경우 checkID 를 1로만듬 
      같은 id 없으면 checkID = 0 으로 만듬 
      (checkID 기본 값 1임.)

    3) 비밀번호 확인 (passwordConfirm)
      password와 passwordConfirm 값이 같으면 완료 메세지 띄우고 그전까지는 경고 메세지 띄움
      같지 않은 상태면 회원가입시 막힘 얼럿 창 뜸

    4) 회원가입 (fnSubmitForm)
      userName
      userID
      password
      passwordConfirm
      이 모두 완료 되면 서버측과 통신하여
      회원가입 시킴

      서버측에서는 dao.userAdd 메서드에서 
      Insert로 user_info db에 추가시킴

      아이디 중복확인이 안될 경우는 특별히 걸러줘야 함.
