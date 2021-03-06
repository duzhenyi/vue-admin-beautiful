<template>
  <div class="nav-bar-container">
    <el-row :gutter="15">
      <el-col :xs="4" :sm="12" :md="12" :lg="12" :xl="12">
        <div class="left-panel">
          <i
            :class="collapse ? 'el-icon-s-unfold' : 'el-icon-s-fold'"
            :title="collapse ? '展开' : '收起'"
            class="fold-unfold"
            @click="handleCollapse"
          ></i>
          <breadcrumb class="hidden-xs-only" />
        </div>
      </el-col>
      <el-col :xs="20" :sm="12" :md="12" :lg="12" :xl="12">
        <div class="right-panel">
          <error-log />
          <byui-screenfull @refresh="refreshSelectedTag"></byui-screenfull>
          <theme-bar></theme-bar>
          <byui-icon
            title="重载路由"
            :pulse="pulse"
            :icon="['fas', 'redo']"
            @click="refreshSelectedTag"
          />

          <el-avatar :src="require('@/assets/user.gif')"></el-avatar>
          <el-tooltip effect="light" placement="bottom">
            <div slot="content">上次登录时间:{{ lastLoginTime }}</div>
            <span class="user-name">{{ name }}</span>
          </el-tooltip>
          <byui-icon
            title="退出系统"
            :icon="['fas', 'sign-out-alt']"
            @click="logout"
          />
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import ErrorLog from "@/components/ErrorLog";
import ByuiScreenfull from "@/components/ByuiScreenfull";
import Breadcrumb from "@/layouts/components/Breadcrumb";
import ThemeBar from "@/layouts/components/ThemeBar";

export default {
  components: {
    Breadcrumb,
    ErrorLog,
    ByuiScreenfull,
    ThemeBar,
  },
  data() {
    return {
      pulse: false,
    };
  },
  computed: {
    ...mapGetters([
      "avatar",
      "collapse",
      "name",
      "loginTimes",
      "lastLoginTime",
      "selectedTag",
      "device",
    ]),
  },
  methods: {
    handleCollapse() {
      this.$store.dispatch("settings/changeCollapse");
      if ("mobile" == this.device && false === this.collapse) {
        $("body").attr("style", "overflow:hidden");
      }
    },
    async logout() {
      await this.$baseConfirm(
        "您确定要退出" + this.$baseTitle + "吗?",
        null,
        () => {
          const fullPath = this.$route.fullPath;
          this.$store.dispatch("user/logout");
          //this.$router.push(`/login?redirect=${fullPath}`);
        }
      );
    },
    refreshSelectedTag() {
      this.pulse = true;
      const view = this.selectedTag;
      this.$store.dispatch("tagsView/delCachedView", view).then(() => {
        const { fullPath } = view;
        this.$nextTick(() => {
          this.$router
            .replace({
              path: "/redirect" + fullPath,
            })
            .then(() => {
              setTimeout(() => {
                this.pulse = false;
              }, 1000);
            });
        });
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.nav-bar-container {
  user-select: none;
  height: 50px;
  overflow: hidden;
  position: relative;
  background: $base-color-white;
  box-shadow: $base-box-shadow;

  .left-panel {
    display: flex;
    justify-items: center;
    align-items: center;
    height: 50px;
    max-height: 50px;

    .fold-unfold {
      font-size: 20px;
      color: $base-color-gray;
      cursor: pointer;
      margin-left: 10px;
    }

    .fold-unfold.el-icon-s-unfold {
    }
    ::v-deep {
      .breadcrumb-container {
        margin-left: 10px;
      }
    }
  }

  .right-panel {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    height: 50px;

    .user-name {
      margin-right: 15px;
      margin-left: 5px;
    }

    ::v-deep {
      svg {
        width: 1em;
        height: 1em;
        color: $base-color-gray;
        fill: $base-color-gray;
        margin-right: 15px;
        cursor: pointer;
        font-size: $base-font-size-big;
      }

      button {
        svg {
          color: $base-color-white;
          fill: $base-color-white;
          margin-right: 0px;
          cursor: pointer;
        }
      }

      .el-badge {
        margin-right: 15px;
      }
    }
  }
}
</style>
