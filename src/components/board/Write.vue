<template>
	<div>
		<h1>게시판 {{num? '수정' : '등록'}}</h1> <!--num 값에 따라 제목 변경-->
		<div class="Addrap">
			<form>
				<table class="tbAdd">
					<colgroup>
						<col width="15%" />
						<col width="*" />
					</colgroup>
					<tr>
						<th>제목</th>
						<td><input type="text" v-model="subject" ref="subject"></td>
					</tr>
					<tr>
						<th>내용</th>
						<td>
              <editor style="height:200px" v-model="cont"/>
            </td>
					</tr>
				</table>
			</form>
		</div>

		<div class="btnWrap">
			<a href="javascript:;" @click="fnList" class="btn">목록</a>
			<a v-if="!num" href="javascript:;" @click="fnAddProc" class="btnAdd btn">등록</a>
			<a v-else href="javascript:;" @click="fnModProc" class="btnAdd btn">수정</a>
		</div>	
	</div>
</template>

<script>
import Editor from './Editor'

// import { defaultExtensions } from '@tiptap/starter-kit'

export default {
  components: {
    Editor,
    // EditorContent,
    // BubbleMenu,
    // FloatingMenu,
  },
  
	data() { //변수 생성
		return{
      body:this.$route.query
      ,board_code:'news'
			,subject:''
			,cont:''
			,id:'admin'
      ,form:'' //form 전송 데이터
      ,num:this.$route.query.num
      ,updown:this.$route.query.updown
      ,LoginFlag:this.$route.query.LoginFlag
      ,userID:this.$route.query.userID
		}
  }
  ,mounted() { //최초 로딩 시 실행
		if(this.num) { // num 값이 있으면 상세 데이터 호출한다.
			this.fnGetView();
		}
	}
  
	,methods:{
    fnList(updown){ //리스트 화면으로 이동 함수
      this.body.updown = updown;
      this.body.LoginFlag = this.LoginFlag;
      this.body.UserID = this.userID;
			this.$router.push({path:'./list',query:this.body});
    }
    ,fnGetView() {
			this.$axios.get('http://localhost:3000/api/board/'+this.num,{params:this.body})
			.then((res)=>{
        this.view = res.data.view.recordset[0];
				this.subject = this.view.subject;
				this.cont = this.view.cont;
			})
			.catch((err)=>{
				console.log(err);
			})
    }
    ,fnView() {
      console.log(this);
      this.$router.push({path:'./view', query:this.body});
		}

		,fnAddProc() { //등록 프로세스
			if(!this.subject) { //제목이 없다면 값을 입력하라고 알려준다.
				alert("제목을 입력해 주세요");
				this.$refs.subject.focus(); //방식으로 선택자를 찾는다.
				return;
			}

			this.form = { //backend로 전송될 POST 데이터
				board_code:this.board_code
				,subject:this.subject
				,cont:this.cont
				,id:this.userID
			} 
			this.$axios.put('http://localhost:3000/api/board/'+this.id,this.form)
			.then((res)=>{
				if(res.data.success) {
					alert('등록되었습니다.');
					this.fnList(2);
				} else {
					alert("실행중 실패했습니다.\n다시 이용해 주세요");
				}
			})
			.catch((err)=>{
				console.log(err);
			})
    }
    ,fnModProc() {
			if(!this.subject) {
				alert("제목을 입력해 주세요");
				this.$refs.subject.focus(); //방식으로 선택자를 찾는다.
				return;
			}
			this.form = {
				board_code:this.board_code
				,subject:this.subject
				,cont:this.cont
				,id:this.id
				,num:this.num
      } 
			this.$axios.put('http://localhost:3000/api/board',this.form)
			.then((res)=>{
				if(res.data.success) {
					alert('수정되었습니다.');
					this.fnView();
				} else {
					alert("실행중 실패했습니다.\n다시 이용해 주세요");
				}
			})
			.catch((err)=>{
				console.log(err);
			})
		}
	}
  ,beforeDestroy() {
    if (this.editor) {
      this.editor.destroy()
    }
  }
  
}
</script>

<style scoped>
	.tbAdd{border-top:1px solid #888;}
	.tbAdd th, .tbAdd td{border-bottom:1px solid #eee; padding:5px 0;}
	.tbAdd td{padding:10px 10px; box-sizing:border-box;}
	.tbAdd td input{width:100%; min-height:30px; box-sizing:border-box; padding:0 10px;}
	.tbAdd td textarea{width:100%; min-height:300px; padding:10px; box-sizing:border-box;}
	.btnWrap{text-align:center; margin:20px 0 0 0;}
	.btnWrap a{margin:0 10px;}
	.btnAdd {background:#43b984}
	.btnDelete{background:#f00;}
</style>


<style lang="scss">
/* Basic editor styles */
.ProseMirror {
    height: 200px;

  > * + * {
    margin-top: 0.75em;
  }


  ul,
  ol {
    padding: 0 1rem;
  }

  h1,
  h2,
  h3,
  h4,
  h5,
  h6 {
    line-height: 1.1;
  }

  code {
    background-color: rgba(#616161, 0.1);
    color: #616161;
  }

  pre {
    background: #0D0D0D;
    color: #FFF;
    font-family: 'JetBrainsMono', monospace;
    padding: 0.75rem 1rem;
    border-radius: 0.5rem;

    code {
      color: inherit;
      padding: 0;
      background: none;
      font-size: 0.8rem;
    }
  }

  img {
    max-width: 100%;
    height: auto;
  }

  blockquote {
    padding-left: 1rem;
    border-left: 2px solid rgba(#0D0D0D, 0.1);
  }

  hr {
    border: none;
    border-top: 2px solid rgba(#0D0D0D, 0.1);
    margin: 2rem 0;
  }
}

.bubble-menu {
  display: flex;
  background-color: #0D0D0D;
  padding: 0.2rem;
  border-radius: 0.5rem;

  button {
    border: none;
    background: none;
    color: #FFF;
    font-size: 0.85rem;
    font-weight: 500;
    padding: 0 0.2rem;
    opacity: 0.6;

    &:hover,
    &.is-active {
      opacity: 1;
    }
  }
}

.floating-menu {
  display: flex;
  background-color: #0D0D0D10;
  padding: 0.2rem;
  border-radius: 0.5rem;

  button {
    border: none;
    background: none;
    font-size: 0.85rem;
    font-weight: 500;
    padding: 0 0.2rem;
    opacity: 0.6;

    &:hover,
    &.is-active {
      opacity: 1;
    }
  }
}
</style>