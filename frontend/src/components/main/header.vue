<template>
  <el-header v-if="isLoggedIn()" class="main-header" :height="'70px'">
    <a href="/home/all" class="logo">
      <img :src="require('@/assets/img-logo.png')" alt="IceBreaking logo" width="80" height="60" />
    </a>
    <ul class="nav">
      <li>
        <el-dropdown trigger="click">
          <el-badge :value="state.inviteCount" type="danger">
            <el-button icon="el-icon-bell" circle></el-button>
          </el-badge>
          <template #dropdown>
            <el-dropdown-menu style="overflow:auto; background-color: var(--color-light);">
              <el-dropdown-item v-if="state.inviteCount==0" icon="el-icon-check" @click="getInvite">알림이 없습니다.</el-dropdown-item>
              <el-dropdown-item v-else v-for="item in state.inviteData" :key="item.id">
                <div v-if="item.type == 'invite'">
                  <i class="el-icon-bell"></i>{{item.fromUser}}님이 {{item.toUser}}님을 초대하셨습니다.
                </div>
                <div v-else>
                  <i class="el-icon-bell"></i>{{item.fromUser}}님의 친구요청.
                </div>
                <div style="text-align:right">
                  {{item.await}}
                  <i v-if="item.type == 'invite'" style="color:green" class="el-icon-right" @click="clickInvite(item.url, item.id)">참가</i>
                  <i v-else style="color:green" class="el-icon-check" @click="clickAccept(item.id, item.fromUser)">수락</i>
                  <i style="color:red" class="el-icon-delete" @click="clickDelete(item.id)">삭제</i>
                </div>
              </el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </li>
      <li>
        <el-dropdown trigger="click">
          <el-button icon="el-icon-user" circle></el-button>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item icon="el-icon-user" @click="clickUserInfo">내 정보</el-dropdown-item>
              <el-dropdown-item icon="el-icon-back" @click="clickLogout">로그아웃</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </li>
    </ul>
  </el-header>
  <el-header v-if="!isLoggedIn()" class="main-header" :height="'70px'">
    <a href="/home/all" class="logo">
      <img :src="require('@/assets/img-logo.png')" alt="IceBreaking logo" width="80" height="60" />
    </a>
    <div class="button-wrapper">
      <el-button icon="el-icon-circle-plus-outline" @click="clickSignup">회원 가입</el-button>
      <el-button icon="el-icon-key" type="primary" @click="clickLogin">로그인</el-button>
    </div>
  </el-header>
</template>

<script>
import { reactive } from 'vue'
import { useStore } from 'vuex'
import { useRouter } from 'vue-router'

export default {
  name:'Header',

  setup(props, {emit}){
    const store = useStore()
    const router = useRouter()
    const alarm = localStorage.getItem('alarm')
    const state = reactive({
      inviteCount: 0,
      inviteData: [],
      fullscreenLoading: false,
    })

    // 로그인 유무 확인
    const isLoggedIn = function () {
      return store.getters['root/isLoggedIn']
    }

    // 로그인 팝업 열기
    const clickLogin = () => {
      emit('openLoginPopup')
    }

    // 회원가입 팝업 열기
    const clickSignup = () => {
      emit('openSignupPopup')
    }

    // 로그아웃
    const clickLogout = () => {
      store.commit('root/removeToken')
      router.push("/")
    }
    const clickUserInfo = () => {
      //console.log('user') 밑에 router 자기 id로 옮겨지게
      router.push("/home/user/id")
    }
    const clickInvite = (url, id) =>{
      store.dispatch('root/deleteInvite', { id: id })
      window.location.href = url;
    }

    const clickAccept = (id, fromUser) =>{
      store.dispatch('root/requestMakeFriend',{ toUser: fromUser})
          .then((result) => {
            store.dispatch('root/deleteInvite', { id: id })
            getInvite()
          })
          .catch((err)=>{
            alert("오류! 다시 시도해주세요.")
          })
    }

    const clickDelete = (id) =>{
      store.dispatch('root/deleteInvite', { id: id })
      getInvite()
    }
    const getInvite = ()=>{
      if(alarm == "off"){
        return
      }
        store.dispatch('root/requestInviteList', { token: localStorage.getItem('jwt') })
          .then((result) => {
            //console.log(result)
            let today = new Date();
            let month = today.getMonth() + 1;  // 월
            let day = today.getDate();  // 날짜
            let hour = today.getHours(); // 시
            let min = today.getMinutes();  // 분
            state.inviteData = []
            state.inviteCount = 0
            result.data.forEach(item => {
              let invite = {id:"", fromUser:"", toUser:"", url:"", await:""}
              invite.id = item.id
              invite.fromUser = item.fromUser
              invite.toUser = item.toUser
              invite.url = item.url
              invite.type = (invite.url == null)? "friend" : "invite"
              if(month!=item.month){
                invite.await = (parseInt(month)-parseInt(item.month)) + "달 전"
              }else if(day != item.day){
                invite.await = (parseInt(day)-parseInt(item.day)) + "일 전"
              }else if(hour != item.hour){
                if((parseInt(hour)*60 + parseInt(min)) - (parseInt(item.hour)*60 + parseInt(item.min)) < 60){
                  invite.await = ((parseInt(hour)*60 + parseInt(min)) - (parseInt(item.hour)*60 + parseInt(item.min))) + "분 전"
                }else{
                  invite.await = (parseInt(hour)-parseInt(item.hour)) + "시간 전"
                }
              }else{
                if(parseInt(min)-parseInt(item.min) < 3){
                  invite.await = "방금"
                }else{
                  invite.await = (parseInt(min)-parseInt(item.min)) + "분 전"
                }
              }
              state.inviteCount ++
              state.inviteData.push(invite)
            });
            // 삭제시 @DeleteMapping("/delete/{inviteId}") 이 주소
          })
          .catch((error) => {
            console.log(error)
          })
    }
    getInvite()
    //setInterval(function(){getInvite()},5000);

      return { state, isLoggedIn, clickLogin, clickSignup, clickLogout, clickUserInfo, clickInvite, clickAccept, clickDelete, getInvite }
  },
}
</script>

<style>
</style>
