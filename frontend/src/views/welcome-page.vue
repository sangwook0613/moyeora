<template>
  <el-container>
    <Header
      @openSignupPopup="onOpenSignupPopup"
      @openLoginPopup="onOpenLoginPopup"
    />
    <el-container>
      <welcome-section
        @openSignupPopup="onOpenSignupPopup"></welcome-section>
      <el-footer>Copyright © MOYEORA All Rights Reserved.</el-footer>
    </el-container>
  </el-container>
  <signup-pop
    :open="state.signupPopupOpen"
    @closeSignupPopup="onCloseSignupPopup"
  />
  <login-pop
    :open="state.loginPopupOpen"
    @closeLoginPopup="onCloseLoginPopup"
  />
</template>

<script>
import Header from "@/components/main/header.vue";
import WelcomeSection from "@/components/welcome/welcome-section.vue";
import LoginPop from "@/components/welcome/login-pop.vue";
import SignupPop from "@/components/welcome/signup-pop.vue";
import { reactive } from 'vue'
import { useStore } from 'vuex'
import { useRouter } from 'vue-router'


export default {
  name: "WelcomePage",
  components: {
    Header,
    WelcomeSection,
    LoginPop,
    SignupPop,
  },

    setup(){
    document.body.className = 'white'
    const store = useStore()
    const router = useRouter()
    const state = reactive({
      signupPopupOpen: false,
      loginPopupOpen: false
    })

    const onOpenSignupPopup = function(){
      state.signupPopupOpen = true
    }
    const onCloseSignupPopup = function(){
      state.signupPopupOpen = false
    }
    const onOpenLoginPopup = function(){
      state.loginPopupOpen = true
    }
    const onCloseLoginPopup = function(){
      state.loginPopupOpen = false
      if(isLoggedIn()){
        router.push("/home/all")
      }
    }
    const isLoggedIn = function () {
      return store.getters['root/isLoggedIn']
    }
    return { state, onOpenSignupPopup, onCloseSignupPopup, onOpenLoginPopup, onCloseLoginPopup, isLoggedIn }
  }
};
</script>

<style scoped>
.el-footer {
  min-height: 150px;
  line-height: 150px;
  background-color: var(--color-dark);
  position: relative;
  bottom: 0px;
  color: var(--color-font);
  text-align: center;
  width: 100%;
}
</style>
