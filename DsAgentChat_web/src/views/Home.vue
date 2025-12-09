<template>
  <div class="app-container" :class="userStore.theme">
    <!-- 侧边栏 -->
    <div class="sidebar" :class="{ 'sidebar-collapsed': isSidebarCollapsed }">
      <div class="sidebar-header">
        <div class="logo-wrapper">
          <span class="logo-text"><span class="logo-highlight">AI电商智能助手</span></span>
          <button class="collapse-btn" @click="toggleSidebar">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M6 2L2 8L6 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
          </button>
        </div>
      </div>
      
      <!-- 添加历史聊天列表 -->
      <div class="chat-history">
        <button class="new-chat-btn" @click="startNewChat">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
            <path d="M8 3.33334V12.6667" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            <path d="M12.6667 8L3.33333 8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          开启新对话
        </button>
        
        <div class="history-list">
          <div v-for="conversation in conversationStore.conversations" 
               :key="conversation.id"
               :class="['history-item', { active: conversation.id === conversationStore.currentConversationId }]">
            <div class="history-content" @click="handleConversationClick(conversation.id)">
              <span class="history-title">{{ conversation.title }}</span>
              <span class="history-time">{{ formatTime(new Date(conversation.created_at)) }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 底部个人信息 -->
      <div class="user-section">
        <!-- 添加电商客服按钮 -->
        <div class="ecommerce-link" @click="goToEcommerce">
          <div class="ecommerce-icon">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <rect width="24" height="24" rx="12" fill="#FF4D4F" opacity="0.1"/>
              <path d="M6 6H7.5L8 9M8 9L9.5 16H16L17.5 9H8Z" stroke="#FF4D4F" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
              <circle cx="10.5" cy="19" r="1" fill="#FF4D4F"/>
              <circle cx="15.5" cy="19" r="1" fill="#FF4D4F"/>
            </svg>
          </div>
          <span class="ecommerce-text">电商客服</span>
          <div class="ecommerce-badge">新</div>
        </div>
        
        <div class="user-menu" @click.stop="showUserMenu = !showUserMenu">
          <div class="user-avatar">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
              <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="1.5"/>
              <circle cx="12" cy="9" r="3" stroke="currentColor" stroke-width="1.5"/>
              <path d="M17.9691 20C17.81 17.1085 16.9247 15 12 15C7.07527 15 6.18997 17.1085 6.03087 20" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
          </div>
          <span class="user-text">{{ userStore.username }}</span>
        </div>
        
        <!-- 用户菜单 -->
        <div v-if="showUserMenu" class="user-dropdown">
          <div class="menu-item" @click="handleLogout">
            <svg width="16" height="16" viewBox="0 0 16 16">
              <path d="M6 14H3C2.44772 14 2 13.5523 2 13V3C2 2.44772 2.44772 2 3 2H6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
              <path d="M10 11L14 8L10 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
              <path d="M14 8H6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
            退出登录
          </div>
        </div>
      </div>
    </div>

    <!-- 添加展开按钮 -->
    <button v-if="isSidebarCollapsed" 
            class="expand-btn" 
            @click="toggleSidebar">
      <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M10 2L14 8L10 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
      </svg>
    </button>

    <!-- 主聊天区域 -->
    <div class="chat-container" id="chat-container">
      <div class="chat-content">
        <!-- 初始状态：欢迎消息和输入框在一起居中 -->
        <div v-if="!messages.length && !uploadedFile" class="initial-state">
          <div class="welcome-message">
            <h2>我是 AI电商智能助手, 很高兴见到你!</h2>
            <p>我可以帮你写代码、读文件、写作各种创意内容，请把你的任务交给我吧~</p>
          </div>
          <div class="chat-input">
            <div class="input-wrapper">
              <input
                v-model="userInput"
                @keyup.enter="sendMessage"
                type="text"
                placeholder="给 AI电商智能助手 发送消息"
              />
                <div class="button-group">
                  <div class="left-buttons">
                    <button 
                      class="tool-btn"
                      :class="{ 'tool-btn-active': isDeepThinking }"
                      @click="toggleDeepThinking"
                    >
                      <div class="icon">🔄</div>
                      深度思考
                    </button>
                    <button 
                      class="tool-btn"
                      :class="{ 'tool-btn-active': isSearching }"
                      @click="toggleSearch"
                    >
                      <div class="icon">🌐</div>
                      {{ isSearching ? '取消搜索' : '联网搜索' }}
                    </button>
                    <button 
                      class="tool-btn"
                      :class="{ 'tool-btn-active': isRagMode }"
                      @click="isRagMode = !isRagMode"
                    >
                      <div class="icon">📚</div>
                      知识库问答
                    </button>
                  </div>
                  <div class="right-buttons">
                  <button class="tool-btn" @click="$refs.fileInput?.click()">
                    <div class="icon">📎</div>
                  </button>
                  <button 
                    class="send-btn" 
                    :class="{ 'send-btn-active': userInput.trim() }" 
                    :disabled="!userInput.trim()"
                    @click="sendMessage"
                  >
                    <div class="icon">
                      <svg width="14" height="16" viewBox="0 0 14 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <path fill-rule="evenodd" clip-rule="evenodd" d="M7 16c-.595 0-1.077-.462-1.077-1.032V1.032C5.923.462 6.405 0 7 0s1.077.462 1.077 1.032v13.936C8.077 15.538 7.595 16 7 16z" fill="currentColor"/>
                        <path fill-rule="evenodd" clip-rule="evenodd" d="M.315 7.44a1.002 1.002 0 0 1 0-1.46L6.238.302a1.11 1.11 0 0 1 1.523 0c.421.403.421 1.057 0 1.46L1.838 7.44a1.11 1.11 0 0 1-1.523 0z" fill="currentColor"/>
                        <path fill-rule="evenodd" clip-rule="evenodd" d="M13.685 7.44a1.11 1.11 0 0 1-1.523 0L6.238 1.762a1.002 1.002 0 0 1 0-1.46 1.11 1.11 0 0 1 1.523 0l5.924 5.678c.42.403.42 1.056 0 1.46z" fill="currentColor"/>
                      </svg>
                    </div>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <!-- 对话状态或有文件上传时：消息列表在上，输入框在底部 -->
        <div v-else class="chat-state">
          <div class="chat-messages">
            <div v-for="(message, index) in messages" 
                 :key="index"
                 :class="['message', message.role === 'user' ? 'user-message' : 'assistant-message']">
              <div class="message-avatar">
                <!-- 用户头像 -->
                <svg v-if="message.role === 'user'" width="24" height="24" viewBox="0 0 24 24" fill="none">
                  <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="1.5"/>
                  <circle cx="12" cy="9" r="3" stroke="currentColor" stroke-width="1.5"/>
                  <path d="M17.9691 20C17.81 17.1085 16.9247 15 12 15C7.07527 15 6.18997 17.1085 6.03087 20" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
                </svg>
                <!-- AI头像 -->
                <svg v-else width="24" height="24" viewBox="0 0 24 24">
                  <path d="M12 2L20 7V17L12 22L4 17V7L12 2Z" fill="#4b4bff" opacity="0.2"/>
                  <path d="M12 2L20 7V17L12 22L4 17V7L12 2Z" stroke="#4b4bff" stroke-width="1.5"/>
                  <circle cx="12" cy="12" r="3" fill="#4b4bff" opacity="0.2" stroke="#4b4bff" stroke-width="1.5"/>
                </svg>
              </div>
              <div class="message-content" v-html="renderMessage(message)" @click="handleMessageClick(message, $event)"></div>
            </div>
          </div>
          <div class="chat-input-container">
            <!-- 文件上传状态显示 -->
            <div v-if="uploadedFile" class="cefa5c26 d5e44c7a">
              <div class="a4380d7b">
                <div class="cd190a50 e5931f90">
                  <div class="d2d04dae">
                    <div class="ds-icon b3a5d6c1" style="font-size: 32px; width: 32px; height: 32px;">
                      <svg width="32" height="32" viewBox="0 0 32 32" fill="none" xmlns="http://www.w3.org/2000/svg"
                           :data-type="getFileType(uploadedFile.type)">
                        <path d="M7 9C7 6.79086 8.79086 5 11 5L18.6383 5C19.1906 5 19.6383 5.44772 19.6383 6V6.92308C19.6383 9.13222 21.4292 10.9231 23.6383 10.9231H24C24.5523 10.9231 25 11.3708 25 11.9231V23C25 25.2091 23.2091 27 21 27H11C8.79086 27 7 25.2091 7 23V9Z"/>
                      </svg>
                    </div>
                    <div class="aea7ca45">
                      <div class="f3a54b52">{{ uploadedFile.name }}</div>
                      <div class="ee357eab">
                        {{ uploadedFile.status === 'uploading' ? '上传中...' :
                           uploadedFile.status === 'error' ? '上传失败' :
                           `${formatFileSize(uploadedFile.size)}` }}
                      </div>
                    </div>
                    <button class="delete-btn" @click="handleDeleteFile">
                      <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
                        <path d="M12 4L4 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
                        <path d="M4 4L12 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
                      </svg>
                    </button>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- 原有的聊天输入框 -->
            <div class="chat-input">
              <div class="input-wrapper">
                <input
                  v-model="userInput"
                  @keyup.enter="sendMessage"
                  type="text"
                  placeholder="给 AI电商智能助手 发送消息"
                />
                <div class="button-group">
                  <div class="left-buttons">
                    <button 
                      class="tool-btn"
                      :class="{ 'tool-btn-active': isDeepThinking }"
                      @click="toggleDeepThinking"
                    > 
                      <div class="icon">🔄</div>
                      深度思考
                    </button>
                    <button 
                      class="tool-btn"
                      :class="{ 'tool-btn-active': isSearching }"
                      @click="toggleSearch"
                    >
                      <div class="icon">🌐</div>
                      {{ isSearching ? '取消搜索' : '联网搜索' }}
                    </button>
                    <button 
                      class="tool-btn"
                      :class="{ 'tool-btn-active': isRagMode }"
                      @click="isRagMode = !isRagMode"
                    >
                      <div class="icon">📚</div>
                      知识库问答
                    </button>
                  </div>
                  <div class="right-buttons">
                    <button class="tool-btn" @click="$refs.fileInput?.click()">
                      <div class="icon">📎</div>
                    </button>
                    <button 
                      class="send-btn" 
                      :class="{ 'send-btn-active': userInput.trim() }" 
                      :disabled="!userInput.trim()"
                      @click="sendMessage"
                    >
                      <div class="icon">
                        <svg width="14" height="16" viewBox="0 0 14 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                          <path fill-rule="evenodd" clip-rule="evenodd" d="M7 16c-.595 0-1.077-.462-1.077-1.032V1.032C5.923.462 6.405 0 7 0s1.077.462 1.077 1.032v13.936C8.077 15.538 7.595 16 7 16z" fill="currentColor"/>
                          <path fill-rule="evenodd" clip-rule="evenodd" d="M.315 7.44a1.002 1.002 0 0 1 0-1.46L6.238.302a1.11 1.11 0 0 1 1.523 0c.421.403.421 1.057 0 1.46L1.838 7.44a1.11 1.11 0 0 1-1.523 0z" fill="currentColor"/>
                          <path fill-rule="evenodd" clip-rule="evenodd" d="M13.685 7.44a1.11 1.11 0 0 1-1.523 0L6.238 1.762a1.002 1.002 0 0 1 0-1.46 1.11 1.11 0 0 1 1.523 0l5.924 5.678c.42.403.42 1.056 0 1.46z" fill="currentColor"/>
                        </svg>
                      </div>
                    </button>
                  </div>
                </div>
              </div>
            </div>
            <div class="disclaimer-text">内容由 AI 生成，请仔细甄别</div>
          </div>
        </div>
      </div>
    </div>

    <!-- 添加搜索结果面板 -->
    <div v-if="showSearchPanel" class="search-panel">
      <div class="search-panel-header">
        <h3>搜索结果</h3>
        <button class="close-btn" @click="showSearchPanel = false">
          <span>×</span>
        </button>
      </div>
      <div class="search-results-list">
        <div v-for="(result, index) in searchResults" 
             :key="index"
             class="search-result-item"
             :class="{ active: result.isExpanded }">
          <div class="result-header" @click="toggleResultExpand(result)">
            <div class="result-source">
              <img :src="getSourceIcon(result.source)" class="source-icon" />
              <span class="source-name">{{ result.source }}</span>
              <span class="result-date">{{ result.date }}</span>
            </div>
            <div class="result-title">{{ result.title }}</div>
          </div>
          <div v-if="result.isExpanded" class="result-content">
            <div class="result-snippet">{{ result.snippet }}</div>
            <a :href="result.url" target="_blank" class="result-link" @click.stop>查看原文</a>
          </div>
        </div>
      </div>
    </div>

    <!-- 添加搜索结果详情面板 -->
    <div v-if="selectedResult" class="result-detail-panel">
      <div class="detail-header">
        <button class="back-btn" @click="selectedResult = null">
          <span>←</span>
        </button>
        <h3>{{ selectedResult.title }}</h3>
      </div>
      <div class="detail-content">
        <div class="detail-meta">
          <span class="detail-source">{{ selectedResult.source }}</span>
          <span class="detail-date">{{ selectedResult.date }}</span>
        </div>
        <div class="detail-text">{{ selectedResult.snippet }}</div>
        <a :href="selectedResult.url" target="_blank" class="detail-link">查看原文</a>
      </div>
    </div>

    <!-- 在 app-container div 内部的最后添加 -->
    <input 
      type="file" 
      ref="fileInput"
      @change="handleFileUpload"
      style="display: none"
      multiple
    />
    
    <!-- 重命名对话框 -->
    <div v-if="showRenameModal" class="modal-overlay">
      <div class="modal-content">
        <h3>重命名对话</h3>
        <input 
          v-model="newConversationName" 
          type="text" 
          class="rename-input" 
          placeholder="请输入新名称"
          @keyup.enter="confirmRename"
        />
        <div class="modal-actions">
          <button class="cancel-btn" @click="cancelRename">取消</button>
          <button class="confirm-btn" @click="confirmRename">确认</button>
        </div>
      </div>
    </div>
    
    <!-- 删除确认对话框 -->
    <div v-if="showDeleteConfirm" class="modal-overlay">
      <div class="modal-content">
        <h3>确认删除</h3>
        <p>确定要删除这个对话吗？此操作无法撤销。</p>
        <div class="modal-actions">
          <button class="cancel-btn" @click="cancelDelete">取消</button>
          <button class="delete-btn" @click="executeDelete">删除</button>
        </div>
      </div>
    </div>
    
    <!-- 添加悬浮聊天按钮 -->
    <div class="chat-float-button" @click="toggleChatPopup" v-if="!showChatPopup">
      <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M20 2H4C2.9 2 2 2.9 2 4V22L6 18H20C21.1 18 22 17.1 22 16V4C22 2.9 21.1 2 20 2Z" fill="#4b4bff"/>
        <path d="M7 9H17M7 13H14" stroke="white" stroke-width="1.5" stroke-linecap="round"/>
      </svg>
    </div>
    
    <!-- 聊天弹窗 -->
    <div class="chat-popup" v-if="showChatPopup">
      <div class="chat-popup-header">
        <h3>客服助手</h3>
        <button class="chat-popup-close" @click="toggleChatPopup">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
            <path d="M12 4L4 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            <path d="M4 4L12 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
        </button>
      </div>
      <div class="chat-popup-messages">
        <div v-for="(message, index) in popupMessages" 
             :key="index"
             :class="['popup-message', message.role === 'user' ? 'popup-user-message' : 'popup-assistant-message']">
          <div class="popup-message-content">
            <template v-if="message.isLoading">
              <div class="loading-dots">
                <span></span>
                <span></span>
                <span></span>
              </div>
            </template>
            <template v-else>
              <div v-html="md.render(message.content)"></div>
            </template>
          </div>
        </div>
      </div>
      <div class="chat-popup-input">
        <input 
          v-model="popupInput" 
          @keyup.enter="sendPopupMessage"
          type="text" 
          pplaceholder="请输入您的问题..."
        />
        <button class="popup-send-btn" @click="sendPopupMessage">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
            <path d="M15 1L7.5 8.5M15 1L10 15L7.5 8.5M15 1L1 6L7.5 8.5" stroke="currentColor" stroke-width="1.5"/>
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick, onMounted, onUnmounted, watch } from 'vue'
import { marked, Renderer } from 'marked'
import DOMPurify from 'dompurify'
import MarkdownIt from 'markdown-it'
import { ApiService, type Conversation } from '../services/api'
import { AuthService } from '../services/api'
import { useUserStore } from '../stores/user'
import { useConversationStore } from '../stores/conversation'
import type { ChatMessage } from '../types'
import { useRouter } from 'vue-router'

