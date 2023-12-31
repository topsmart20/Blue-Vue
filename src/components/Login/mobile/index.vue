<script lang="ts">
import { defineComponent, reactive, toRefs, computed, watch } from "vue";
import { useI18n } from "vue-i18n";
import LoginHeader from "./Header.vue";
import { authStore } from "@/store/auth";
import { userStore } from "@/store/user";
import { socketStore } from "@/store/socket";
import Notification from "@/components/global/notification/index.vue";
import { storeToRefs } from "pinia";
import { onMounted } from "vue";
import { useDisplay } from "vuetify";
import { ElNotification } from "element-plus";
import SuccessIcon from "@/components/global/notification/SuccessIcon.vue";
import WarningIcon from "@/components/global/notification/WarningIcon.vue";
import { useToast } from "vue-toastification";

const Login = defineComponent({
  components: {
    LoginHeader,
    Notification,
    ElNotification,
    SuccessIcon,
    WarningIcon,
  },
  emits: ["close", "switch"],
  setup(props, { emit }) {
    // translation
    const { t } = useI18n();
    const { dispatchSignIn } = authStore();
    const { dispatchUserProfile } = authStore();
    const { setAuthModalType } = authStore();
    const { dispatchUserBalance } = userStore();
    const { setToken } = authStore();
    const { dispatchSocketConnect } = socketStore();
    const { width } = useDisplay();

    // initiate component state
    const state = reactive({
      currentPage: 0, // default login form
      PAGE_TYPE: {
        LOGIN_FORM: 0,
        FORGOT_PASSWORD: 1,
      },
      formData: {
        emailAddress: "",
        password: "",
      },
      socialIconList: [
        new URL("@/assets/public/svg/icon_public_28.svg", import.meta.url).href,
        new URL("@/assets/public/svg/icon_public_29.svg", import.meta.url).href,
        new URL("@/assets/public/svg/icon_public_30.svg", import.meta.url).href,
        new URL("@/assets/public/svg/icon_public_31.svg", import.meta.url).href,
      ],
      isShowPassword: false,
      notificationShow: false,
      checkIcon: new URL("@/assets/public/svg/icon_public_18.svg", import.meta.url).href,
      notificationText: t("login.forgotPasswordPage.notification"),
      loading: false,
      mailCardHeight: 0,
      emailPartName: "",
      closeBtnHeight: 0,
      closeBtnShow: false,
      containerHeight: 0 as number | undefined,
      bodyHeight: 0,
    });

    const mobileWidth = computed(() => {
      return width.value;
    });

    // computed variables
    const isFormDataReady = computed(
      (): boolean =>
        state.formData.emailAddress.length > 0 && state.formData.password.length > 0
    );

    // flag when login successed
    const success = computed(() => {
      const { getSuccess } = storeToRefs(authStore());
      return getSuccess.value;
    });

    // error message when login failed

    const errMessage = computed(() => {
      const { getErrMessage } = storeToRefs(authStore());
      return getErrMessage.value;
    });

    const dialogCheckbox = computed(() => {
      const { getDialogCheckbox } = storeToRefs(authStore());
      return getDialogCheckbox.value;
    });

    watch(
      dialogCheckbox,
      (newValue) => {
        state.closeBtnShow = false;
      },
      { deep: true }
    );

    // forgot password function when password fogot

    const handleForgotPassword = () => {
      console.log("???????????????????????");
      const toast = useToast();
      toast.success(t("login.forgotPasswordPage.notification"), {
        timeout: 3000,
        closeOnClick: false,
        pauseOnFocusLoss: false,
        pauseOnHover: false,
        draggable: false,
        showCloseButtonOnHover: false,
        hideProgressBar: true,
        closeButton: "button",
        icon: SuccessIcon,
        rtl: false,
      });
      state.currentPage = state.PAGE_TYPE.LOGIN_FORM;
      // state.notificationShow = !state.notificationShow;
      // state.checkIcon = new URL(
      //   "@/assets/public/svg/icon_public_18.svg",
      //   import.meta.url
      // ).href;
      // state.notificationText = t("login.forgotPasswordPage.notification");
    };

    // methods
    const handleLoginFormSubmit = async () => {
      state.loading = true;

      // setToken("token");
      // state.notificationShow = !state.notificationShow;
      // state.checkIcon = new URL("@/assets/public/svg/icon_public_18.svg", import.meta.url).href
      // state.notificationText = t('login.submit_result.success_text')
      // setTimeout(() => {
      //     setAuthModalType("");
      //     emit('close');
      // }, 1000)

      // return;

      await dispatchSignIn({
        uid: state.formData.emailAddress,
        password: state.formData.password,
      });

      if (success.value) {
        await dispatchUserProfile();
        await dispatchUserBalance();
        await dispatchSocketConnect();
        const toast = useToast();
        toast.success(t("login.submit_result.success_text"), {
          timeout: 3000,
          closeOnClick: false,
          pauseOnFocusLoss: false,
          pauseOnHover: false,
          draggable: false,
          showCloseButtonOnHover: false,
          hideProgressBar: true,
          closeButton: "button",
          icon: SuccessIcon,
          rtl: false,
        });
        // state.notificationShow = !state.notificationShow;
        // state.checkIcon = new URL(
        //   "@/assets/public/svg/icon_public_18.svg",
        //   import.meta.url
        // ).href;
        // state.notificationText = t("login.submit_result.success_text");
        setTimeout(() => {
          setAuthModalType("");
          emit("close");
        }, 100);
      } else {
        const toast = useToast();
        toast.success(t("login.submit_result.err_text"), {
          timeout: 3000,
          closeOnClick: false,
          pauseOnFocusLoss: false,
          pauseOnHover: false,
          draggable: false,
          showCloseButtonOnHover: false,
          hideProgressBar: true,
          closeButton: "button",
          icon: WarningIcon,
          rtl: false,
        });
        // state.notificationShow = !state.notificationShow;
        // state.checkIcon = new URL(
        //   "@/assets/public/svg/icon_public_17.svg",
        //   import.meta.url
        // ).href;
        // state.notificationText = t("login.submit_result.err_text");
      }

      state.loading = false;
    };

    const showPassword = () => {
      state.isShowPassword = !state.isShowPassword;
    };

    const handleEmailBlur = () => {
      // console.log("onblur")
      setTimeout(() => {
        state.mailCardHeight = 0;
      }, 100);
    };

    const handleEmailChange = () => {
      // console.log("onchange")
      if (state.formData.emailAddress.includes("@")) {
        state.emailPartName = state.formData.emailAddress.split("@")[0];
        state.mailCardHeight = 220;
      } else {
        setTimeout(() => {
          state.mailCardHeight = 0;
        }, 100);
      }
    };

    const handleEmailFocus = () => {
      // console.log("onFocus")
      if (state.formData.emailAddress.includes("@")) {
        state.emailPartName = state.formData.emailAddress.split("@")[0];
        state.mailCardHeight = 220;
      }
    };

    const mergeEmail = (mail: string) => {
      state.formData.emailAddress = state.formData.emailAddress.split("@")[0] + mail;
      setTimeout(() => {
        state.mailCardHeight = 0;
      }, 100);
    };

    watch(
      mobileWidth,
      (newValue) => {
        state.containerHeight = window.innerHeight - 54;
        state.bodyHeight = window.innerHeight - 194;
      },
      { deep: true }
    );

    const handleResize = () => {
      if (window.visualViewport?.height != undefined) {
        state.containerHeight = window.visualViewport?.height - 54;
        state.bodyHeight = window.innerHeight - 194;
      }
    };

    onMounted(() => {
      if (window.visualViewport?.height != undefined) {
        state.containerHeight = window.visualViewport?.height - 54;
        state.bodyHeight = window.innerHeight - 194;
      }
      window.addEventListener("resize", handleResize);
      // setTimeout(() => {
      //   state.closeBtnShow = true;
      // }, 300);
    });

    return {
      t,
      ...toRefs(state),
      isFormDataReady,
      handleLoginFormSubmit,
      handleForgotPassword,
      showPassword,
      handleEmailBlur,
      handleEmailChange,
      handleEmailFocus,
      mergeEmail,
    };
  },
});

