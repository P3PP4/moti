<template>
  <div>
    <div class="auth-layout">
      <p class="guide-comment">사용자의 닉네임을 입력해주세요.</p>
      <!-- 닉네임 설정 -->
      <div class="auth-form">
        <div class="auth-input-form">
          <div class="account-id-box">
            <input type="text" id="input-nickname" class="inputbox" name="input-nickname" placeholder="닉네임" @input="nicknameInput">
            <button class="btn-green double-check" @click="doubleCheck">중복체크</button>
            <div v-if="nicknameActive" class="infobox">
              <p
                v-for="(condition, idx) in nicknameConditions"
                :key="idx"
                :class="[condition.valid? 'condition-valid': 'condition-invalid']"
              >
                {{ condition.comment }}
              </p>
            </div>
          </div>
        </div>
        <button v-if="isvalid" id="register-btn" class="btn-green" @click="register">완료</button>
        <button v-else id="register-btn" class="btn-green-inactive" @click="doubleCheckeWarning">완료</button>
      </div>
    </div>
    <basic-modal
      v-if="openModal"
      :content="modalContent"
      @close="modalClose"
    >
    </basic-modal>
  </div>
</template>

<script>
import * as regex from '@/tools/regex.js'
import BasicModal from '@/components/Common/BasicModal'
import { basicModalMixin } from '@/tools/basicModalMixin.js'

export default {
	name: 'SignupNickname',
  components: {
    BasicModal,
  },
  mixins: [
    basicModalMixin,
  ],
  data() {
    return {
      nickname: null,

      nicknameActive: false,
      nicknameDoubleChecked: false,
    }
  },
  methods: {
    // 닉네임 입력 받기
    nicknameInput(event) {
      if (this.nicknameDoubleChecked) {
        this.nicknameDoubleChecked = false
      }
      this.nicknameActive = true
      // 띄어쓰기 및 특수문자 제거
      const regexResult = regex.characterCheckNickname(event.target.value)
      this.nickname = regexResult[0]
      if (regexResult[1]) {
        this.modalContent = regexResult[1]
        this.openModal = true
      }
      event.target.value = this.nickname
    },
    // 닉네임 중복 체크
    doubleCheck() {
      // 임시로 중복체크 true로 바꿔줌
      // this.idDoubleChecked = true
      
      // 닉네임 조건을 충족하지 못한 경우
      if (!this.nicknameDoubleChecked) {
        if ( !this.nicknameConditions[0].valid ) {
          this.openModal = true
          this.modalContent = '닉네임 형식을 지켜주세요.'
        }
        // 닉네임 조건을 충족한 경우
        else {
          // 중복체크
          this.$axios({
            method: 'get',
            url: `${this.$baseUrl}/users/check?type=nickname&value=${this.nickname}`
          })
            .then((response) => {
              // 응답 예시
              // 성공시(이미 존재할 때 ) : 200, already exists
              // 실패시(없을 때) : 200, success 
              // 서버 에러시 : 500, fail

              // 이미 아이디가 존재할 경우
              if ( response.data.message === 'already exists' ) {
                this.openModal = true
                this.modalContent = '이미 사용 중인 닉네임이에요.'
                this.id = null
              } else if ( response.data.message === 'success' ) {
                this.openModal = true
                this.modalContent = '사용할 수 있는 닉네임이에요.'
                // const idInputTag = document.querySelector('#input-nickname')
                // 현재 닉네임으로 고정
                // idInputTag.setAttribute('disabled', true)
                this.nicknameDoubleChecked = true
              } else {
                this.openModal = true
                this.modalContent = '알 수 없는 에러가 발생했습니다. 고객센터에 문의해주세요.'
              }
            })
            .catch((error) => {
              console.log(error)
            })
          }
      }
    },
    register() {
      const payload = {
        nickname: this.nickname,
      }
      this.$store.dispatch('register', payload)
    },
    doubleCheckeWarning() {
      if (!this.nicknameDoubleChecked) {
        this.openModal = true
        this.modalContent = '닉네임 중복체크를 진행해주세요.'
      }
    }
  },
  computed: {
    // 닉네임 입력되었는지 여부 return
    isvalid() {
      if (this.nicknameDoubleChecked) {
        return true
      } else {
        return false
      }
    },
    nicknameConditions() {
      let conditions = [{
        comment: 'X 2~16자. 띄어쓰기와 특수문자는 사용이 안 돼요',
        valid: false,
      }]
      // 닉네임 길이 체크
      if ( this.nickname ) {
        if ( this.nickname.length >= 2 & this.nickname.length <= 16) {
        conditions[0].comment = 'O ' + conditions[0].comment.substring(1, )
        conditions[0].valid = true
        }
      }
      return conditions
    },
  },
  created() {
    // 아이디, 비밀번호, 이메일 입력이 안 된 상태에서는 접근 불가
    if (!this.$store.state.id) {
      this.$router.push({ path: '/signup/main' })
    } else if (!this.$store.state.email) {
      this.$router.push({ path: '/signup/auth' })
    }
  },
  watch: {
    openModal(newVal) {
    // focus를 이메일 인증 버튼으로 이동
      if (!newVal) {
        const registerBtn = document.querySelector('#register-btn')
        registerBtn.focus()
      }
    }
  }
}
</script>

<style lang="scss">
// SignupAuth와 동일. SignupView에 지정되어 있음
</style>