interface StreamResponse {
  content: string
  done: boolean
}

interface ExecutionResponse {
  type: 'conclusion' | 'process' | 'files' | 'cost' | 'error' | 'done'
  content: any
}

interface ExecutionRequest {
  prompt: string
  taskId?: string
}

interface ChatRequest {
  messages: {
    role: string
    content: string
  }[]
}

interface MessageUI {
  role: string
  content: string
  thinking?: string
  conclusion?: string
  process?: string
  files?: {
    files: string[]
    urls: string[]
  }
  cost?: {
    total: number
    details: Record<string, number>
  }
  hasDetails?: boolean
  showDetails?: boolean
  isSearching?: string
}

interface ChatHistory {
  id: number
  title: string
  time: Date
  messages: MessageUI[]
}

interface SearchResult {
  title: string
  url: string
  snippet: string
  date: string
  source: string
  isExpanded?: boolean
}

interface SearchResponse {
  type: 'search_results'
  total: number
  query: string
  results: SearchResult[]
  error?: string
}

// 定义消息类型
interface PopupMessage {
  role: string;
  content: string;
  isLoading?: boolean;
}

const userInput = ref('')
const messages = ref<MessageUI[]>([])
const isSidebarCollapsed = ref(false)
const chatHistory = ref<ChatHistory[]>([])
const currentChatIndex = ref(0)
const currentAgent = ref('openai')
const currentResponse = ref('')
const ws = ref<WebSocket | null>(null)
const isDeepThinking = ref(false)
const thinkStartTime = ref<number | null>(null)
const isSearching = ref(false)
const searchResults = ref<SearchResult[]>([])
const showSearchPanel = ref(false)
const selectedResult = ref<SearchResult | null>(null)
const fileInput = ref<HTMLInputElement | null>(null)
const uploadedFileInfo = ref<{
  name: string;
  type: string;
  size: number;
} | null>(null)
const uploadStatus = ref<{
  isUploading: boolean;
  fileName: string;
} | null>(null)
const uploadedFile = ref<{
  name: string;
  size: number;
  type: string;
  status: 'uploading' | 'success' | 'error';
} | null>(null)
const isRagMode = ref(false)
const currentIndexId = ref<string | null>(null)
const currentConversationId = ref<number | null>(null)

