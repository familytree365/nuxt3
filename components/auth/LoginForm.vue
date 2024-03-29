<template>
  <AuthIndex>
    <template #form>
        <ValidationObserver slim v-slot="{ handleSubmit }">
        <form
          @submit.prevent="handleSubmit(submit)"
          class="
            column
            is-6-tablet
            is-7-desktop
            is-7-widescreen
            is-7-fullhd
            is-gapless
            is-flex
            ai--c
          "
        >
            <div class="auth-form is-gapless">
            <div>
                <div class="mb-5">
                <NuxtLink
                    to="/"
                    class="
                    is-size-6 is-flex
                    has-text-link has-text-weight-medium
                    mb-2
                    "
                >
                    <font-awesome-icon :icon="['fas', 'angle-left']" class="mt-1 mr-2" />
                        Back to Home
                </NuxtLink>
                <h1 class="is-size-4 has-text-black has-text-weight-bold">
                    Sign in to your account
                </h1>
                </div>

                <!-- <div
                v-for="(error, index) in errors"
                :key="index"
                class="notification is-danger"
                >
                {{ error[0] }}
                </div> -->

                <div class="mb-5">
                <div class="field">
                    <ValidationProvider
                    name="Email"
                    ref="email"
                    rules="required|email"
                    v-slot="{ errors }"
                    >
                    <p class="control has-icons-left has-icons-right">
                        <input
                        class="input"
                        :class="{ 'is-danger': errors[0] }"
                        type="email"
                        placeholder="Email address"
                        v-model="email"
                        />
                    </p>

                    <p
                        v-if="errors[0]"
                        class="has-text-danger p-2"
                        v-text="errors[0]"
                    ></p>
                    </ValidationProvider>
                </div>
                </div>
                <div class="mb-5">
                <div class="field">
                    <ValidationProvider
                    name="Password"
                    ref="password"
                    v-slot="{ errors }"
                    rules="required"
                    >
                    <p class="control has-icons-left has-icons-right">
                        <input
                        class="input"
                        :class="{ 'is-danger': errors[0] }"
                        type="password"
                        placeholder="Password"
                        v-model="password"
                        />
                    </p>

                    <p
                        v-if="errors[0]"
                        class="has-text-danger p-2"
                        v-text="errors[0]"
                    ></p>
                    </ValidationProvider>
                </div>
                </div>
                <div class="mb-5">
                <div class="columns is-mobile is-gapless">
                    <div class="column">
                    <label class="checkbox">
                        <input
                        v-model="remember"
                        class="mr-1"
                        type="checkbox"
                        />
                        Remember me
                    </label>
                    </div>
                    <div class="column has-text-right">
                    <NuxtLink
                        to="/forgotpassword"
                        class="has-text-link has-text-weight-medium"
                    >
                        Forgot password?
                    </NuxtLink>
                    </div>
                </div>
                </div>
                <div class="mb-6">
                <button
                    class="
                    button
                    theme-button theme-button-xl
                    has-background-primary
                    is-uppercase
                    has-text-weight-medium has-text-white
                    "
                >
                    Login
                </button>
                </div>
                <div>
                <p
                    class="
                    is-size-6
                    has-text-dark has-text-centered has-text-weight-regular
                    "
                >
                    Don't have an account?
                    <NuxtLink
                    to="/register"
                    class="has-text-link has-text-weight-medium"
                    >
                    Register Now
                    </NuxtLink>
                </p>
                </div>
            </div>
            <div class="divider">or</div>
            <a
                @click="loginSocial('google')"
                href="javascript:;"
                class="btn cnt-g"
            >
                <img src="~assets/images/google.jpg" />
                Continue with google
            </a>
            <a
                @click="loginSocial('facebook')"
                href="javascript:;"
                class="btn cnt-g"
            >
                <img src="~assets/images/facebook.png" />
                Continue with Facebook
            </a>
            <a
                @click="loginSocial('github')"
                href="javascript:;"
                class="btn cnt-g mb-5"
            >
                <img src="~assets/images/github.png" />
                Continue with Github
            </a>
            </div>
        </form>
        </ValidationObserver>
    </template>

    <template #footerImageForm>
      <img class="auth-img" src="~assets/images/mockup01@2x.webp" alt="" />
    </template>
  </AuthIndex>
