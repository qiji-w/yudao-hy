<template>
  <el-form
    v-show="getShow"
    ref="formLogin"
    :model="loginData.loginForm"
    class="login-form bg-white rounded-lg shadow-xl w-[400px] p-30px"
    label-position="top"
  >
    <!-- 标题 -->
    <div class="text-center mb-20px">
      <h2 class="text-20px font-medium text-gray-900">User</h2>
    </div>

    <!-- 租户名称 -->
    <el-form-item v-if="loginData.tenantEnable === 'true'" prop="tenantName">
      <el-input
        v-model="loginData.loginForm.tenantName"
        :placeholder="t('login.tenantNamePlaceholder')"
        :prefix-icon="iconHouse"
      />
    </el-form-item>

    <!-- 用户名 -->
    <el-form-item label="Email" prop="username">
      <el-input
        v-model="loginData.loginForm.username"
        :placeholder="t('login.username')"
        :prefix-icon="iconAvatar"
      />
    </el-form-item>

    <!-- 密码 -->
    <el-form-item label="Password" prop="password" class="mb-15px">
      <el-input
        v-model="loginData.loginForm.password"
        type="password"
        :placeholder="t('login.password')"
        :prefix-icon="iconLock"
        show-password
        @keyup.enter="getCode()"
      />
    </el-form-item>

    <!-- 记住我 -->
    <el-form-item class="mb-20px">
      <el-checkbox v-model="loginData.loginForm.rememberMe">
        {{ t('login.remember') }}
      </el-checkbox>
    </el-form-item>

    <!-- 登录按钮 -->
    <el-button :loading="loginLoading" class="w-full login-btn" type="primary" @click="getCode">
      {{ t('login.login') }}
    </el-button>

    <!-- 验证码组件 -->
    <Verify
      v-if="loginData.captchaEnable === 'true'"
      ref="verify"
      :captchaType="captchaType"
      :imgSize="{ width: '400px', height: '200px' }"
      mode="pop"
      @success="handleLogin"
    />

    <!-- 底部链接 -->
    <div class="mt-20px flex justify-between text-14px">
      <a href="#" class="text-blue-400 hover:text-blue-500">{{ t('login.backHome') }}</a>
      <a
        href="javascript:;"
        class="text-blue-400 hover:text-blue-500"
        @click="setLoginState(LoginStateEnum.RESET_PASSWORD)"
      >
        {{ t('login.forgetPassword') }}
      </a>
    </div>
  </el-form>
</template>

<script lang="ts" setup>
import { ElLoading } from 'element-plus'
import LoginFormTitle from './LoginFormTitle.vue'
import type { RouteLocationNormalizedLoaded } from 'vue-router'
import { useIcon } from '@/hooks/web/useIcon'
import * as authUtil from '@/utils/auth'
import { usePermissionStore } from '@/store/modules/permission'
import * as LoginApi from '@/api/login'
import { LoginStateEnum, useFormValid, useLoginState } from './useLogin'

defineOptions({ name: 'LoginForm' })

const { t } = useI18n()
const message = useMessage()
const iconHouse = useIcon({ icon: 'ep:house' })
const iconAvatar = useIcon({ icon: 'ep:avatar' })
const iconLock = useIcon({ icon: 'ep:lock' })
const formLogin = ref()
const { validForm } = useFormValid(formLogin)
const { setLoginState, getLoginState } = useLoginState()
const { currentRoute, push } = useRouter()
const permissionStore = usePermissionStore()
const redirect = ref<string>('')
const loginLoading = ref(false)
const verify = ref()
const captchaType = ref('blockPuzzle') // blockPuzzle 滑块 clickWord 点击文字

const getShow = computed(() => unref(getLoginState) === LoginStateEnum.LOGIN)

const loginRules = {
  tenantName: [required],
  username: [required],
  password: [required]
}
const loginData = reactive({
  isShowPassword: false,
  captchaEnable: import.meta.env.VITE_APP_CAPTCHA_ENABLE,
  tenantEnable: import.meta.env.VITE_APP_TENANT_ENABLE,
  loginForm: {
    tenantName: import.meta.env.VITE_APP_DEFAULT_LOGIN_TENANT || '',
    username: '',
    password: '',
    captchaVerification: '',
    rememberMe: true // 默认记录我。如果不需要，可手动修改
  }
})

const socialList = [
  { icon: 'ant-design:wechat-filled', type: 30 },
  { icon: 'ant-design:dingtalk-circle-filled', type: 20 },
  { icon: 'ant-design:github-filled', type: 0 },
  { icon: 'ant-design:alipay-circle-filled', type: 0 }
]

