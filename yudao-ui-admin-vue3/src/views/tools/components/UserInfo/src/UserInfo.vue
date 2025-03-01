<script lang="ts" setup>
import { ElMessageBox } from 'element-plus'
import { ArrowDown } from '@element-plus/icons-vue'

import avatarImg from '@/assets/imgs/avatar.gif'
import { useDesign } from '@/hooks/web/useDesign'
import { useTagsViewStore } from '@/store/modules/tagsView'
import { useUserStore } from '@/store/modules/user'
import LockDialog from './components/LockDialog.vue'
import LockPage from './components/LockPage.vue'
import { useLockStore } from '@/store/modules/lock'

defineOptions({ name: 'UserInfoTemp' })

const { t } = useI18n()

const { push, replace } = useRouter()

const userStore = useUserStore()

const tagsViewStore = useTagsViewStore()

const { getPrefixCls } = useDesign()

const prefixCls = getPrefixCls('user-info')

const avatar = computed(() => userStore.user.avatar || avatarImg)
const userName = computed(() => userStore.user.nickname ?? 'Admin')

// 锁定屏幕
const lockStore = useLockStore()
const getIsLock = computed(() => lockStore.getLockInfo?.isLock ?? false)
const dialogVisible = ref<boolean>(false)
const lockScreen = () => {
  dialogVisible.value = true
}

const loginOut = async () => {
  try {
    await ElMessageBox.confirm(t('common.loginOutMessage'), t('common.reminder'), {
      confirmButtonText: t('common.ok'),
      cancelButtonText: t('common.cancel'),
      type: 'warning'
    })
    await userStore.loginOut()
    tagsViewStore.delAllViews()
    replace('/login?redirect=/index')
  } catch {}
}
const toProfile = async () => {
  push('/user/profile')
}
const toDocument = () => {
  window.open('https://doc.iocoder.cn/')
}
</script>

<template>
  <ElDropdown class="user-dropdown !h-8 !px-4 !text-sm" :class="prefixCls" trigger="click">
    <div class="flex items-center cursor-pointer hover:text-[#3b82f6] transition-colors">
      <span class="text-sm font-medium">
        {{ userName }}
      </span>
      <el-icon class="ml-1 text-xs transition-transform duration-200">
        <ArrowDown />
      </el-icon>
    </div>
    <template #dropdown>
      <ElDropdownMenu
        class="!border-none !p-1.5 !rounded-xl !shadow-[0_0_20px_rgba(0,0,0,0.08)] !backdrop-blur-sm !bg-[#f8f9fa]/95"
      >
        <ElDropdownItem divided @click="loginOut" class="!rounded-lg !px-4 !py-2.5 !text-sm">
          <div
            class="flex items-center gap-2 text-[#4a5568] hover:text-[#3b82f6] transition-colors"
          >
            <Icon icon="ep:switch-button" />
            <span>{{ t('common.loginOut') }}</span>
          </div>
        </ElDropdownItem>
      </ElDropdownMenu>
    </template>
  </ElDropdown>

  <LockDialog v-if="dialogVisible" v-model="dialogVisible" />

  <teleport to="body">
    <transition name="fade-bottom" mode="out-in">
      <LockPage v-if="getIsLock" />
    </transition>
  </teleport>
</template>

<style lang="scss" scoped>
:deep(.el-dropdown-menu__item) {
  &:not(:last-child) {
    margin-bottom: 4px;
  }

  &:hover {
    background-color: rgb(243 244 246 / 90%);
  }
}

.user-dropdown {
  :deep(.el-dropdown-menu__item--divided) {
    &::before {
      display: none;
    }
  }

  :deep(.el-dropdown-menu) {
    border: 1px solid rgb(255 255 255 / 10%);
  }
}

.fade-bottom-enter-active,
.fade-bottom-leave-active {
  transition:
    opacity 0.25s,
    transform 0.3s;
}

.fade-bottom-enter-from {
  opacity: 0;
  transform: translateY(-10%);
}

.fade-bottom-leave-to {
  opacity: 0;
  transform: translateY(10%);
}
</style>