export default Login;
</script>

<template>
  <div class="m-login-container" :style="{ height: containerHeight + 'px' }">
    <LoginHeader v-if="currentPage === PAGE_TYPE.LOGIN_FORM" />
    <div
      class="m-login-body px-6"
      :style="{
        height:
          currentPage == PAGE_TYPE.FORGOT_PASSWORD
            ? containerHeight + 'px'
            : bodyHeight + 'px',
      }"
    >
      <!-- SIGN UP FORM  -->
      <v-form v-if="currentPage === PAGE_TYPE.LOGIN_FORM" ref="form" class="full-width">
        <v-row class="relative mt-8">
          <v-text-field
            :label="t('signup.formPage.emailAddress')"
            class="form-textfield dark-textfield m-login-email"
            variant="solo"
            density="comfortable"
            v-model="formData.emailAddress"
            :onblur="handleEmailBlur"
            @input="handleEmailChange"
            :onfocus="handleEmailFocus"
          />
        </v-row>
        <div class="m-login-mail-card" :style="{ height: mailCardHeight + 'px' }">
          <v-list theme="dark" bg-color="#211F31">
            <v-list-item
              class="text-600-12 white"
              value="gmail"
              @click="mergeEmail('@gmail.com')"
            >
              {{ emailPartName }}@gmail.com
            </v-list-item>
            <v-list-item
              class="text-600-12 white"
              value="hotmail"
              @click="mergeEmail('@hotmail.com')"
              >{{ emailPartName }}@hotmail.com</v-list-item
            >
            <v-list-item
              class="text-600-12 white"
              value="yahoo"
              @click="mergeEmail('@yahoo.com')"
              >{{ emailPartName }}@yahoo.com</v-list-item
            >
            <v-list-item
              class="text-600-12 white"
              value="icloud"
              @click="mergeEmail('@icloud.com')"
              >{{ emailPartName }}@icloud.com</v-list-item
            >
            <v-list-item
              class="text-600-12 white"
              value="outlook"
              @click="mergeEmail('@outlook.com')"
              >{{ emailPartName }}@outlook.com</v-list-item
            >
          </v-list>
        </div>
        <div class="mt-6 relative pa-0">
          <v-text-field
            :label="t('signup.formPage.password')"
            class="form-textfield dark-textfield ma-0 m-login-password"
            variant="solo"
            density="comfortable"
            :type="isShowPassword ? 'text' : 'password'"
            v-model="formData.password"
          />
          <img
            v-if="isShowPassword"
            src="@/assets/public/svg/icon_public_07.svg"
            class="m-disable-password"
            @click="showPassword"
            width="16"
          />
          <img
            v-else
            src="@/assets/public/svg/icon_public_06.svg"
            class="m-disable-password"
            @click="showPassword"
            width="16"
          />
        </div>
        <v-row class="mt-1">
          <p
            class="ml-9 login-forget-passwrod-text text-400-12"
            @click="currentPage = PAGE_TYPE.FORGOT_PASSWORD"
          >
            {{ t("login.formPage.forgetPassword") }}
          </p>
        </v-row>
        <v-row style="margin-top: 116px">
          <v-btn
            class="ma-3 button-bright m-signin-btn-text"
            width="-webkit-fill-available"
            height="48px"
            :loading="loading"
            :disabled="!isFormDataReady"
            :onclick="handleLoginFormSubmit"
          >
            {{ t("login.formPage.button") }}
          </v-btn>
        </v-row>
        <v-row class="mt-4">
          <p class="m-divide-text">
            {{ t("signup.formPage.divider") }}
          </p>
          <v-divider color="white" />
        </v-row>
        <v-row class="mt-6">
          <v-col cols="8" offset="2">
            <div
              class="d-flex justify-space-around bg-surface-variant social-icon-wrapper"
            >
              <v-sheet
                v-for="(item, index) in socialIconList"
                :key="index"
                color="#131828"
                class="rounded"
              >
                <v-btn
                  color="grey-darken-4"
                  class="m-social-icon-button"
                  icon=""
                  width="36px"
                  height="36px"
                >
                  <img :src="item" width="36" />
                </v-btn>
              </v-sheet>
            </div>
          </v-col>
        </v-row>
      </v-form>
      <!-- Forgot password -->
      <div v-if="currentPage == PAGE_TYPE.FORGOT_PASSWORD" class="full-width">
        <v-row class="mt-10 d-flex justify-center">
          <img src="@/assets/public/image/logo_public_01.png" width="182" />
          <!-- <span class="logo-text purple text-large">{{ t('logo_text_1') }}</span>
                    <span class="logo-text yellow text-large">{{ t('main.logo_text_2') }}</span> -->
        </v-row>
        <v-row class="mt-1 justify-center">
          <!-- <p class="label-text-md2 white center full-width pl-12 pr-12">
                        {{ t('login.forgotPasswordPage.title') }}
                    </p> -->
          <p class="m-logo-text2 center white">
            {{ t("signup.formPage.header.titleLine1") }}
            <br />
            {{ t("signup.formPage.header.titleLine2") }}
          </p>
        </v-row>
        <v-row class="relative mt-8">
          <v-text-field
            :label="t('signup.formPage.emailAddress')"
            class="form-textfield dark-textfield m-login-forgot"
            variant="solo"
            density="comfortable"
            v-model="formData.emailAddress"
            :onblur="handleEmailBlur"
            @input="handleEmailChange"
            :onfocus="handleEmailFocus"
          />
          <div class="m-forgot-mail-card" :style="{ height: mailCardHeight + 'px' }">
            <v-list theme="dark" bg-color="#211F31">
              <v-list-item
                class="text-600-12 white"
                value="gmail"
                @click="mergeEmail('@gmail.com')"
              >
                {{ emailPartName }}@gmail.com
              </v-list-item>
              <v-list-item
                class="text-600-12 white"
                value="hotmail"
                @click="mergeEmail('@hotmail.com')"
                >{{ emailPartName }}@hotmail.com</v-list-item
              >
              <v-list-item
                class="text-600-12 white"
                value="yahoo"
                @click="mergeEmail('@yahoo.com')"
                >{{ emailPartName }}@yahoo.com</v-list-item
              >
              <v-list-item
                class="text-600-12 white"
                value="icloud"
                @click="mergeEmail('@icloud.com')"
                >{{ emailPartName }}@icloud.com</v-list-item
              >
              <v-list-item
                class="text-600-12 white"
                value="outlook"
                @click="mergeEmail('@outlook.com')"
                >{{ emailPartName }}@outlook.com</v-list-item
              >
            </v-list>
          </div>
        </v-row>
        <v-row style="margin-top: 200px">
          <v-btn
            class="ma-3 button-bright m-signin-btn-text"
            width="-webkit-fill-available"
            height="48"
            autocapitalize="off"
            @click="handleForgotPassword"
          >
            {{ t("login.forgotPasswordPage.submit") }}
          </v-btn>
        </v-row>
        <v-row class="mt-4">
          <p class="m-divide-text">
            {{ t("signup.formPage.divider") }}
          </p>
          <v-divider color="white" />
        </v-row>
        <v-row class="mt-6">
          <v-col cols="8" offset="2">
            <div
              class="d-flex justify-space-around bg-surface-variant social-icon-wrapper"
            >
              <v-sheet
                v-for="(item, index) in socialIconList"
                :key="index"
                color="#131828"
                class="rounded"
              >
                <v-btn
                  color="grey-darken-4"
                  class="m-social-icon-button"
                  icon=""
                  width="36px"
                  height="36px"
                >
                  <img :src="item" width="36" />
                </v-btn>
              </v-sheet>
            </div>
          </v-col>
        </v-row>
      </div>
    </div>
    <v-btn
      class="m-close-button"
      icon="true"
      @click="$emit('close')"
      width="30"
      height="30"
      v-if="closeBtnShow"
    >
      <img src="@/assets/public/svg/icon_public_10.svg" />
      <!-- <v-icon :color="currentPage !== PAGE_TYPE.LOGIN_FORM ? '#7782AA' : '#FFFFFF'">
                mdi-close
            </v-icon> -->
    </v-btn>
    <Notification
      :notificationShow="notificationShow"
      :notificationText="notificationText"
      :checkIcon="checkIcon"
    />
  </div>
