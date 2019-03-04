<template>
  <div id="app">
    <div class="app-wrapper">
      <el-container>
        <el-header>
          <!-- <app-header></app-header> -->
          <app-nav-menu></app-nav-menu>
        </el-header>
        <el-main>
          <div id="main-container" v-cloak>
            <router-view />
          </div>
        </el-main>
        <!-- <el-footer>
      <app-footer></app-footer>
    </el-footer> -->
      </el-container>
    </div>
  </div>
</template>

<script>
import installService from './api/install'
import appHeader from './components/common/header.vue'
import appNavMenu from './components/common/navMenu.vue'
import appFooter from './components/common/footer.vue'

export default {
  name: 'App',
  data () {
    return {}
  },
  created () {
    installService.status((data) => {
      if (!data) {
        this.$router.push('/install')
      }
    })
  },
  components: {
    appHeader,
    appNavMenu,
    appFooter
  }
}
</script>
