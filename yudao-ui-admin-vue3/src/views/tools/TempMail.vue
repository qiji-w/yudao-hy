<template>
  <div class="temp-mail-wrapper">
    <!-- 顶部导航 -->
    <header class="absolute top-0 left-0 right-0 py-4">
      <div class="max-w-[1200px] mx-auto flex justify-between items-center px-6">
        <div class="flex items-center">
          <img src="@/assets/imgs/email/mail_logo.png" alt="logo" class="h-6" />
        </div>
        <div class="flex items-center gap-6">
          <el-dropdown trigger="click">
            <span class="text-sm font-medium cursor-pointer flex items-center">
              English
              <el-icon class="ml-1"><ArrowDown /></el-icon>
            </span>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item @click="handleSetLanguage('en')">English</el-dropdown-item>
                <el-dropdown-item @click="handleSetLanguage('zh-CN')">中文</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
          <span class="text-sm font-medium cursor-pointer">Pricing</span>
          <!-- <el-button class="login-btn !h-8 !px-4 !text-sm" round @click="login">Login</el-button> -->
          <UserInfo />
        </div>
      </div>
    </header>

    <!-- 主要内容区域 -->
    <main class="min-h-screen">
      <!-- 邮箱地址区域 -->
      <div class="text-center pt-32 pb-20">
        <h1
          class="text-[56px] font-bold mb-6 bg-gradient-to-r from-[#ec4899] to-[#8b5cf6] bg-clip-text text-transparent"
        >
          {{ t('email.title') }}
        </h1>
        <p class="text-lg mb-16">
          <span class="font-bold text-[#1a1a1a]"
            >No registration required, safe and anonymous.</span
          >
          <span class="text-[#64748b] ml-1">Say goodbye to spam and protect your privacy.</span>
        </p>

        <!-- 邮箱输入框 -->
        <div class="email-container">
          <div
            class="email-input-container bg-white rounded-[40px] shadow-[0_0_20px_rgba(0,0,0,0.08)] flex items-center"
          >
            <button
              class="w-11 h-11 flex items-center justify-center hover:bg-gray-50 rounded-full ml-1 transition-colors"
              @click="refreshEmail"
            >
              <el-icon class="text-xl text-[#64748b]"><Refresh /></el-icon>
            </button>
            <input
              v-model="emailAddress"
              readonly
              class="flex-1 h-12 px-4 text-base text-[#1a1a1a] border-none focus:outline-none bg-transparent"
              :placeholder="loading ? 'Generating...' : 'Click refresh to generate new email'"
            />
            <button
              class="w-11 h-11 flex items-center justify-center hover:bg-gray-50 rounded-full mr-1 transition-colors"
              @click="copyEmail"
            >
              <el-icon class="text-xl text-[#64748b]"><CopyDocument /></el-icon>
            </button>
          </div>
        </div>
      </div>

      <!-- 收件箱区域 -->
      <div class="email-container">
        <div class="bg-white rounded-[24px] shadow-[0_0_20px_rgba(0,0,0,0.08)] overflow-hidden">
          <div class="flex items-center justify-between px-6 py-4 border-b border-[#f1f5f9]">
            <h2 class="text-xl font-bold text-[#1a1a1a]">Inbox</h2>
            <button
              class="px-4 py-1.5 bg-[#1a1a1a] text-white text-sm rounded-full hover:opacity-90 transition-opacity"
              @click="refreshInbox"
            >
              Refresh
            </button>
          </div>

          <div v-if="!emailList.length" class="py-40 text-center">
            <p class="text-[#94a3b8] text-lg">No Email</p>
          </div>

          <!-- 邮件列表 -->
          <el-table v-else :data="emailList" class="custom-table">
            <el-table-column label="From" min-width="200">
              <template #default="{ row }">
                <span class="text-[#1a1a1a]">{{ row.from }}</span>
              </template>
            </el-table-column>
            <el-table-column label="Subject" min-width="300">
              <template #default="{ row }">
                <span class="text-[#1a1a1a]">{{ row.subject }}</span>
              </template>
            </el-table-column>
            <el-table-column label="Time" width="180">
              <template #default="{ row }">
                <span class="text-[#64748b]">{{ row.date }}</span>
              </template>
            </el-table-column>
            <el-table-column label="Actions" width="120" fixed="right">
              <template #default="{ row }">
                <div class="flex gap-4">
                  <a
                    href="javascript:;"
                    class="text-[#3b82f6] hover:text-[#2563eb]"
                    @click="viewEmail(row)"
                  >
                    View
                  </a>
                  <a
                    href="javascript:;"
                    class="text-[#ef4444] hover:text-[#dc2626]"
                    @click="deleteEmail(row)"
                  >
                    Delete
                  </a>
                </div>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>

      <!-- 价格方案区域 -->
      <div class="pricing-section py-20 text-center">
        <h2 class="text-4xl font-bold mb-4">Pricing</h2>
        <p class="text-gray-600 mb-12">
          Check out our pricing plan and engage in with which works best for you
        </p>

        <!-- 切换按钮 -->
        <div class="inline-flex bg-white rounded-full p-1 mb-12">
          <button
            v-for="cycle in ['month', 'year']"
            :key="cycle"
            class="px-8 py-2 rounded-full text-sm font-medium transition-all"
            :class="billingCycle === cycle ? 'bg-purple-500 text-white' : 'text-gray-500'"
            @click="billingCycle = cycle"
          >
            {{ cycle.charAt(0).toUpperCase() + cycle.slice(1) }}
          </button>
        </div>

        <!-- 价格卡片 -->
        <div class="grid md:grid-cols-2 gap-8 max-w-4xl mx-auto">
          <!-- 免费版卡片 -->
          <div class="pricing-card">
            <h3 class="text-2xl font-bold text-pink-500 mb-4">Free</h3>
            <div class="text-4xl font-bold mb-8">
              $0
              <span class="text-base font-normal text-gray-500">/mo</span>
            </div>
            <ul class="space-y-4">
              <li class="flex items-center">
                <el-icon class="text-green-500 mr-2"><Check /></el-icon>
                <span>100% Private address with full ownership</span>
              </li>
              <li class="flex items-center">
                <el-icon class="text-green-500 mr-2"><Check /></el-icon>
                <span>Enhanced privacy and security</span>
              </li>
              <li class="flex items-center">
                <el-icon class="text-green-500 mr-2"><Check /></el-icon>
                <span>No ads</span>
              </li>
            </ul>
          </div>
          <!-- 高级版卡片 -->
          <div class="pricing-card">
            <h3 class="text-2xl font-bold text-pink-500 mb-4">Premium</h3>
            <div class="text-4xl font-bold mb-8">
              ${{ billingCycle === 'month' ? '5' : '50' }}
              <span class="text-base font-normal text-gray-500"
                >/{{ billingCycle === 'month' ? 'mo' : 'yr' }}</span
              >
            </div>
            <ul class="space-y-4">
              <li class="flex items-center">
                <el-icon class="text-green-500 mr-2"><Check /></el-icon>
                <span>Unlimited address switching every day</span>
              </li>
              <li class="flex items-center">
                <el-icon class="text-green-500 mr-2"><Check /></el-icon>
                <span>Dedicated premium domains</span>
              </li>
              <li class="flex items-center">
                <el-icon class="text-green-500 mr-2"><Check /></el-icon>
                <span>100% Private address with full ownership</span>
              </li>
              <li class="flex items-center">
                <el-icon class="text-green-500 mr-2"><Check /></el-icon>
                <span>Enhanced privacy and security</span>
              </li>
              <li class="flex items-center">
                <el-icon class="text-green-500 mr-2"><Check /></el-icon>
                <span>Favorite address</span>
              </li>
            </ul>
          </div>
        </div>
      </div>

      <!-- FAQ区域 -->
      <div class="bg-[#faf7f5] rounded-[24px] max-w-[800px] mx-auto my-8 p-8">
        <h2 class="text-3xl font-bold mb-6 text-[#1a1a1a]">Frequently asked questions</h2>
        <div class="space-y-6">
          <div>
            <h3 class="text-base font-semibold mb-2 text-[#1a1a1a]">What is temporary mail?</h3>
            <p class="text-sm text-[#64748b] leading-relaxed"
              >Temporary email is a short-term use mailbox that helps protect privacy, avoid spam,
              and register quickly.</p
            >
          </div>
          <div>
            <h3 class="text-base font-semibold mb-2 text-[#1a1a1a]"
              >What scenarios are temporary mailboxes suitable for?</h3
            >
            <p class="text-sm text-[#64748b] leading-relaxed"
              >Temporary email is suitable for scenarios such as registering on websites, trying out
              services, protecting privacy, avoiding spam, and participating in short-term
              activities.</p
            >
          </div>
          <div>
            <h3 class="text-base font-semibold mb-2 text-[#1a1a1a]"
              >The email address I have used will be assigned to someone else?</h3
            >
            <p class="text-sm text-[#64748b] leading-relaxed"
              >We have built-in tens of billions of email addresses to ensure that used addresses
              will not be assigned to others.</p
            >
          </div>
          <div>
            <h3 class="text-base font-semibold mb-2 text-[#1a1a1a]"
              >Is the temporary mailbox free?</h3
            >
            <p class="text-sm text-[#64748b] leading-relaxed"
              >Yes, TempMail100 provides free temporary mailboxes forever.</p
            >
          </div>
          <div>
            <h3 class="text-base font-semibold mb-2 text-[#1a1a1a]"
              >I changed my mind, can I request a refund?</h3
            >
            <p class="text-sm text-[#64748b] leading-relaxed"
              >You have a 3-day cooling-off period, which means if you change your mind within 3
              days, you can get a full refund. We do not offer refunds if you change your mind after
              3 days.</p
            >
          </div>
        </div>
      </div>

      <!-- 社区评价区域 -->
      <div class="bg-[#f8fafc] py-20">
        <div class="max-w-[800px] mx-auto">
          <h2 class="text-[32px] font-bold text-center text-[#1a1a1a] mb-8">
            What the community is saying
          </h2>
          <div class="space-y-4">
            <div
              v-for="item in testimonials"
              :key="item.id"
              class="bg-white rounded-[16px] p-6 shadow-[0_4px_20px_rgba(0,0,0,0.05)]"
            >
              <div class="flex items-center gap-3 mb-3">
                <img :src="item.avatar" :alt="item.name" class="w-10 h-10 rounded-full" />
                <span class="font-medium text-[15px] text-[#1a1a1a]">{{ item.name }}</span>
              </div>
              <p class="text-[15px] leading-[1.6] text-[#1a1a1a]">{{ item.content }}</p>
            </div>
          </div>
        </div>
      </div>

      <!-- 页脚 -->
      <footer class="text-center pb-8">
        <div class="space-y-4">
          <div class="flex justify-center gap-8">
            <a href="#" class="text-[#475569] hover:text-[#1a1a1a]">Privacy Policy</a>
            <a href="#" class="text-[#475569] hover:text-[#1a1a1a]">Terms & Conditions</a>
          </div>
          <div class="flex justify-center gap-8">
            <a href="#" class="text-[#475569] hover:text-[#1a1a1a]">Blog</a>
            <a href="#" class="text-[#475569] hover:text-[#1a1a1a]">Contact support</a>
          </div>
          <div class="text-[#475569]">Copyright ©TempMail</div>
        </div>
      </footer>
    </main>

    <!-- 邮件内容对话框 -->
    <el-dialog
      v-model="showEmailDialog"
      :title="currentEmail?.subject"
      width="70%"
      destroy-on-close
      class="email-dialog"
    >
      <div class="email-header mb-4 p-4 bg-gray-50 rounded-lg">
        <div class="flex justify-between items-center mb-2">
          <span class="font-medium">From: {{ currentEmail?.from }}</span>
          <span class="text-gray-500">{{ currentEmail?.date }}</span>
        </div>
        <div class="font-medium">Subject: {{ currentEmail?.subject }}</div>
      </div>
      <div class="email-content" v-html="currentEmail?.content"></div>
      <template #footer>
        <el-button @click="showEmailDialog = false">Close</el-button>
        <el-button type="primary" @click="copyEmailContent">Copy Content</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