const md = new MarkdownIt({
  breaks: true,
  linkify: true,
  typographer: true
})

const scrollToBottom = async () => {
  await nextTick()
  const container = document.querySelector('.chat-container')
  if (container) {
    // 使用平滑滚动
    container.scrollTo({
      top: container.scrollHeight,
      behavior: 'smooth'
    })
  }
}

const userStore = useUserStore()
const conversationStore = useConversationStore()

// 对话重命名相关变量
const showRenameModal = ref(false)
const conversationToRename = ref<Conversation | null>(null)
const newConversationName = ref('')

// 对话删除相关变量
const showDeleteConfirm = ref(false)
const conversationIdToDelete = ref<number | null>(null)

// 添加聊天弹窗相关变量
const showChatPopup = ref(false)
const popupMessages = ref<PopupMessage[]>([
  { role: 'assistant', content: '您好！我是智能客服助手，有什么可以帮您的吗？' }
])
const popupInput = ref('')

const router = useRouter()

// 前往电商客服页面
const goToEcommerce = () => {
  router.push('/ecommerce')
}

onMounted(async () => {
  try {
    const userInfo = await AuthService.getUserInfo()
    userStore.setUserInfo(userInfo)
    // 加载用户的会话列表
    await conversationStore.loadUserConversations()
    // 创建新会话
    if (conversationStore.isNewConversation) {
      await conversationStore.createNewConversation()
    }
  } catch (error) {
    console.error('Failed to fetch user info:', error)
  }
})

// 修改开启新会话的方法
const startNewChat = async () => {
  try {
    await conversationStore.createNewConversation()
    // 创建新会话后刷新会话列表
    await conversationStore.loadUserConversations()
    messages.value = []
    currentChatIndex.value = 0
    scrollToBottom()
  } catch (error) {
    console.error('Failed to start new chat:', error)
  }
}

const loadChat = (index: number) => {
  currentChatIndex.value = index
  messages.value = [...chatHistory.value[index].messages]
  scrollToBottom()
}

const formatTime = (date: Date) => {
  return new Intl.DateTimeFormat('zh-CN', {
    month: 'numeric',
    day: 'numeric',
    hour: 'numeric',
    minute: 'numeric'
  }).format(date)
}

