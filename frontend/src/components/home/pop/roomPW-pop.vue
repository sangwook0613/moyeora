<template>
  <el-dialog width="30%" title="비밀번호 확인" v-model="state.popupVisible" @close="handleClose">
    <div class="pwPopup" >
      <el-input label="PW:" class="pwInput" placeholder="비밀번호를 입력하세요." v-model="state.input" clearable></el-input>
      <br/>
      <el-button type="primary" @click="checkPW">제출</el-button>
      <el-button type="danger" @click="handleClose">취소</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { reactive, computed } from 'vue'
import { useRouter } from 'vue-router';
import { useStore } from 'vuex'

export default {
  name:"RoomPWPop",
  props:{
    open: {
      type: Boolean,
      default: false,
    },
    type:"",
    id:"",
    },
  setup(props, { emit }) {
    const router = useRouter()
    const store = useStore()

    const state = reactive({
      input: "",
      popupVisible: computed(() => props.open),
    })
    const checkPW = ()=>{
      store.dispatch('root/checkRoomPwd', {roomId:props.id, pwd:state.input})
      .then(function (result) {
        //console.log(result.data)
        if(result.data.statusCode == 200){
          emit('closePwPopup')
          router.push("/game/" + props.type + "/" + props.id)
        }else{
          alert("비밀번호가 틀립니다!")
          emit('closePwPopup')
        }
      })
      .catch(function () {
        alert("서버오류")
        emit('closePwPopup')
      })
    }

    const handleClose = function () {
      emit('closePwPopup')
    }
    return { state, checkPW, handleClose }
  }
}
</script>

<style>
</style>