// 获取验证码
const getCode = async () => {
  // 情况一，未开启：则直接登录
  if (loginData.captchaEnable === 'false') {
    await handleLogin({})
  } else {
    // 情况二，已开启：则展示验证码；只有完成验证码的情况，才进行登录
    // 弹出验证码
    verify.value.show()
  }
}
// 获取租户 ID
const getTenantId = async () => {
  if (loginData.tenantEnable === 'true') {
    const res = await LoginApi.getTenantIdByName(loginData.loginForm.tenantName)
    authUtil.setTenantId(res)
  }
}
// 记住我
const getLoginFormCache = () => {
  const loginForm = authUtil.getLoginForm()
  if (loginForm) {
    loginData.loginForm = {
      ...loginData.loginForm,
      username: loginForm.username ? loginForm.username : loginData.loginForm.username,
      password: loginForm.password ? loginForm.password : loginData.loginForm.password,
      rememberMe: loginForm.rememberMe,
      tenantName: loginForm.tenantName ? loginForm.tenantName : loginData.loginForm.tenantName
    }
  }
}
// 根据域名，获得租户信息
const getTenantByWebsite = async () => {
  const website = location.host
  const res = await LoginApi.getTenantByWebsite(website)
  if (res) {
    loginData.loginForm.tenantName = res.name
    authUtil.setTenantId(res.id)
  }
}
const loading = ref() // ElLoading.service 返回的实例
// 登录
const handleLogin = async (params: any) => {
  loginLoading.value = true
  try {
    await getTenantId()
    const data = await validForm()
    if (!data) {
      return
    }
    const loginDataLoginForm = { ...loginData.loginForm }
    loginDataLoginForm.captchaVerification = params.captchaVerification
    const res = await LoginApi.login(loginDataLoginForm)
    if (!res) {
      return
    }
    loading.value = ElLoading.service({
      lock: true,
      text: t('common.loading'),
      background: 'rgba(0, 0, 0, 0.7)'
    })
    if (loginDataLoginForm.rememberMe) {
      authUtil.setLoginForm(loginDataLoginForm)
    } else {
      authUtil.removeLoginForm()
    }
    authUtil.setToken(res)
    if (!redirect.value) {
      redirect.value = '/tools/email'
    }
    // 判断是否为SSO登录
    if (redirect.value.indexOf('sso') !== -1) {
      window.location.href = window.location.href.replace('/login?redirect=', '')
    } else {
      await push({ path: redirect.value || permissionStore.addRouters[0].path })
    }
  } finally {
    loginLoading.value = false
    loading.value.close()
  }
}

// 社交登录
const doSocialLogin = async (type: number) => {
  if (type === 0) {
    message.error('此方式未配置')
  } else {
    loginLoading.value = true
    if (loginData.tenantEnable === 'true') {
      // 尝试先通过 tenantName 获取租户
      await getTenantId()
      // 如果获取不到，则需要弹出提示，进行处理
      if (!authUtil.getTenantId()) {
        try {
          const data = await message.prompt('请输入租户名称', t('common.reminder'))
          if (data?.action !== 'confirm') throw 'cancel'
          const res = await LoginApi.getTenantIdByName(data.value)
          authUtil.setTenantId(res)
        } catch (error) {
          if (error === 'cancel') return
        } finally {
          loginLoading.value = false
        }
      }
    }
    // 计算 redirectUri
    // tricky: type、redirect需要先encode一次，否则钉钉回调会丢失。
    // 配合 Login/SocialLogin.vue#getUrlValue() 使用
    const redirectUri =
      location.origin +
      '/social-login?' +
      encodeURIComponent(`type=${type}&redirect=${redirect.value || '/'}`)

    // 进行跳转
    window.location.href = await LoginApi.socialAuthRedirect(type, encodeURIComponent(redirectUri))
  }
}
watch(
  () => currentRoute.value,
  (route: RouteLocationNormalizedLoaded) => {
    redirect.value = route?.query?.redirect as string
  },
  {
    immediate: true
  }
)
onMounted(() => {
  getLoginFormCache()
  getTenantByWebsite()
})
</script>

<style lang="scss" scoped>
:deep(.el-form-item__label) {
  padding-bottom: 8px;
  font-size: 14px;
  color: #374151;
}

:deep(.el-input__wrapper) {
  padding: 8px 12px;
  background-color: transparent;
  border: 1px solid #e5e7eb;
  border-radius: 4px;

  &:hover,
  &.is-focus {
    border-color: #3b82f6;
    box-shadow: 0 0 0 1px rgb(59 130 246 / 10%);
  }
}

:deep(.el-button) {
  height: 40px;
  font-size: 14px;
  font-weight: 500;
  background: linear-gradient(to right, #ec4899, #8b5cf6);
  border: none;
  border-radius: 4px;

  &:hover {
    opacity: 0.9;
  }
}

.login-btn {
  background: linear-gradient(to right, #ec4899, #8b5cf6);
  transition: all 0.3s ease;
}

// 背景样式
.login-form {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 10px 15px -3px rgb(0 0 0 / 10%);
}
</style>