const renderMessage = (message: MessageUI) => {
  // 清理内容中的转义字符和多余的引号
  const cleanContent = (text: string) => {
    if (!text) return '';
    
    // 移除可能存在的data:前缀
    text = text.replace(/^data:\s*/g, '');
    
    // 移除多余的引号
        // 如果文本以引号开始和结束，移除它们
        text = text.replace(/^["']|["']$/g, '');
    
    // 移除每行开头的引号
    text = text.split('\n').map(line => line.replace(/^["']\s*/, '')).join('\n');
    
    // 移除每行结尾的引号
        text = text.split('\n').map(line => line.replace(/\s*["']$/, '')).join('\n');
    
    // 移除文本中连续的两个引号
    text = text.replace(/""/g, '');
    
    // 替换转义的换行符
    text = text.replace(/\\n/g, '\n');
    
    // 替换转义的引号
        text = text.replace(/\\"/g, '"');
    
    return text;
  };

  // 如果是搜索状态的消息，直接返回原始 HTML
  if (message.isSearching) {
    return message.content;
  }

  // 如果包含思考过程（使用Markdown格式）
  if (message.content.includes('### 思考过程')) {
    // 分割内容为思考过程和回复
    const parts = message.content.split('---');
    
    // 清理并渲染内容
        const cleanParts = parts.map(part => cleanContent(part));
    
    // 渲染为Markdown
    if (parts.length > 1) {
      // 有思考过程和回复
      return `
        <div class="message-wrapper">
          <div class="thinking-process">
            <div class="thinking-content">
              ${md.render(cleanParts[0])}
            </div>
          </div>
          <div class="message-text">
            ${md.render(cleanParts[1])}
          </div>
        </div>
      `;
    } else {
      // 只有思考过程
            return `
        <div class="message-wrapper">
          <div class="thinking-process">
            <div class="thinking-content">
              ${md.render(cleanParts[0])}
            </div>
          </div>
        </div>
      `;
    }
  }

  // 如果包含搜索加载状态
  if (message.content.includes('search-loading-container')) {
    const modelResponseDiv = message.content.indexOf('<div class="model-response">');
    if (modelResponseDiv !== -1) {
      const searchStatus = message.content.slice(0, modelResponseDiv);
      const response = message.content.slice(modelResponseDiv);
      return searchStatus + md.render(cleanContent(response));
    }
  }
  
  // 处理普通消息内容
    return md.render(cleanContent(message.content));
};

// 修改处理流式响应的函数，在每次内容更新时都滚动
const handleStreamResponse = async (reader: ReadableStreamDefaultReader<Uint8Array>, 
                                  onData: (content: string) => void) => {
  const decoder = new TextDecoder()
  
  while (true) {
    const { done, value } = await reader.read()
    if (done) break
    
    const text = decoder.decode(value)
    const lines = text.split('\n')
    
    for (const line of lines) {
      if (!line.startsWith('data: ')) continue
      
      const content = line.slice(6)
      if (content === '[DONE]') continue

      onData(content)
      // 每次更新内容后滚动
            scrollToBottom()
    }
  }
}

// 处理搜索请求
const handleSearch = async (reader: ReadableStreamDefaultReader<Uint8Array>) => {
  if (!reader) throw new Error('No reader available')

  let currentContent = ''
  const decoder = new TextDecoder()

  try {
    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      const chunk = decoder.decode(value)
      const lines = chunk.split('\n')

      for (const line of lines) {
        if (!line.startsWith('data: ')) continue
        
        const content = line.slice(6).trim()
        if (!content || content === '[DONE]') continue

        try {
          const data = JSON.parse(content)
          
          switch (data.type) {
            case 'search_start':
              // 显示"正在联网检索"
              messages.value.push({
                role: 'assistant' as const,
                content: `<div class="search-loading-container">
                  <div class="search-loading-text">🔍 正在联网检索...</div>
                </div>
                <div class="model-response"></div>`,
                isSearching: 'true'
              })
              break

            case 'search_results':
              // 处理搜索结果
              searchResults.value = data.results.map((result: any) => ({
                ...result,
                date: new Date().toLocaleDateString('zh-CN'),
                source: new URL(result.url).hostname,
                isExpanded: false
              }))
              
              // 更新搜索状态消息
                            const lastMessage = messages.value[messages.value.length - 1]
              if (lastMessage && lastMessage.isSearching === 'true') {
                const modelResponseDiv = lastMessage.content.indexOf('<div class="model-response">')
                if (modelResponseDiv !== -1) {
                  const beforeResponse = lastMessage.content.slice(0, modelResponseDiv)
                  lastMessage.content = beforeResponse.replace(
                    '正在联网检索...',
                    '联网检索完成，点击查看结果'
                  ) + lastMessage.content.slice(modelResponseDiv)
                }
              }
              break

            case 'direct_answer':
              // 隐藏搜索提示，显示直接回复
              messages.value.push({
                role: 'assistant' as const,
                content: ''
              })
              break

            case 'direct_content':
              // 更新最后一条消息的内容
              const message = messages.value[messages.value.length - 1]
              if (message && message.role === 'assistant') {
                message.content += data.content
              }
              break

            default:
              // 处理普通文本内容
                            const cleanContent = content.replace(/^"|"$/g, '').replace(/\\n/g, '\n')
              currentContent += cleanContent

              // 更新消息内容
              const currentMessage = messages.value[messages.value.length - 1]
              if (currentMessage && currentMessage.isSearching === 'true') {
                const modelResponseDiv = currentMessage.content.indexOf('<div class="model-response">')
                if (modelResponseDiv !== -1) {
                  const beforeResponse = currentMessage.content.slice(0, modelResponseDiv + '<div class="model-response">'.length)
                  currentMessage.content = beforeResponse + md.render(currentContent) + '</div>'
                }
              }
          }

          scrollToBottom()
        } catch (e) {
          console.error('Error parsing message:', e)
        }
      }
    }
  } catch (error) {
    console.error('Error:', error)
    messages.value[messages.value.length - 1].content = '抱歉，搜索时发生了错误，请稍后重试。'
  }
}

// 修改 sendMessage 中的搜索部分
const sendMessage = async () => {
  if (!userInput.value.trim()) return

  try {
    const userMessage = {
      role: 'user' as const,
      content: userInput.value
    }
    messages.value.push(userMessage)
    userInput.value = ''

    if (isSearching.value) {
      const reader = await ApiService.search([userMessage], conversationStore.currentConversationId!)
  if (!reader) throw new Error('No reader available')
      await handleSearch(reader)
    } else if (isDeepThinking.value) {
      const reader = await ApiService.reason([userMessage], conversationStore.currentConversationId!)
      if (!reader) throw new Error('No reader available')
      await handleChatStream(reader)
    } else {
      const reader = await ApiService.chat([userMessage], conversationStore.currentConversationId!)
      if (!reader) throw new Error('No reader available')
      await handleChatStream(reader)
    }

    await scrollToBottom()
    // 发送消息后刷新会话列表
    await conversationStore.loadUserConversations()
  } catch (error) {
    console.error('Error:', error)
    messages.value.push({
      role: 'assistant' as const,
      content: '抱歉，发生了错误，请稍后重试。'
    })
  }
}

const selectAgent = (agent: string) => {
  if (agent === 'TwoAgentChat') {
    messages.value = []
    messages.value.push({
      role: 'assistant',
      content: '已切换到 TwoAgentChat 模式。请描述您的任务，我会协助您完成。'
    })
  }
  currentAgent.value = agent
}

const toggleSidebar = () => {
  isSidebarCollapsed.value = !isSidebarCollapsed.value;
}

const toggleDeepThinking = () => {
  isDeepThinking.value = !isDeepThinking.value;
  if (isDeepThinking.value) {
    // 如果开启深度思考，自动关闭搜索
    isSearching.value = false;
  }
}

const toggleSearch = () => {
  isSearching.value = !isSearching.value
  if (!isSearching.value) {
    // 如果取消搜索，重置状态
        isDeepThinking.value = false
  }
}

// 监听消息变化
watch(messages, () => {
  scrollToBottom()
}, { deep: true })

// 添加 ResizeObserver 来监听内容高度变化
onMounted(() => {
  const container = document.querySelector('.chat-container')
  if (container) {
    const resizeObserver = new ResizeObserver(() => {
      scrollToBottom()
    })
    
    resizeObserver.observe(container)
    
    // 组件卸载时清理
    onUnmounted(() => {
      resizeObserver.disconnect()
    })
  }
})

// 添加获取图标的方法
const getSourceIcon = (source: string) => {
  // 这里可以根据不同来源返回不同的图标URL
  return `https://www.google.com/s2/favicons?domain=${source}`
}

// 修改处理搜索结果的逻辑
const handleSearchResults = (results: SearchResult[]) => {
  searchResults.value = results.map(result => ({
    ...result,
    date: new Date().toLocaleDateString('zh-CN'),
    source: new URL(result.url).hostname,
    isExpanded: false
  }));
  
  // 更新搜索状态消息
  const lastMessage = messages.value[messages.value.length - 1];
  if (lastMessage && lastMessage.isSearching === 'true') {
    const modelResponseDiv = lastMessage.content.indexOf('<div class="model-response">');
    if (modelResponseDiv !== -1) {
      const beforeResponse = lastMessage.content.slice(0, modelResponseDiv);
      lastMessage.content = beforeResponse.replace(
        '正在联网检索...',
        '联网检索完成，点击查看结果'
      ) + lastMessage.content.slice(modelResponseDiv);
    }
    lastMessage.isSearching = 'false';
  }
  
  // 不自动显示搜索面板
    showSearchPanel.value = false;
};

// 添加切换展开状态的方法
const toggleResultExpand = (result: SearchResult) => {
  result.isExpanded = !result.isExpanded
}

// 修改消息点击事件处理
const handleMessageClick = (message: MessageUI, event: MouseEvent) => {
  // 检查点击的元素是否包含搜索完成的文本
    const clickedElement = event.target as HTMLElement;
  const searchContainer = clickedElement.closest('.search-loading-container');
  
  if (searchContainer && searchResults.value.length > 0) {
    showSearchPanel.value = true;
  }
};

const handleFileUpload = async (event: Event) => {
  const input = event.target as HTMLInputElement
  if (!input.files?.length) return
  
  const userId = localStorage.getItem('user_id')
  if (!userId) {
    console.error('No user ID found')
    return
  }

  // 处理多个文件
  for (const file of input.files) {
    const formData = new FormData()
    formData.append('file', file)
    formData.append('user_id', userId)
    
    // 设置上传中状态
    uploadedFile.value = {
      name: file.name,
      size: file.size,
      type: file.type,
      status: 'uploading'
    }
    
    try {
      const response = await fetch('http://localhost:8000/api/upload', {
        method: 'POST',
        body: formData
      })
      
      if (!response.ok) {
        throw new Error('Upload failed')
      }
      
      const result = await response.json()
      
      if (result.status === 'success') {
        // 保存 index_id
        currentIndexId.value = result.index_id
        
        // 更新上传成功状态
        uploadedFile.value = {
          name: result.original_name,
          size: result.size,
          type: result.type,
          status: 'success'
        }
      }
      
    } catch (err: any) {
      console.error('Upload failed:', err)
      if (uploadedFile.value) {
        uploadedFile.value.status = 'error'
      }
      currentIndexId.value = null
    }
  }
  
  input.value = ''
}

// 格式化文件大小的辅助函数
const formatFileSize = (bytes: number) => {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB'
}

// 添加删除文件的方法
const handleDeleteFile = () => {
  uploadedFile.value = null
  currentIndexId.value = null
}

// 添加获取文件类型的方法
const getFileType = (mimeType: string) => {
  if (mimeType.includes('pdf')) return 'pdf'
  if (mimeType.includes('word')) return 'doc'
  if (mimeType.includes('excel') || mimeType.includes('spreadsheet')) return 'xls'
  if (mimeType.includes('powerpoint') || mimeType.includes('presentation')) return 'ppt'
  if (mimeType.includes('text')) return 'txt'
  if (mimeType.includes('image')) return 'image'
  return 'default'
}

const handleLogout = () => {
  AuthService.logout()
}

const showUserMenu = ref(false)

// 关闭菜单的点击外部处理
const handleClickOutside = (event: MouseEvent) => {
  const target = event.target as HTMLElement
  if (!target.closest('.user-menu')) {
    showUserMenu.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
})

const handleSubmit = async () => {
  if (!userInput.value.trim()) return

  try {
    // 如果没有当前会话ID，先创建新会话
    if (!currentConversationId.value) {
      currentConversationId.value = await ApiService.createConversation()
    }

    // 添加用户消息
    const userMessage = {
      role: 'user' as const,
      content: userInput.value
    }
    messages.value.push(userMessage)
    userInput.value = ''

    // 如果开启了搜索模式
    if (isSearching.value) {
      const reader = await ApiService.search([userMessage], currentConversationId.value)
      if (!reader) throw new Error('No reader available')
      await handleSearch(reader)
    } else if (isDeepThinking.value) {
      const reader = await ApiService.reason([userMessage], currentConversationId.value)
      if (!reader) throw new Error('No reader available')
      await handleChatStream(reader)
    } else {
      const reader = await ApiService.chat([userMessage], currentConversationId.value)
      if (!reader) throw new Error('No reader available')
      await handleChatStream(reader)
    }

    // 发送消息后刷新会话列表
    await conversationStore.loadUserConversations()
  } catch (error) {
    console.error('Error:', error)
    messages.value.push({
      role: 'assistant' as const,
      content: '抱歉，发生了错误，请稍后重试。'
    })
  }
}

// 处理新会话按钮点击
const handleNewConversation = async () => {
  try {
    // 创建新会话
    currentConversationId.value = await ApiService.createConversation()
    // 清空消息
    messages.value = []
    userInput.value = ''
  } catch (error) {
    console.error('Error creating new conversation:', error)
  }
}

const handleChatStream = async (reader: ReadableStreamDefaultReader<Uint8Array>) => {
  try {
    // 添加助手消息
    messages.value.push({
      role: 'assistant',
      content: '',
      thinking: '' // 添加thinking字段用于存储思考过程
          })
    
    let hasResponse = false;
    
    // 使用ApiService处理流
    await ApiService.handleChatStream(reader, (chunk) => {
      // 清理内容的函数
      const cleanChunkContent = (content: string) => {
        if (!content) return '';
        
        // 移除可能的data:前缀
        content = content.replace(/^data:\s*/g, '').trim();
        
        // 移除多余的引号
                // 如果文本以引号开始和结束，移除它们
                content = content.replace(/^["']|["']$/g, '');
        
        // 移除每行开头的引号
        content = content.split('\n').map(line => line.replace(/^["']\s*/, '')).join('\n');
        
        // 移除每行结尾的引号
                content = content.split('\n').map(line => line.replace(/\s*["']$/, '')).join('\n');
        
        // 移除文本中连续的两个引号
        content = content.replace(/""/g, '');
        
        // 替换转义的换行符
        content = content.replace(/\\n/g, '\n');
        
        // 替换转义的引号
                content = content.replace(/\\"/g, '"');
        
        return content;
      };
      
      const lastMessage = messages.value[messages.value.length - 1]
      
      if (chunk.type === 'think') {
        // 存储思考过程，但不修改content字段
        lastMessage.thinking = cleanChunkContent(chunk.content);
        
        // 只有在深度思考模式下才显示思考过程
        if (isDeepThinking.value) {
          // 如果已经有普通响应，则将思考过程放在前面
          if (hasResponse) {
            lastMessage.content = `### 思考过程
${lastMessage.thinking}

---

${lastMessage.content}`
          } else {
            // 否则只显示思考过程
            lastMessage.content = `### 思考过程\n\n${lastMessage.thinking}`
          }
        }
      } else if (chunk.type === 'response') {
        // 清理响应内容
        const cleanResponse = cleanChunkContent(chunk.content);
        hasResponse = true;
        
        // 如果是普通响应，检查是否在深度思考模式下且有思考过程
        if (isDeepThinking.value && lastMessage.thinking) {
          // 深度思考模式：在回复前添加思考过程
          lastMessage.content = `### 思考过程
${lastMessage.thinking}

---

${cleanResponse}`
        } else {
          // 普通模式：只显示回复内容
          lastMessage.content = cleanResponse;
        }
      }
    })
    
    await scrollToBottom()
  } catch (error) {
    console.error('Error handling chat stream:', error)
    const lastMessage = messages.value[messages.value.length - 1]
    lastMessage.content = '抱歉，发生了错误，请稍后重试。'
  }
}

// 处理会话点击
const handleConversationClick = async (conversationId: number) => {
  try {
    await conversationStore.loadConversationMessages(conversationId)
    // 转换消息格式以匹配现有的渲染逻辑
    messages.value = conversationStore.currentMessages.map(msg => ({
      role: msg.sender === 'user' ? 'user' : 'assistant',
      content: msg.content
    } as MessageUI))
  } catch (error) {
    console.error('Failed to load conversation:', error)
  }
}

// 显示重命名对话框
const showRenameDialog = (conversation: Conversation) => {
  conversationToRename.value = conversation
  newConversationName.value = conversation.title
  showRenameModal.value = true
}

// 确认重命名对话
const confirmRename = async () => {
  if (!conversationToRename.value || !newConversationName.value.trim()) return
  
  try {
    await conversationStore.updateConversationName(
      conversationToRename.value.id,
      newConversationName.value.trim()
    )
    showRenameModal.value = false
    conversationToRename.value = null
    newConversationName.value = ''
  } catch (error) {
    console.error('Failed to rename conversation:', error)
  }
}

// 取消重命名对话
const cancelRename = () => {
  showRenameModal.value = false
  conversationToRename.value = null
  newConversationName.value = ''
}

// 显示删除确认对话框
const confirmDelete = (conversationId: number) => {
  conversationIdToDelete.value = conversationId
  showDeleteConfirm.value = true
}

// 确认删除对话
const executeDelete = async () => {
  if (!conversationIdToDelete.value) return
  
  try {
    await conversationStore.deleteConversation(conversationIdToDelete.value)
    showDeleteConfirm.value = false
    conversationIdToDelete.value = null
    
    // 如果删除的是当前会话，创建新会话
    if (conversationStore.isNewConversation) {
      await conversationStore.createNewConversation()
      messages.value = []
    }
  } catch (error) {
    console.error('Failed to delete conversation:', error)
  }
}

// 取消删除对话
const cancelDelete = () => {
  showDeleteConfirm.value = false
  conversationIdToDelete.value = null
}

// 切换聊天弹窗显示
const toggleChatPopup = () => {
  showChatPopup.value = !showChatPopup.value
}

// 发送弹窗消息
const sendPopupMessage = async () => {
  if (!popupInput.value.trim()) return
  
  // 添加用户消息
  popupMessages.value.push({
    role: 'user',
    content: popupInput.value
  })
  
  const userQuestion = popupInput.value
  popupInput.value = ''
  
  try {
    // 显示加载状态
    popupMessages.value.push({
      role: 'assistant',
      content: '正在思考...',
      isLoading: true
    })
    
    // 获取用户ID
    const userId = localStorage.getItem('user_id')
    if (!userId) {
      throw new Error('用户ID不存在')
    }
    
    // 调用LangGraph接口
    const response = await fetch('http://localhost:8000/api/langgraph/query', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        query: userQuestion,
        user_id: parseInt(userId),
        conversation_id: conversationStore.currentConversationId ? 
          conversationStore.currentConversationId.toString() : 
          undefined
      })
    })
    
    if (!response.ok) {
      throw new Error('请求失败: ' + response.statusText)
    }
    
    // 移除加载消息
    popupMessages.value.pop()
    
    // 检查响应类型
    const contentType = response.headers.get('Content-Type')
    
    // 如果是流式响应
    if (contentType && contentType.includes('text/event-stream')) {
      // 添加一个空的助手消息
      popupMessages.value.push({
        role: 'assistant',
        content: ''
      })
      
      // 获取reader
      const reader = response.body?.getReader()
      if (!reader) throw new Error('无法读取响应流')
      
      // 处理流
      await handlePopupChatStream(reader)
    } else {
      // 非流式响应，处理JSON
      const result = await response.json()
      popupMessages.value.push({
        role: 'assistant',
        content: result.response || '抱歉，我无法处理您的请求。'
      })
    }
  } catch (error) {
    console.error('发送消息失败:', error)
    
    // 移除加载消息
    const loadingMsgIndex = popupMessages.value.findIndex(msg => msg.isLoading)
    if (loadingMsgIndex !== -1) {
      popupMessages.value.splice(loadingMsgIndex, 1)
    }
    
    // 显示错误消息
    popupMessages.value.push({
      role: 'assistant',
      content: '抱歉，发生了错误，请稍后再试。'
    })
  }
}

// 处理弹窗的聊天流
const handlePopupChatStream = async (reader: ReadableStreamDefaultReader<Uint8Array>) => {
  const decoder = new TextDecoder()
  let currentContent = ''
  
  try {
    while (true) {
      const { done, value } = await reader.read()
      if (done) break
  
      const chunk = decoder.decode(value)
      const lines = chunk.split('\n')
      
      for (const line of lines) {
        if (!line.startsWith('data: ')) continue
        
        const content = line.slice(6) // 移除 'data: ' 前缀
        if (content === '[DONE]') continue
        
        // 处理所有可能的换行符形式
        if (content === '"\\n\\n"' || content === '"\n\n"' || content === '\n\n') {
          currentContent += '\n\n'
          continue
        }
        
        // 移除引号
        let cleanedContent = content
        if (cleanedContent.startsWith('"') && cleanedContent.endsWith('"')) {
          cleanedContent = cleanedContent.slice(1, -1)
        }
        
        // 处理转义的换行符
        cleanedContent = cleanedContent.replace(/\\n\\n/g, '\n\n').replace(/\\n/g, '\n')
        // 添加到当前内容
        currentContent += cleanedContent
        
        // 更新最后一条消息的内容
        const lastMessage = popupMessages.value[popupMessages.value.length - 1]
        lastMessage.content = currentContent
      }
    }
  } catch (error) {
    console.error('Error handling chat stream:', error)
    const lastMessage = popupMessages.value[popupMessages.value.length - 1]
    lastMessage.content = '抱歉，发生了错误，请稍后重试。'
  }
}

// 格式化弹窗消息 - 使用pre-wrap样式确保换行显示
const formatPopupMessage = (content: string) => {
  if (!content) return ''
  return content
}
</script>

<style>
/* 添加全局样式 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>

<style scoped>
/* 根容�?*/
.app-container {
  display: flex;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-color: #1e1e1e;
}

.app-container.light {
  background-color: #ffffff;
  color: #333333;
}

.app-container.light .sidebar {
  background-color: #f5f5f5;
  border-right-color: #e0e0e0;
}

.app-container.light .chat-input {
  background-color: #f5f5f5;
}

.app-container.light input {
  color: #333333;
}

/* 左侧�?*/
.sidebar {
  width: 260px;
  height: 100vh;
  background-color: #2d2d2d;
  border-right: 1px solid #333;
  display: flex;
  flex-direction: column;
  transition: width 0.3s ease;
}

.sidebar-collapsed {
  width: 0;
  overflow: hidden;
  padding: 0;
}

/* 右侧聊天区域 */
.chat-container {
  flex: 1;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #1e1e1e;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: #4a4a4a #1e1e1e;
}

/* 聊天内容区域 */
.chat-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  transition: all 0.3s ease;
  overflow-y: visible;
  position: relative;
}

.chat-container::-webkit-scrollbar {
  width: 8px;
  position: absolute;
  right: 0;
}

.chat-container::-webkit-scrollbar-track {
  background: #1e1e1e;
}

.chat-container::-webkit-scrollbar-thumb {
  background-color: #4a4a4a;
  border-radius: 4px;
  border: 2px solid #1e1e1e;
}

.chat-container::-webkit-scrollbar-thumb:hover {
  background-color: #666;
}

/* 初始状态样�?*/
.initial-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 40px;
  margin: auto;
  width: 100%;
  max-width: 600px;
  opacity: 1;
  transform: translateY(0);
  transition: all 0.3s ease;
}

/* 对话状态样�?*/
.chat-state {
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100%;
  position: relative;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 消息区域样式 */
.chat-messages {
  flex: 1;
  padding-bottom: 280px; /* 增加更多底部padding，确保内容不被遮�?*/
}

.chat-input-container {
  position: fixed;
  bottom: 0;
  left: calc(50% + 130px); /* 考虑侧边栏宽度的一�?(260px / 2 = 130px) */
  transform: translateX(-50%);
  width: 100%;
  max-width: 800px;
  background-color: #1e1e1e;
  padding: 0 20px 20px; /* 修改padding */
  border-top: 1px solid #333;
  z-index: 10;
  transition: left 0.3s ease; /* 添加过渡效果 */
}

/* 当侧边栏收起时的样式 */
.sidebar-collapsed ~ .chat-container .chat-input-container {
  left: 50%; /* 侧边栏收起时回到中间 */
}

.chat-input {
  width: 100%;
  margin-bottom: 8px;
  transition: all 0.3s ease;
}

.chat-input .input-wrapper {
  max-width: 800px;
}

.disclaimer-text {
  text-align: center;
  color: #666;
  font-size: 12px;
  padding: 4px 0;
}

/* 聊天框样�?*/
.center-chat-box {
  display: flex;
  flex-direction: column;
  gap: 40px;
  width: 100%;
  max-width: 600px;
}

/* 欢迎消息样式 */
.welcome-message {
  text-align: center;
}

.welcome-message h2 {
  color: #fff;
  font-size: 24px;
  margin: 0;
  font-weight: normal;
}

.welcome-message p {
  color: #666;
  font-size: 13px;
  margin: 0;
  margin-top: 2px;
}

/* 消息样式 */
.message {
  padding: 12px 20px;
  display: flex;
  align-items: flex-start;
  gap: 12px;
  width: 100%;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 用户消息样式 */
.user-message {
  background-color: #1e1e1e;
  flex-direction: row-reverse;
}

/* AI消息样式 */
.assistant-message {
  background: none;
}

/* 头像样式 */
.message-avatar {
  width: 24px;
  height: 24px;
  flex-shrink: 0;
}

.user-message .message-avatar {
  color: #666;
}

/* 消息内容样式 */
.message-content {
  font-size: 14px;
  line-height: 1.6;
  color: #fff;
}

/* 底部个人信息样式 */
.user-section {
  padding: 16px;
  border-top: 1px solid #333;
  margin-top: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.user-menu {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 12px;
  cursor: pointer;
  border-radius: 6px;
  transition: all 0.2s;
  color: #fff;
}

.user-menu:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.arrow-icon {
  transition: transform 0.2s;
}

.user-menu:hover .arrow-icon {
  transform: rotate(180deg);
}

.user-dropdown {
  position: absolute;
  bottom: 100%;
  left: 0;
  right: 0;
  background: #2d2d2d;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 -4px 12px rgba(0, 0, 0, 0.2);
  margin: 0 8px 8px;
  z-index: 1000;
}

.menu-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  color: #fff;
  cursor: pointer;
  transition: all 0.2s;
}

.menu-item:hover {
  background: rgba(255, 255, 255, 0.1);
}

.menu-item:last-child {
  color: #ff4b4b;
}

.menu-item:last-child:hover {
  background: rgba(255, 75, 75, 0.1);
}

.user-avatar {
  width: 24px;
  height: 24px;
  color: #888;
}

.user-text {
  font-size: 14px;
  color: #fff;
}

/* 侧边栏样�?*/
.sidebar-header {
  padding: 0;
  border-bottom: 1px solid #333;
}

.logo-wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px;
  gap: 8px;
}

.logo-text {
  color: #fff;
  font-size: 2px;
  font-weight: 500;
  letter-spacing: 0.5px;
  background: linear-gradient(90deg, #fff 0%, #e0e0e0 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  flex: 1;
  min-width: 0;
}

.logo-highlight {
  background: linear-gradient(90deg, #4b4bff 0%, #6b6bff 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 600;
}

.collapse-btn {
  background: none;
  border: none;
  color: #666;
  padding: 8px;
  cursor: pointer;
  transition: all 0.2s;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.collapse-btn:hover {
  color: #fff;
  background: rgba(255, 255, 255, 0.1);
}

.agents-list {
  padding: 1rem 0;
  overflow-y: auto;
}

.agent-item {
  display: flex;
  align-items: center;
  padding: 0.8rem 1rem;
  cursor: pointer;
  transition: background-color 0.2s;
  gap: 0.8rem;
}

.agent-item:hover {
  background-color: #363636;
}

.agent-item.active {
  background-color: #4a4eff33;
}

.agent-icon {
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.2rem;
}

.agent-name {
  font-size: 0.9rem;
  color: #fff;
}

.new-chat-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  margin: 0 16px;
  padding: 8px 12px;
  font-size: 13px;
  color: #fff;
  background-color: #4b4bff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.new-chat-btn:hover {
  background-color: #3a3aff;
}

.new-chat-btn svg {
  width: 14px;
  height: 14px;
  color: #fff;
  opacity: 1;
}

/* 输入框容器样�?*/
.input-wrapper {
  width: 100%;
  background-color: #2d2d2d;
  border-radius: 25px;
  padding: 12px 16px;
}

input {
  width: 100%;
  background: none;
  border: none;
  color: #fff;
  font-size: 14px;
  outline: none;
  padding: 8px 0;
  margin-bottom: 4px;
}

input::placeholder {
  color: #666;
}

/* 按钮组样�?*/
.button-group {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 8px;
}

.left-buttons, .right-buttons {
  display: flex;
  gap: 8px;
  align-items: center;
}

.tool-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  background: none;
  border: none;
  color: #888;
  padding: 4px 8px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
  border-radius: 4px;
}

.tool-btn:hover {
  color: #fff;
  background: rgba(75, 75, 255, 0.1);
}

.tool-btn-active {
  color: #4b4bff !important;
  background: rgba(75, 75, 255, 0.1);
}

.tool-btn-active:hover {
  background: rgba(75, 75, 255, 0.2);
}

.send-btn {
  background: none;
  border: none;
  padding: 8px;
  cursor: pointer;
  color: #666;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.send-btn:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

.send-btn-active {
  color: #4b4bff;
}

.send-btn-active:hover {
  color: #3a3aff;
}

.send-btn .icon {
  width: 16px;
  height: 16px;
}

/* 空状态样式调�?*/
.empty-state {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  padding: 1rem;
}

.empty-state-content {
  background: none;
  padding: 0;
  box-shadow: none;
  width: 100%;
  max-width: 600px;
}

.empty-state-content h3 {
  color: #fff;
  font-size: 1rem;
  margin-bottom: 0.3rem;
}

.empty-state-content p {
  color: #666;
  font-size: 0.9rem;
  margin-bottom: 1.5rem;
}

.empty-state-content .input-wrapper {
  margin-top: 0;
}

/* 工具提示样式 */
.tool-btn .tooltip {
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background-color: #4a4a4a;
  color: #fff;
  padding: 0.3rem 0.6rem;
  border-radius: 4px;
  font-size: 0.8rem;
  white-space: nowrap;
  opacity: 0;
  visibility: hidden;
  transition: all 0.2s;
  margin-bottom: 0.5rem;
  pointer-events: none;
}

.tool-btn:hover .tooltip {
  opacity: 1;
  visibility: visible;
}

.tool-btn .tooltip::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  border-width: 4px;
  border-style: solid;
  border-color: #4a4a4a transparent transparent transparent;
}

/* 修改空状态下的输入框样式 */
.empty-state-content .input-wrapper {
  margin-top: 1.5rem;
  width: 100%;
}

/* 修改历史聊天列表样式 */
.chat-history {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.history-list {
  margin-top: 16px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.history-item {
  padding: 12px 16px;
  cursor: pointer;
  transition: all 0.2s ease;
  background-color: transparent;
  border-radius: 0;
}

.history-item:hover {
  background-color: #2d2d2d;
}

.history-item.active {
  background-color: #343541;
  border-left: none;
}

.history-content {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.history-title {
  color: #ececf1;
  font-size: 13px;
  line-height: 1.4;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.history-time {
  color: #666;
  font-size: 12px;
}

/* Markdown 样式优化 */
:deep(.message-content) {
  color: #fff;
  font-size: 14px;
  line-height: 1.6;

  h1, h2, h3, h4, h5, h6 {
    color: #fff;
    margin: 1em 0;
  }

  p {
    color: #fff;
    margin: 0.5em 0;
  }

  a {
    text-decoration: none;
    transition: all 0.2s ease;
  }

  a:hover {
    text-decoration: underline;
    opacity: 0.8;
  }

  ul, ol {
    color: #fff;
    margin: 0.5em 0;
    padding-left: 2em;
  }

  li {
    margin: 0.25em 0;
  }

  blockquote {
    margin: 1em 0;
    padding: 1em;
    background: rgba(51, 51, 51, 0.6);
    border-left: 4px solid #4b4bff;
    border-radius: 4px;
    color: #666;
  }

  blockquote strong {
    color: #4b4bff;
  }
}

/* 修改展开按钮样式 */
.expand-btn {
  position: fixed;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  color: #666;
  padding: 8px;
  cursor: pointer;
  transition: all 0.2s;
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
}

.expand-btn:hover {
  color: #fff;
}

/* 搜索结果样式 */
.search-result {
  background: rgba(75, 75, 255, 0.05);
  border: 1px solid rgba(75, 75, 255, 0.1);
  border-radius: 8px;
  padding: 16px;
  margin: 12px 0;
  transition: all 0.2s ease;
}

.search-result:hover {
  background: rgba(75, 75, 255, 0.1);
  border-color: rgba(75, 75, 255, 0.2);
}

.search-title {
  color: #4b4bff;
  font-size: 16px;
  font-weight: 500;
  text-decoration: none;
  display: block;
  margin-bottom: 8px;
  line-height: 1.4;
}

.search-title:hover {
  text-decoration: underline;
}

.search-snippet {
  color: #ccc;
  font-size: 14px;
  line-height: 1.6;
  margin-bottom: 8px;
}

.search-url {
  color: #666;
  font-size: 12px;
  word-break: break-all;
}

:deep(.message-content) {
  .search-result {
    background: rgba(75, 75, 255, 0.05);
    border: 1px solid rgba(75, 75, 255, 0.1);
    border-radius: 8px;
    padding: 16px;
    margin: 12px 0;
    transition: all 0.2s ease;
  }

  .search-result:hover {
    background: rgba(75, 75, 255, 0.1);
    border-color: rgba(75, 75, 255, 0.2);
  }

  .search-title {
    color: #4b4bff;
    font-size: 16px;
    font-weight: 500;
    text-decoration: none;
    display: block;
    margin-bottom: 8px;
    line-height: 1.4;
  }

  .search-title:hover {
    text-decoration: underline;
  }

  .search-snippet {
    color: #ccc;
    font-size: 14px;
    line-height: 1.6;
    margin-bottom: 8px;
  }

  .search-url {
    color: #666;
    font-size: 12px;
    word-break: break-all;
  }
}

/* 搜索面板样式 */
.search-panel {
  width: 320px;
  height: 100vh;
  background: #2d2d2d;
  border-right: 1px solid #333;
  display: flex;
  flex-direction: column;
}

.search-panel-header {
  padding: 16px;
  border-bottom: 1px solid #333;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.search-panel-header h3 {
  color: #fff;
  font-size: 16px;
  margin: 0;
}

.close-btn {
  background: none;
  border: none;
  color: #666;
  font-size: 20px;
  cursor: pointer;
  padding: 4px 8px;
}

.close-btn:hover {
  color: #fff;
}

.search-results-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
}

.search-result-item {
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  margin-bottom: 8px;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.search-result-item:hover {
  background: rgba(255, 255, 255, 0.05);
}

.search-result-item.active {
  background: rgba(75, 75, 255, 0.05);
  border-color: rgba(75, 75, 255, 0.2);
}

.result-header {
  cursor: pointer;
}

.result-source {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.source-name {
  color: #fff;
  font-size: 13px;
}

.result-date {
  color: #666;
  font-size: 12px;
  margin-left: auto;
}

.result-title {
  color: #fff;
  font-size: 14px;
  line-height: 1.4;
  margin-bottom: 4px;
}

.result-content {
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  animation: slideDown 0.2s ease;
}

.result-snippet {
  color: #ccc;
  font-size: 13px;
  line-height: 1.6;
  margin-bottom: 8px;
}

.result-link {
  display: inline-block;
  color: #4b4bff;
  text-decoration: none;
  font-size: 13px;
  transition: all 0.2s ease;
}

.result-link:hover {
  text-decoration: underline;
}

/* 搜索结果详情面板 */
.result-detail-panel {
  flex: 1;
  height: 100vh;
  background: #1e1e1e;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.detail-header {
  padding: 16px;
  border-bottom: 1px solid #333;
  display: flex;
  align-items: center;
  gap: 16px;
}

.back-btn {
  background: none;
  border: none;
  color: #666;
  font-size: 20px;
  cursor: pointer;
  padding: 4px 8px;
}

.back-btn:hover {
  color: #fff;
}

.detail-header h3 {
  color: #fff;
  font-size: 18px;
  margin: 0;
  line-height: 1.4;
}

.detail-content {
  padding: 24px;
}

.detail-meta {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 16px;
}

.detail-source {
  color: #fff;
  font-size: 14px;
}

.detail-date {
  color: #666;
  font-size: 14px;
}

.detail-text {
  color: #fff;
  font-size: 14px;
  line-height: 1.8;
  margin-bottom: 24px;
}

.detail-link {
  display: inline-block;
  color: #4b4bff;
  text-decoration: none;
  font-size: 14px;
  padding: 8px 16px;
  border: 1px solid #4b4bff;
  border-radius: 4px;
  transition: all 0.2s ease;
}

.detail-link:hover {
  background: rgba(75, 75, 255, 0.1);
}

/* 更新搜索加载状态样�?*/
:deep(.search-loading-container) {
  background: rgba(75, 75, 255, 0.1);
  border: 1px solid rgba(75, 75, 255, 0.3);
  border-radius: 20px;
  padding: 0px 0px;
  margin: 12px 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  position: sticky;
  top: 0;
  z-index: 10;
  cursor: pointer;
  transition: all 0.2s ease;
  width: auto;
}

:deep(.search-loading-container:hover) {
  background: rgba(75, 75, 255, 0.2);
  border-color: rgba(75, 75, 255, 0.4);
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

:deep(.search-loading-text) {
  color: #4b4bff;
  font-size: 13px;
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 500;
  padding: 4px 8px;
}

.thinking-process {
  background: rgba(75, 75, 255, 0.05);
  border: 1px solid rgba(75, 75, 255, 0.1);
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 16px;
}

.thinking-header {
  color: #4b4bff;
  font-size: 14px;
  font-weight: 500;
  margin-bottom: 8px;
}

.thinking-content {
  color: #888;
  font-size: 14px;
  line-height: 1.6;
}

.response-content {
  color: #fff;
  font-size: 14px;
  line-height: 1.6;
}

/* 添加渐变遮罩效果 */
.chat-input-container::before {
  content: '';
  position: absolute;
  top: -40px;
  left: 0;
  right: 0;
  height: 40px;
  background: linear-gradient(to bottom, transparent, #1e1e1e);
  pointer-events: none;
  z-index: -1;
}

/* 在已有样式中添加 */
.file-info-container {
  margin-top: 12px;
  padding: 12px;
  background: rgba(75, 75, 255, 0.05);
  border: 1px solid rgba(75, 75, 255, 0.1);
  border-radius: 8px;
}

.file-info-header {
  color: #4b4bff;
  font-size: 14px;
  font-weight: 500;
  margin-bottom: 8px;
}

.file-info-content {
  color: #fff;
  font-size: 14px;
  line-height: 1.6;
}

.file-info-content p {
  margin: 4px 0;
}

.search-loading-container.error {
  background: rgba(255, 75, 75, 0.1);
  border-color: rgba(255, 75, 75, 0.3);
}

.search-loading-container.error .search-loading-text {
  color: #ff4b4b;
}

/* 文件消息样式 */
:deep(.file-message) {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: rgba(75, 75, 255, 0.05);
  border: 1px solid rgba(75, 75, 255, 0.1);
  border-radius: 8px;
  margin: 8px 0;
}

:deep(.file-message.error) {
  background: rgba(255, 75, 75, 0.05);
  border-color: rgba(255, 75, 75, 0.1);
}

:deep(.file-icon) {
  font-size: 24px;
}

:deep(.file-info) {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

:deep(.file-name) {
  color: #fff;
  font-size: 14px;
  font-weight: 500;
}

:deep(.file-status) {
  color: #4b4bff;
  font-size: 12px;
}

:deep(.file-message.error .file-status) {
  color: #ff4b4b;
}

.upload-status-container {
  margin-bottom: 16px;
}

.upload-status {
  background: #2d2d2d;
  border-radius: 8px;
  overflow: hidden;
}

.upload-header {
  padding: 12px 16px;
  color: #fff;
  font-size: 14px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.upload-content {
  padding: 12px 16px;
}

.upload-file {
  display: flex;
  align-items: center;
  gap: 12px;
}

.file-icon {
  width: 32px;
  height: 32px;
}

.file-info {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.file-name {
  color: #fff;
  font-size: 14px;
  font-weight: 500;
}

.file-status {
  color: #888;
  font-size: 12px;
}

/* 文件上传消息样式 */
:deep(.cefa5c26) {
  background: #2d2d2d;
  border-radius: 6px;
  overflow: hidden;
  margin-bottom: 8px;
  box-shadow: 0 -4px 12px rgba(0, 0, 0, 0.1);
  width: 220px;
  z-index: 11;
  position: relative;  /* 将相对定位移到父容器 */
}

:deep(.ca114c67) {
  display: none;
}

:deep(.cd190a50) {
  display: flex;
  align-items: center;
  gap: 12px;
}

:deep(.d2d04dae) {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 10px;
}

:deep(.aea7ca45) {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

:deep(.f3a54b52) {
  color: #fff;
  font-size: 13px;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 120px;  /* 减小文件名显示宽�?*/
}

:deep(.ee357eab) {
  color: #888;
  font-size: 11px;
}

:deep(.error-message) {
  color: #ff4b4b;
  padding: 12px;
  background: rgba(255, 75, 75, 0.1);
  border: 1px solid rgba(255, 75, 75, 0.3);
  border-radius: 8px;
  margin: 8px 0;
}

:deep(.header-content) {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

:deep(.delete-btn) {
  background: none;
  border: none;
  color: #666;
  padding: 3px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: all 0.2s;
  position: absolute;  /* 保持绝对定位 */
  top: 4px;
  right: 4px;
  z-index: 12;  /* 确保按钮始终在最上层 */
}

:deep(.delete-btn:hover) {
  color: #ff4b4b;
  background: rgba(255, 75, 75, 0.1);
}

:deep(.ds-icon) {
  svg {
    &[data-type="pdf"] path {
      fill: #F8CA27;
    }
    &[data-type="doc"] path, &[data-type="docx"] path {
      fill: #4B8BF4;
    }
    &[data-type="xls"] path, &[data-type="xlsx"] path {
      fill: #34A853;
    }
    &[data-type="ppt"] path, &[data-type="pptx"] path {
      fill: #EA4335;
    }
    &[data-type="txt"] path {
      fill: #9AA0A6;
    }
    &[data-type="image"] path {
      fill: #4B8BF4;
    }
  }
}

:deep(.ds-icon.b3a5d6c1) {
  font-size: 24px;  /* 减小图标大小 */
  width: 24px;
  height: 24px;
}

.logout-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: none;
  border: none;
  color: #666;
  cursor: pointer;
  transition: all 0.2s;
}

.logout-btn:hover {
  color: #ff4b4b;
}

.history-title {
  font-size: 14px;
  color: #fff;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 180px;  /* 限制宽度，防止标题太�?*/
}

.history-time {
  font-size: 12px;
  color: #888;
}

.history-item {
  padding: 12px;
  cursor: pointer;
  transition: background-color 0.2s;
  border-radius: 6px;
  margin: 4px 0;
}

.history-item:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.history-item.active {
  background-color: rgba(75, 75, 255, 0.2);
}

.history-actions {
  display: flex;
  gap: 8px;
}

.action-btn {
  background: none;
  border: none;
  color: #666;
  cursor: pointer;
  transition: all 0.2s;
  border-radius: 4px;
}

.action-btn:hover {
  background: rgba(255, 255, 255, 0.1);
}

.rename-btn {
  display: flex;
  align-items: center;
  gap: 8px;
}

.delete-btn {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* 对话框样�?*/
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background-color: #2d2d2d;
  border-radius: 8px;
  padding: 20px;
  width: 100%;
  max-width: 400px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.app-container.light .modal-content {
  background-color: #ffffff;
  color: #333333;
}

.modal-content h3 {
  margin-bottom: 16px;
  font-size: 18px;
  font-weight: 500;
}

.modal-content p {
  margin-bottom: 20px;
  color: #aaa;
}

.app-container.light .modal-content p {
  color: #666;
}

.rename-input {
  width: 100%;
  padding: 10px;
  border: 1px solid #555;
  background-color: #333;
  color: #fff;
  border-radius: 4px;
  font-size: 14px;
  margin-bottom: 20px;
}

.app-container.light .rename-input {
  background-color: #f5f5f5;
  color: #333;
  border-color: #ddd;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

button.cancel-btn {
  background-color: transparent;
  border: 1px solid #555;
  color: #aaa;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

button.cancel-btn:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

button.confirm-btn {
  background-color: #0070f3;
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

button.confirm-btn:hover {
  background-color: #0051b3;
}

button.delete-btn {
  background-color: #e53e3e;
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

button.delete-btn:hover {
  background-color: #c53030;
}

.app-container.light button.cancel-btn {
  border-color: #ddd;
  color: #666;
}

.app-container.light button.cancel-btn:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

/* 悬浮聊天按钮样式 */
.chat-float-button {
  position: fixed;
  right: 30px;
  bottom: 30px;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background-color: #4b4bff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 1000;
  transition: all 0.3s ease;
}

.chat-float-button:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

/* 聊天弹窗样式 */
.chat-popup {
  position: fixed;
  right: 30px;
  bottom: 30px;
  width: 320px;
  height: 400px;
  background-color: #2d2d2d;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  display: flex;
  flex-direction: column;
  z-index: 1000;
  overflow: hidden;
  animation: popupFadeIn 0.3s ease;
}

.app-container.light .chat-popup {
  background-color: #ffffff;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

@keyframes popupFadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.chat-popup-header {
  padding: 12px 16px;
  border-bottom: 1px solid #333;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.app-container.light .chat-popup-header {
  border-bottom-color: #e0e0e0;
}

.chat-popup-header h3 {
  color: #fff;
  font-size: 16px;
  margin: 0;
}

.app-container.light .chat-popup-header h3 {
  color: #333;
}

.chat-popup-close {
  background: none;
  border: none;
  color: #666;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.chat-popup-close:hover {
  color: #fff;
  background: rgba(255, 255, 255, 0.1);
}

.app-container.light .chat-popup-close:hover {
  color: #333;
  background: rgba(0, 0, 0, 0.05);
}

.chat-popup-messages {
  flex: 1;
  padding: 16px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.popup-message {
  max-width: 80%;
  padding: 8px 12px;
  border-radius: 12px;
  font-size: 14px;
  line-height: 1.4;
}

.popup-user-message {
  align-self: flex-end;
  background-color: #4b4bff;
  color: white;
  border-bottom-right-radius: 4px;
}

.popup-assistant-message {
  align-self: flex-start;
  background-color: #3a3a3a;
  color: white;
  border-bottom-left-radius: 4px;
}

.app-container.light .popup-assistant-message {
  background-color: #f0f0f0;
  color: #333;
}

.chat-popup-input {
  padding: 12px;
  border-top: 1px solid #333;
  display: flex;
  gap: 8px;
}

.app-container.light .chat-popup-input {
  border-top-color: #e0e0e0;
}

.chat-popup-input input {
  flex: 1;
  background-color: #3a3a3a;
  border: none;
  color: white;
  padding: 8px 12px;
  border-radius: 20px;
  font-size: 14px;
  outline: none;
}

.app-container.light .chat-popup-input input {
  background-color: #f0f0f0;
  color: #333;
}

.chat-popup-input input::placeholder {
  color: #999;
}

.popup-send-btn {
  background-color: #4b4bff;
  border: none;
  color: white;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
}

.popup-send-btn:hover {
  background-color: #3a3aff;
}

.popup-send-btn:disabled {
  background-color: #555;
  cursor: not-allowed;
}

/* �?popup-message 样式下添加加载状态样�?*/
.popup-message.popup-assistant-message:has(.loading-dots) {
  background-color: #4b4bff20;
}

.loading-dots {
  display: inline-flex;
  align-items: center;
}

.loading-dots span {
  display: inline-block;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: currentColor;
  margin: 0 2px;
  animation: loadingDots 1.4s infinite ease-in-out both;
}

.loading-dots span:nth-child(1) {
  animation-delay: -0.32s;
}

.loading-dots span:nth-child(2) {
  animation-delay: -0.16s;
}

@keyframes loadingDots {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1); }
}

/* 电商客服按钮样式 */
.ecommerce-btn {
  background-color: #ff4d4f !important;
  color: white !important;
  transition: all 0.3s ease;
}

.ecommerce-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.ecommerce-link {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  background: linear-gradient(135deg, #ff4d4f 0%, #ff7875 60%, #ff9c9d 100%);
  border-radius: 12px;
  cursor: pointer;
  margin-bottom: 16px;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 4px 12px rgba(255, 77, 79, 0.3);
  position: relative;
  overflow: hidden;
}

.ecommerce-link:before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0) 70%);
  z-index: 1;
}

.ecommerce-link:hover {
  transform: translateY(-4px) scale(1.02);
  box-shadow: 0 8px 16px rgba(255, 77, 79, 0.4);
}

.ecommerce-link:active {
  transform: translateY(0) scale(0.98);
  box-shadow: 0 2px 8px rgba(255, 77, 79, 0.3);
}

.ecommerce-icon {
  width: 28px;
  height: 28px;
  margin-right: 12px;
  position: relative;
  z-index: 2;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.1));
}

.ecommerce-text {
  color: white;
  font-size: 15px;
  font-weight: 600;
  letter-spacing: 0.5px;
  position: relative;
  z-index: 2;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.ecommerce-badge {
  position: absolute;
  top: -6px;
  right: -6px;
  background-color: #ffeb3b;
  color: #d32f2f;
  font-size: 12px;
  font-weight: bold;
  padding: 2px 6px;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  z-index: 3;
  transform: rotate(15deg);
}

/* 思考过程样�?*/
:deep(.message-wrapper) {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
}

:deep(.thinking-process) {
  background: rgba(75, 75, 255, 0.05);
  border-radius: 8px;
  padding: 2px;
  margin-bottom: 16px;
  position: relative;
}

:deep(.thinking-content) {
  font-size: 14px;
  line-height: 1.6;
  overflow: auto;
  padding: 0;
}

:deep(.thinking-content blockquote) {
  margin: 0;
  padding: 16px;
  background: rgba(51, 51, 51, 0.4);
  border-left: 4px solid #4b4bff;
  border-radius: 0 8px 8px 0;
  color: #e0e0e0;
}

:deep(.thinking-content strong) {
  color: #4b4bff;
  font-weight: 600;
  display: block;
  margin-bottom: 8px;
}

:deep(.thinking-content p) {
  margin: 8px 0;
  color: #e0e0e0;
}

:deep(.thinking-content code) {
  background: rgba(75, 75, 255, 0.1);
  padding: 2px 4px;
  border-radius: 4px;
  color: #e0e0e0;
  font-family: monospace;
}

:deep(.thinking-content pre) {
  background: rgba(51, 51, 51, 0.6);
  padding: 12px;
  border-radius: 4px;
  overflow-x: auto;
  margin: 10px 0;
}

:deep(.thinking-content pre code) {
  background: transparent;
  padding: 0;
}

:deep(.message-text) {
  font-size: 14px;
  line-height: 1.6;
}
</style>