defineOptions({ name: 'ToolsMail' })

import { ref, onMounted, onBeforeUnmount } from 'vue'
import { ElMessage } from 'element-plus'
import { Refresh, CopyDocument, Check, ArrowDown } from '@element-plus/icons-vue'
import { useClipboard } from '@vueuse/core'
import { useI18n } from '@/hooks/web/useI18n'
import { useLocale } from '@/hooks/web/useLocale'
import { CACHE_KEY, useCache } from '@/hooks/web/useCache'
import { useRouter } from 'vue-router'
import { testimonials } from './data'
import { UserInfo } from './components/UserInfo'

const { t } = useI18n()
const { changeLocale } = useLocale()
const { wsCache } = useCache('localStorage')
const router = useRouter()

interface EmailItem {
  id: string
  from: string
  subject: string
  date: string
  content: string
}

const { copy } = useClipboard()
const loading = ref(false)
const emailAddress = ref('')
const emailList = ref<EmailItem[]>([])
const showEmailDialog = ref(false)
const currentEmail = ref<EmailItem | null>(null)
let refreshTimer: NodeJS.Timer | null = null
const billingCycle = ref<'month' | 'year'>('month')

// 生成新邮箱
const refreshEmail = async () => {
  loading.value = true
  try {
    // TODO: 调用生成临时邮箱的 API
    emailAddress.value = `temp${Date.now()}@tempmail100.com`
    ElMessage.success('邮箱地址已更新')
  } catch (error) {
    ElMessage.error('生成邮箱失败')
  } finally {
    loading.value = false
  }
}

