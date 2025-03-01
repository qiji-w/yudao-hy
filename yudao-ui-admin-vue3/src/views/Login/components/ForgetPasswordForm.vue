<template>
  <el-dialog
    :title="t('login.resetPassword')"
    v-model="visible"
    width="400px"
    :close-on-click-modal="false"
    @close="handleClose"
  >
    <el-form ref="formRef" :model="formData" :rules="formRules" label-width="80px">
      <el-form-item :label="t('login.username')" prop="username">
        <el-input v-model="formData.username" :placeholder="t('login.usernamePlaceholder')" />
      </el-form-item>

      <el-form-item :label="t('login.email')" prop="email">
        <el-input v-model="formData.email" :placeholder="t('login.email')" />
      </el-form-item>

      <el-form-item :label="t('login.password')" prop="password">
        <el-input
          v-model="formData.password"
          type="password"
          show-password
          :placeholder="t('login.passwordPlaceholder')"
        />
      </el-form-item>

      <el-form-item :label="t('login.checkPassword')" prop="confirmPassword">
        <el-input
          v-model="formData.confirmPassword"
          type="password"
          show-password
          :placeholder="t('login.checkPassword')"
        />
      </el-form-item>
    </el-form>

    <template #footer>
      <el-button @click="handleClose">{{ t('common.cancel') }}</el-button>
      <el-button type="primary" :loading="loading" @click="handleSubmit">
        {{ t('common.ok') }}
      </el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { ref, reactive } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'
import { ElMessage } from 'element-plus'
import { useI18n } from '@/hooks/web/useI18n'
import * as LoginApi from '@/api/login'

const { t } = useI18n()
const emit = defineEmits(['close'])

const visible = ref(true)
const loading = ref(false)
const formRef = ref<FormInstance>()

const formData = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
})

const validateConfirmPassword = (rule: any, value: string, callback: any) => {
  if (value !== formData.password) {
    callback(new Error(t('login.diffPwd')))
  } else {
    callback()
  }
}

const formRules: FormRules = {
  username: [{ required: true, message: t('login.usernamePlaceholder'), trigger: 'blur' }],
  email: [
    { required: true, message: t('login.email'), trigger: 'blur' },
    { type: 'email', message: t('profile.rules.truemail'), trigger: ['blur', 'change'] }
  ],
  password: [{ required: true, message: t('login.passwordPlaceholder'), trigger: 'blur' }],
  confirmPassword: [
    { required: true, message: t('login.checkPassword'), trigger: 'blur' },
    { validator: validateConfirmPassword, trigger: 'blur' }
  ]
}

const handleClose = () => {
  emit('close')
}

const handleSubmit = async () => {
  if (!formRef.value) return

  try {
    await formRef.value.validate()
    loading.value = true

    await LoginApi.smsResetPassword(formData)
    ElMessage.success(t('login.resetPasswordSuccess'))
    handleClose()
  } catch (error: any) {
    console.error('Reset password error:', error)
    ElMessage.error(error.message || t('sys.api.operationFailed'))
  } finally {
    loading.value = false
  }
}
</script>