</template>
<script>
  import AuthIndex from "~/components/auth/Index.vue";
  import { mapState, mapGetters } from "vuex";
  import { library } from "@fortawesome/fontawesome-svg-core";
  import {
    faEnvelope,
    faCheck,
    faExclamationTriangle,
    faLock,
    faUser,
  } from "@fortawesome/free-solid-svg-icons";
  import { focus } from "@enso-ui/directives";
  import Errors from "@enso-ui/laravel-validation";
  import RevealPassword from "@enso-ui/forms/src/bulma/parts/RevealPassword.vue";
  import { ref, computed, useStore, watch } from 'vue';

  library.add([faEnvelope, faCheck, faExclamationTriangle, faLock, faUser]);

  export default {
    name: "LoginForm",
    components: { AuthIndex },
    directives: { focus },
    props: {
      action: {
        required: true,
        type: String,
      },
      route: {
        required: true,
        type: String,
      },
    },
    setup(){
      const errors = new Errors()
      const provider = ref(null)
      const email = ref('')
      const password = ref('')
      const remember = ref(false)
      const device_name = ref('mac')
      const store = useStore()
      return {
        one: computed(() => store.getters['${isWebview}']),
        two: computed(() => store.state[meta])
      }
      const hasPassword = computed(() => {
        return this.password   != null && this.password
      })
      const match = computed(() => {
        return this.hasPassword && this.password === this.password_confirmation
      })
      const postParams = computed(() => {
        return this.loginParams
      })
      const loginParams = computed(() => {
        const {
          email,
          password,
          remember
        } = this;

        return {
          email,
          password,
          remember
        };
      })
      const loginLink = computed(() => {
        return "api/login";
      })
      const config = computed(() => {
        return this.isWebview ? { headers: { isWebview: true } } : {};
      })
      function validate() {
        this.$refs.email.validate().then((res) => {
          console.log(res.valid);
        })
      }
      function loginSocial(provider) {
        this.provider = provider
        window.location.href = `${process.env.baseUrl}/api/login/${provider}`;    
      }
      function submit() {
         this.loading = true;
          this.isSuccessful = false;
          this.oldLogin()
      }
      function oldLogin() {
        this.$axios
        .get('/sanctum/csrf-cookie').then(() =>{
          this.$axios.post('/api/login', {
            email: this.email,
            password: this.password,
          }, this.config)
            .then(({ data }) => {
              this.loading = false;
              this.isSuccessful = true;
              this.$emit("success", data);
            })
            .catch((error) => {
              this.loading = false;
              const { status, data } = error.response;
              switch (status) {
                case 422:
                  this.errorInput = data.errors;
                  break;
                case 429:
                  // this.$toastr.error(data.message); // error Toastr can't displayed
                  this.message = data.message;
                  break;
                case 500:
                  this.message = data.message;
                  this.$forceUpdate();
                  break;
                default:
                  throw error;
              }
            });
        })
      }
    }
  };

  function openWindow(url, title, options = {}) {
    if (typeof url === 'object') {
        options = url
        url = ''
    }
    options = {
        url,
        title,
        width: 600,
        height: 720,
        ...options
    }
    const dualScreenLeft = window.screenLeft !== undefined ? window.screenLeft : window.screen.left
    const dualScreenTop = window.screenTop !== undefined ? window.screenTop : window.screen.top
    const width = window.innerWidth || document.documentElement.clientWidth || window.screen.width
    const height = window.innerHeight || document.documentElement.clientHeight || window.screen.height
    options.left = ((width / 2) - (options.width / 2)) + dualScreenLeft
    options.top = ((height / 2) - (options.height / 2)) + dualScreenTop
    const optionsStr = Object.keys(options).reduce((acc, key) => {
        acc.push(`${key}=${options[key]}`)
        return acc
    }, []).join(',')
    const newWindow = window.open(url, title, optionsStr)
    if (window.focus) {
        newWindow.focus()
    }
    // console.log(newWindow);
    return newWindow
}
</script>

<style scoped>
  @import '~/assets/css/base.css';
</style>