// 复制邮箱地址
const copyEmail = async () => {
  if (!emailAddress.value) {
    ElMessage.warning('请先生成邮箱地址')
    return
  }
  await copy(emailAddress.value)
  ElMessage.success('邮箱地址已复制')
}

// 刷新收件箱
const refreshInbox = async () => {
  loading.value = true
  try {
    // TODO: 调用获取邮件列表的 API
    // 模拟数据
    emailList.value = [
      {
        id: '1',
        from: 'service@example.com',
        subject: '欢迎使用临时邮箱服务',
        date: new Date().toLocaleString(),
        content: '感谢您使用我们的临时邮箱服务...'
      }
    ]
  } catch (error) {
    ElMessage.error('刷新收件箱失败')
  } finally {
    loading.value = false
  }
}

// 查看邮件
const viewEmail = (email: EmailItem) => {
  currentEmail.value = email
  showEmailDialog.value = true
}

// 删除邮件
const deleteEmail = async (email: EmailItem) => {
  try {
    // TODO: 调用删除邮件的 API
    emailList.value = emailList.value.filter((item) => item.id !== email.id)
    ElMessage.success('邮件已删除')
  } catch (error) {
    ElMessage.error('删除邮件失败')
  }
}

// 复制邮件内容
const copyEmailContent = async () => {
  if (currentEmail.value?.content) {
    await copy(currentEmail.value.content)
    ElMessage.success('邮件内容已复制')
  }
}