</template>

<style lang="scss">
.m-login-mail-card {
  position: absolute;
  top: 86px;
  left: 50%;
  transform: translateX(-50%);
  background: #211f31;
  width: calc(100% - 48px);
  border-radius: 16px;
  z-index: 200;
  overflow: hidden;
  transition: height 0.3s ease-out;
}

.m-forgot-mail-card {
  position: absolute;
  top: 55px;
  left: 50%;
  transform: translateX(-50%);
  background: #211f31;
  width: calc(100% - 48px);
  border-radius: 16px;
  z-index: 200;
  overflow: hidden;
  transition: height 0.3s ease-out;
}

.v-overlay__content {
  overflow-y: unset !important;
}

.m-signin-btn-text {
  .v-btn__content {
    text-align: center;
    font-family: "Inter";
    font-size: 14px;
    font-style: normal;
    font-weight: 700;
    line-height: normal;
  }
}

.m-disable-password {
  position: absolute;
  top: 16px;
  right: 24px;
  cursor: pointer;
}

// button:active:enabled {
//     transform: scale(0.95);
//     filter: brightness(80%);
// }

// container
.m-login-container {
  height: 613px;
  border-radius: 26px 26px 0px 0px;
  position: absolute;
  bottom: 0;
  width: 100%;

  .v-field--variant-solo {
    background: transparent !important;
  }
}

