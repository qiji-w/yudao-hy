<template>
  <div
    :class="prefixCls"
    class="relative h-[100vh] w-full bg-gradient-to-br from-teal-500 to-purple-600"
  >
    <div class="relative mx-auto h-full flex items-center justify-center">
      <!-- 登录表单容器 -->
      <LoginForm />
    </div>
  </div>
</template>

<script lang="ts" setup>
import { underlineToHump } from '@/utils'
import { useDesign } from '@/hooks/web/useDesign'
import { useAppStore } from '@/store/modules/app'
import { ThemeSwitch } from '@/layout/components/ThemeSwitch'
import { LocaleDropdown } from '@/layout/components/LocaleDropdown'
import {
  LoginForm,
  MobileForm,
  QrCodeForm,
  RegisterForm,
  SSOLoginVue,
  ForgetPasswordForm
} from './components'

defineOptions({ name: 'Login' })

const { t } = useI18n()
const appStore = useAppStore()
const { getPrefixCls } = useDesign()
const prefixCls = getPrefixCls('login')
</script>

<style lang="scss" scoped>
$prefix-cls: #{$namespace}-login;

.#{$prefix-cls} {
  &::before {
    position: absolute;
    inset: 0;
    background: linear-gradient(135deg, rgb(0 0 0 / 10%) 0%, rgb(255 255 255 / 10%) 100%);
    content: '';
    backdrop-filter: blur(10px);
  }
}
</style>

<style lang="scss">
.dark .login-form {
  .el-divider__text {
    background-color: var(--login-bg-color);
  }

  .el-card {
    background-color: var(--login-bg-color);
  }
}
</style>
