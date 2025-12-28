<template>
  <div class="sidebar-container" :class="{ 'is-collapsed': isCollapsed }">
    <div class="logo">
      <span class="logo-icon">📚</span>
      <span class="logo-text" v-show="!isCollapsed">员工交流论坛</span>
    </div>
    <div class="menu-wrapper">
      <el-menu 
        :default-active="activeMenu" 
        :collapse="isCollapsed" 
        :collapse-transition="false" 
        mode="vertical" 
        class="sidebar-menu"
        text-color="#bfcbd9" 
        active-text-color="#409EFF" 
        router
      >
        <template v-for="menu in menuItems">
          <!-- 没有子菜单 -->
          <el-menu-item 
            v-if="!menu.children?.length" 
            :key="menu.path"
            :index="getMenuPath(menu)"
          >
            <el-icon v-if="menu.icon">
              <component :is="menu.icon" />
            </el-icon>
            <template #title>{{ getMenuTitle(menu) }}</template>
          </el-menu-item>
          
          <!-- 有子菜单 -->
          <el-sub-menu v-else :key="menu.path" :index="getMenuPath(menu)">
            <template #title>
              <el-icon v-if="menu.icon">
                <component :is="menu.icon" />
              </el-icon>
              <span>{{ getMenuTitle(menu) }}</span>
            </template>
            
            <el-menu-item 
              v-for="child in menu.children" 
              :key="child.path" 
              :index="getChildMenuPath(menu, child)"
            >
              <el-icon v-if="child.icon">
                <component :is="child.icon" />
              </el-icon>
              <template #title>{{ getMenuTitle(child) }}</template>
            </el-menu-item>
          </el-sub-menu>
        </template>
      </el-menu>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useUserStore } from '@/store/user'
import { useAppStore } from '@/store/app'
import * as ElementPlusIcons from '@element-plus/icons-vue'

const route = useRoute()
const router = useRouter()
const userStore = useUserStore()
const appStore = useAppStore()

const isCollapsed = computed(() => appStore.sidebarCollapsed)

// 获取菜单数据
const menuItems = computed(() => {
  if (userStore.menus && userStore.menus.length > 0) {
    return userStore.menus
  }
  return []
})

// 获取菜单路径
const getMenuPath = (menu) => {
  if (!menu.path) return '/back/dashboard'
  
  // 如果是完整路径（以/开头），直接返回
  if (menu.path.startsWith('/')) {
    return menu.path
  }
  
  // 如果已经包含后台路径前缀，直接返回
  if (menu.path.startsWith('back/')) {
    return `/${menu.path}`
  }
  
  // 默认添加后台路径前缀
  return `/back/${menu.path}`
}

// 获取子菜单路径
const getChildMenuPath = (parentMenu, childMenu) => {
  const parentPath = getMenuPath(parentMenu)
  // 移除末尾的斜杠（如果有）
  const basePath = parentPath.endsWith('/') ? parentPath.slice(0, -1) : parentPath
  return `${basePath}/${childMenu.path}`
}

// 获取菜单项标题
const getMenuTitle = (menu) => {
  return menu.meta?.title || menu.name || '菜单项'
}

// 当前激活的菜单
const activeMenu = computed(() => {
  const { meta, path } = route
  if (meta.activeMenu) {
    return '/back/' + meta.activeMenu
  }
  return path
})
</script>

<style lang="scss" scoped>
.sidebar-container {
  height: 100%;
  background: linear-gradient(180deg, #1c1c1c 0%, #2d2d2d 100%);
  display: flex;
  flex-direction: column;
  width: 220px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  
  &.is-collapsed {
    width: 64px;
    
    .logo {
      padding: 0;
      justify-content: center;
      
      .logo-icon {
        margin: 0;
      }
    }

    :deep(.el-menu) {
      .el-sub-menu__title span,
      .el-menu-item span {
        opacity: 0;
        transition: opacity 0.2s;
      }
    }
  }
  
  .logo {
    height: 60px;
    flex-shrink: 0;
    line-height: 60px;
    text-align: center;
    background: rgba(255, 255, 255, 0.05);
    backdrop-filter: blur(10px);
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
    display: flex;
    align-items: center;
    padding: 0 16px;
    overflow: hidden;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    
    .logo-icon {
      font-size: 24px;
      margin-right: 8px;
      transition: margin 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    }
    
    .logo-text {
      color: #ffffff;
      font-size: 18px;
      font-weight: 600;
      white-space: nowrap;
      opacity: 1;
      transition: opacity 0.2s;
    }
  }

  .menu-wrapper {
    flex: 1;
    overflow-y: auto;
    overflow-x: hidden;

    &::-webkit-scrollbar {
      width: 6px;
    }

    &::-webkit-scrollbar-thumb {
      background: rgba(255, 255, 255, 0.2);
      border-radius: 3px;
    }

    &::-webkit-scrollbar-track {
      background: transparent;
    }
  }

  :deep(.sidebar-menu) {
    border: none;
    background: transparent;
    transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1);

    .el-menu-item, .el-sub-menu__title {
      height: 50px;
      line-height: 50px;
      color: rgba(255, 255, 255, 0.7);
      background: transparent;
      transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
      
      span {
        opacity: 1;
        transition: opacity 0.3s;
      }
      
      &:hover {
        background: rgba(255, 255, 255, 0.05) !important;
        color: #ffffff;
      }
    }

    .el-menu-item.is-active {
      background: #000000 !important;
      color: #ffffff !important;
      
      &::before {
        content: '';
        position: absolute;
        left: 0;
        top: 0;
        width: 3px;
        height: 100%;
        background: #409EFF;
      }
    }

    .el-sub-menu {
      &.is-opened {
        > .el-sub-menu__title {
          color: #ffffff;
          background: rgba(0, 0, 0, 0.2) !important;
        }
      }

      .el-menu {
        background: rgba(0, 0, 0, 0.3);
        
        .el-menu-item {
          background: transparent;
          
          &:hover {
            background: rgba(255, 255, 255, 0.05) !important;
          }
          
          &.is-active {
            background: #000000 !important;
          }
        }
      }
    }

    // 折叠状态下的弹出菜单样式
    &.el-menu--collapse {
      .el-sub-menu {
        &.is-opened {
          > .el-sub-menu__title {
            background: transparent !important;
          }
        }
      }
    }
  }

  .el-icon {
    vertical-align: middle;
    margin-right: 5px;
    width: 24px;
    text-align: center;
    color: inherit;
  }

  span {
    vertical-align: middle;
  }
}
</style> 