// 语言切换
const handleSetLanguage = async (lang: string) => {
  await changeLocale(lang)
  ElMessage.success('切换成功')
}

// 登录
const login = async () => {
  const user = wsCache.get(CACHE_KEY.USER)
  if (!user) {
    router.push('/login')
  }
}

onMounted(() => {
  refreshEmail()
  // 自动刷新收件箱
  refreshTimer = setInterval(refreshInbox, 30000)
})

onBeforeUnmount(() => {
  if (refreshTimer) {
    clearInterval(refreshTimer)
  }
})
</script>

<style lang="scss" scoped>
.temp-mail-wrapper {
  position: relative;
  width: 100%;
  min-height: 100vh;
  background: linear-gradient(135deg, rgb(167 243 208) 0%, rgb(216 180 254) 100%);
}

// 调整头部样式
header {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  z-index: 10;
  padding: 1.5rem 4rem;
  background: transparent;

  > div {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
  }
}

main {
  position: relative;
  width: 100%;
  min-height: 100vh;
  padding-bottom: 4rem;
}

:global(body) {
  padding: 0;
  margin: 0;
  overflow: hidden auto;
}

:global(#app) {
  height: auto;
  min-height: 100vh;
  overflow: visible;
}

// 邮箱地址区域
.text-center {
  padding: 8rem 2rem 2rem;
  text-align: center;

  h1 {
    margin-bottom: 3rem;
    font-size: 2.5rem;
    font-weight: 600;
    background: linear-gradient(135deg, #ec4899 0%, #8b5cf6 100%);
    background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  p {
    margin-bottom: 2rem;
    font-size: 1rem;
    color: #666;
  }
}

// 邮箱输入框容器
.email-container {
  width: 100%;
  max-width: 600px;
  padding: 0 2rem;
  margin: 0 auto 3rem;

  .email-input-container {
    display: flex;
    height: 3.5rem;
    padding: 0 0.5rem;
    background: white;
    border-radius: 1.75rem;
    box-shadow: 0 4px 20px rgb(0 0 0 / 10%);
    align-items: center;

    button {
      display: flex;
      width: 2.75rem;
      height: 2.75rem;
      margin: 0 0.25rem;
      border-radius: 50%;
      align-items: center;
      justify-content: center;

      &:hover {
        background-color: rgb(0 0 0 / 5%);
      }

      .el-icon {
        font-size: 1.25rem;
        color: #666;
      }
    }

    input {
      height: 100%;
      padding: 0 1rem;
      font-size: 1rem;
      color: #333;
      background: transparent;
      border: none;
      flex: 1;

      &::placeholder {
        color: #999;
      }
    }
  }
}

// 收件箱区域
.email-container + .email-container {
  max-width: 800px;
  margin-bottom: 4rem;

  .bg-white {
    overflow: hidden;
    background: white;
    border-radius: 1rem;
    box-shadow: 0 4px 20px rgb(0 0 0 / 10%);

    .flex {
      padding: 1.25rem 1.5rem;
      border-bottom: 1px solid #eee;

      h2 {
        font-size: 1.25rem;
        font-weight: 600;
        color: #333;
      }

      button {
        padding: 0.5rem 1rem;
        font-size: 0.875rem;
        font-weight: 500;
        color: white;
        background: #000;
        border-radius: 1rem;
      }
    }

    .py-40 {
      padding: 6rem 0;
      font-size: 1rem;
      color: #999;
      text-align: center;
    }
  }
}

// 价格方案区域
.pricing-section {
  padding: 4rem 2rem;
  text-align: center;
  background: rgb(255 255 255 / 50%);

  h2 {
    margin-bottom: 1rem;
    font-size: 2rem;
    font-weight: 600;
    color: #333;
  }

  p {
    margin-bottom: 3rem;
    color: #666;
  }

  .grid {
    display: grid;
    max-width: 1000px;
    padding: 0 2rem;
    margin: 0 auto;
    grid-template-columns: repeat(2, 1fr);
    gap: 2rem;
  }

  .pricing-card {
    padding: 2rem;
    background: white;
    border-radius: 1rem;
    box-shadow: 0 4px 20px rgb(0 0 0 / 10%);

    h3 {
      margin-bottom: 1rem;
      color: #ec4899;
    }

    .text-4xl {
      margin-bottom: 2rem;
      font-size: 2.5rem;
      font-weight: 600;
    }

    ul {
      text-align: left;

      li {
        display: flex;
        align-items: center;
        margin-bottom: 1rem;
        color: #666;

        .el-icon {
          margin-right: 0.75rem;
          color: #22c55e;
        }
      }
    }
  }
}

// 对话框样式
.email-dialog {
  :deep(.el-dialog) {
    overflow: hidden;
    border-radius: 1rem;

    .el-dialog__header {
      padding: 1rem 1.5rem;
      border-bottom: 1px solid #eee;
    }

    .el-dialog__body {
      padding: 1.5rem;
    }

    .el-dialog__footer {
      padding: 1rem 1.5rem;
      border-top: 1px solid #eee;
    }
  }
}

// 表格样式优化
.custom-table {
  :deep(.el-table) {
    background: transparent;

    &::before {
      display: none;
    }

    th {
      padding: 1rem 1.5rem;
      font-size: 0.875rem;
      font-weight: 500;
      color: #666;
      background: transparent;
      border: none;
    }

    td {
      padding: 1rem 1.5rem;
      font-size: 0.875rem;
      color: #333;
      border: none;
    }

    tr {
      background: transparent;

      &:hover > td {
        background: rgb(0 0 0 / 2%);
      }
    }
  }
}
</style>
