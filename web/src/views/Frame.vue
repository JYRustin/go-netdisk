<template>

  <div id="body">

    <div>
      <SideNavigation/>
      <div id="page-wrapper" ref="pageWrapper" :class="{'show-drawer':globalConfig.showDrawer}" @click="blankClick">
        <DragObscure v-show="dragEnterCount > 0"/>
        <div>

          <div class="breadcrumb-wrapper" v-if="breadcrumbs && breadcrumbs.length">

            <router-link to="/" class="breadcrumb-item">
              <i class="fa fa-home f16"></i>
            </router-link>

            <span v-for="b in breadcrumbs" class="breadcrumb-item">
              <span class="separator">/</span>
							<router-link v-if="(b.name || b.path) && b.name !== route.name" :to="b">
                {{b.displayDirect?b.title:$t(b.title)}}
              </router-link>
							<span v-else>
								{{b.displayDirect?b.title:$t(b.title)}}
							</span>
            </span>

          </div>
          <router-view></router-view>

        </div>


      </div>
      <TopNavigation v-if="breadcrumbs && breadcrumbs.length" />
      <!--手机上不显示bottomNavigation，而采用弹出的形式-->
      <BottomNavigation/>
    </div>
  </div>


</template>

<script>
  import Vue from 'vue'
  import SideNavigation from './layout/SideNavigation.vue'
  import TopNavigation from './layout/TopNavigation.vue'
  import BottomNavigation from './layout/BottomNavigation.vue'
  import DragObscure from './layout/DragObscure';
  import enquire from 'enquire.js/dist/enquire';
  import { mapState } from 'vuex';

  export default {
    data() {
      return {
        member: this.$store.state.member,
        dragEnterCount: 0,
      }
    },
    computed: {
      ...mapState({
        route: state => state.route,
        breadcrumbs: state => state.breadcrumbs,
        globalConfig: state => state.config,
      }),
      config() {
        return this.$store.state.config
      }
    },
    components: {
      SideNavigation,
      TopNavigation,
      BottomNavigation,
      DragObscure
    },
    methods: {
      blankClick() {
        if (this.config.mobile) {
          if (this.config.showDrawer) {
            this.$store.state.config.showDrawer = false
          }
        }
      },
      listenResponsiveEvent() {
        let that = this
        enquire.register('(max-width: 768px)', {
          match: function () {
            that.$store.state.config.showDrawer = false
            that.$store.state.config.mobile = true
          },
          unmatch: function () {
            that.$store.state.config.showDrawer = true
            that.$store.state.config.mobile = false
          }
        })
      },
      listenDrag() {
          this.$refs.pageWrapper.addEventListener('dragenter', e => {
              if(this.$route.name === 'MatterList') {
                  this.dragEnterCount++;
                  e.preventDefault();
              }
          })
          this.$refs.pageWrapper.addEventListener('dragleave', e => {
              if(this.$route.name === 'MatterList') {
                  this.dragEnterCount--;
                  e.preventDefault();
              }
          })
          this.$refs.pageWrapper.addEventListener('dragover', e => {
              if(this.$route.name === 'MatterList') {
                  e.preventDefault();
              }
          })
          this.$refs.pageWrapper.addEventListener('drop', e => {
              if(this.$route.name === 'MatterList') {
                  this.dragEnterCount = 0;
                  e.preventDefault();
                  new Vue().dropUploadFile(e.dataTransfer.files);
              }
          })
      }
    },
    mounted() {
      let that = this
      this.$store.state.environment = 'views'
      this.listenResponsiveEvent();
      this.listenDrag()
    }
  }
</script>

<style lang="less" rel="stylesheet/less">
  @import "../assets/css/global/variables";

  #page-wrapper {

    position: fixed;
    left: @sidebar-width;
    top: @top-navigation-height;
    right: 0;
    bottom: @power-footer-height;
    overflow-y: auto;
    overflow-x: hidden;
    z-index: 10;

    padding: 10px;

    -webkit-transition: all 0.4s;
    -moz-transition: all 0.4s;
    -o-transition: all 0.4s;
    transition: all 0.4s;

    background-color: #f3f3f4;

    //大屏幕
    @media (min-width: @screen-sm-min) {
      left: @sidebar-width;
    }
    //小屏幕
    @media (max-width: @screen-xs-max) {
      left: 0;
      bottom: 0;
    }

    &.show-drawer {
      //大屏幕
      @media (min-width: @screen-sm-min) {
        left: @sidebar-width;
      }

      //小屏幕
      @media (max-width: @screen-xs-max) {
        left: 0;
        bottom: 0;
      }
    }

    .breadcrumb-wrapper {
      background: #fafafa;
      padding: 15px;
      margin-bottom: 10px;

      .breadcrumb-item span.separator {
        margin: 0 5px;
        font-size: 14px;
      }
    }
  }

</style>