// wrapper
.m-login-body {
  border-radius: 26px 26px 0px 0px;
  background: var(--bg-2-e-274-c, #2e274c);
  position: absolute;
  bottom: 0px;
  width: 100%;
  height: 473px;
  z-index: 2000;
  overflow-y: auto;
  .form-textfield div.v-field__field {
    box-shadow: 2px 0px 4px 1px rgba(0, 0, 0, 0.12) inset !important;
  }
}

.m-login-body::-webkit-scrollbar {
  width: 0px;
}

.m-login-body-height {
  height: 613px !important;
}

// close modal button
.m-close-button {
  box-shadow: none !important;
  background-color: transparent !important;
  position: absolute !important;
  top: -53px;
  right: 5px;
}

// divider
.divide-text {
  font-family: "Inter";
  font-style: normal;
  font-weight: 500;
  font-size: 14px;
  color: #414968;
  position: relative;
  top: 12px;
  text-align: center;
  width: 120px;
  background-color: #2e274c;
  margin: auto;
  z-index: 1;
}

// social icon list component
.social-icon-wrapper {
  background-color: #2e274c !important;

  .v-sheet {
    border-radius: 50px !important;
  }
}

.m-social-icon-button {
  background-color: #131828 !important;
  width: 36px;
  height: 36px;
}

.login-forget-passwrod-text {
  cursor: pointer;
  font-family: "Inter";
  font-style: normal;
  font-weight: 400;
  font-size: 14px;
  line-height: 17px;
  color: #7782aa;
}

.text-large {
  font-size: 32px !important;
}

.m-login-email {
  transform-origin: top !important;

  .v-field__field {
    .v-label.v-field-label {
      font-family: "Inter";
      font-size: 12px !important;
      font-style: normal;
      font-weight: 400;
      line-height: normal;
      color: #7782aa !important;
      opacity: 1 !important;
    }

    .v-label.v-field-label--floating {
      --v-field-label-scale: 0.88em;
      font-size: 10px !important;
      max-width: 100%;
      color: #7782aa !important;
      opacity: 1 !important;
    }
  }
}

.m-login-password {
  transform-origin: top !important;

  .v-field__field {
    .v-label.v-field-label {
      font-family: "Inter";
      font-size: 12px !important;
      font-style: normal;
      font-weight: 400;
      line-height: normal;
      color: #7782aa !important;
      opacity: 1 !important;
    }

    .v-label.v-field-label--floating {
      --v-field-label-scale: 0.88em;
      font-size: 10px !important;
      max-width: 100%;
      color: #7782aa !important;
      opacity: 1 !important;
    }
  }
}

.m-login-forgot {
  transform-origin: top !important;

  .v-field__field {
    .v-label.v-field-label {
      font-family: "Inter";
      font-size: 12px !important;
      font-style: normal;
      font-weight: 400;
      line-height: normal;
      color: #7782aa !important;
      opacity: 1 !important;
    }

    .v-label.v-field-label--floating {
      --v-field-label-scale: 0.88em;
      font-size: 10px !important;
      max-width: 100%;
      color: #7782aa !important;
      opacity: 1 !important;
    }
  }
}
</style>
