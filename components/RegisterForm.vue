<template>

  <div>
    <div v-if="show" @click="show = !show" class="popup-bg"></div>
    <div v-if="show" class="popup-content register">
      <div class="panel register">
        <div class="title">
          <h3>register</h3>
        </div>
        <div v-if="step == 1" class="form">
          <div class="input">
            <label>Email <span v-if="errors.email" class="error">{{errors.email[0]}}</span></label>
            <input v-model="userInfo.email" type="email" class="input input-form input-form1">
          </div>
          <div class="input">
            <label>Username <span v-if="errors.username" class="error">{{errors.username[0]}}</span></label>
            <input v-model="userInfo.username" type="text" class="input input-form input-form1">
          </div>
          <div class="input">
            <label>Password <span v-if="errors.password" class="error">{{errors.password[0]}}</span></label>
            <input v-model="userInfo.password" type="password" class="input input-form input-form1">
          </div>
          <div class="input">
            <label>Confirm <span v-if="errors.password" class="error">{{errors.password[0]}}</span></label>
            <input v-model="userInfo.password_confirmation" type="password" class="input input-form input-form1">
          </div>
        </div>
        <div v-if="step == 2" class="form">
          vunt
        </div>
        <div @click="submitForm()" class="submit gridcenter">register</div>
      </div>
    </div>
  </div>

</template>

<style lang="scss">
.error{
  color: var(--danger);
  font-weight: 500;
  font-size: calc(0.8rem + 0.3vw);
}
.register{
    max-height: 550px;
    max-width: 550px;

}
.popup-bg{
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.25);
  z-index: 998;
}
.popup-content{
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 999;
  height: 90vh;
  width: 95vw;

}
.title, .form{
  margin-bottom: 5%;
}
 .title{
   text-align: center;
   
      h3{
        text-shadow: -4px 4px 0px var(--main);
        color: var(--dark);
        font-weight: 800;
        font-size: calc(2.5rem + 1.2vw);
      }
  }
  .form{
    display: grid;
    .input{
      display: grid;
      font-size: calc(0.8rem + 0.6vw);
      label{
        font-size: calc(1rem + 0.4vw);
        font-weight: 800;
      }
    }
  }
  .submit{
    color: white;
    margin: auto;
    background: var(--main);
    max-width: 100px;
    min-height: 40px;
    background: var(--main);
    box-shadow: -3px 3px 0px var(--mainDark);
    border-radius: 1px;
    transition: 200ms ease-in-out;
    &:hover{
      background-color: var(--mainHover);
      box-shadow: -1px 1px 0px var(--mainDark);
    }
    &:active{
      box-shadow: 3px -3px 0px var(--mainDark) !important;
      transform: translateY(3px);
      background-color: var(--mainActive);
    }
  }
</style>

<script>
export default {
  data() {
    return {
      userInfo: {
        email: '6à9@1337jjj.com',
        password: 'lollol',
        password_confirmation: 'lollol',
        firstName: 'lollol',
        lastName: 'dzdz',
        username: 'lollojjjl'
      },
      show: true,
      errors: {},
      step: 1
    }
  },

  methods: {
    validateForm() {
      this.errors = {}

      // Email
      if (!this.userInfo.email) {
        this.errors.email = "Email is required"
      } else if (!/^\S+@\S+\.\S+$/.test(this.userInfo.email)) {
        this.errors.email = "Invalid email format"
      }

      // Username
      if (!this.userInfo.username) {
        this.errors.username = "Username is required"
      } else if (this.userInfo.username.length < 3) {
        this.errors.username = "Username must be at least 3 characters"
      }

      // First Name
      if (!this.userInfo.firstName) {
        this.errors.firstName = "First name is required"
      }

      // Last Name
      if (!this.userInfo.lastName) {
        this.errors.lastName = "Last name is required"
      }

      // Password
      if (!this.userInfo.password) {
        this.errors.password = "Password is required"
      } else if (this.userInfo.password.length < 6) {
        this.errors.password = "Password must be at least 6 characters"
      }

      // Password Confirmation
      if (this.userInfo.password !== this.userInfo.password_confirmation) {
        this.errors.password_confirmation = "Passwords do not match"
      }

      return Object.keys(this.errors).length === 0
    },

    async submitForm() {
      // ✅ Validate before API call
      if (!this.validateForm()) {
        this.notify([false, "Please fix the errors"])
        return
      }

      const loader = this.$loading.show()

      try {
        const res = await this.$axios.post('api/register', this.userInfo)

        if (res.data.success) {
          // Auto login
          await this.$auth.loginWith('local', {
            data: {
              username: this.userInfo.email,
              password: this.userInfo.password
            }
          })

          await this.load(this.$auth.user, this.$store)
          this.notify([true, "Welcome to chopshop."])
          this.step = 2
        } else {
          this.notify([false, "Registration failed"])
        }

      } catch (err) {
        const status = err.response?.status

        if ([422, 403, 404].includes(status)) {
          const message = status === 422
            ? err.response.data.message
            : err.response.data.data?.message

          const errors = status === 422
            ? err.response.data.errors
            : err.response.data.data?.errors

          this.notify([false, message || "Validation error"])
          this.errors = errors || {}

        } else {
          console.error(err)
          this.notify([false, "Something went wrong :O, contact us"])
        }

      } finally {
        loader.hide()
      }
    }
  }
}
</script>