<template>
  <div class="header" :style="{ '--bg': bgcolor,'--color':color }">
    <!-- 折叠按钮 -->
    <div class="head_left">
      <div class="logo">{{$t('i18n.moneySystem')}}</div>
    </div>
    <div class="head_btn">
      <div class="collapse-btn" @click="collapseChage">
        <i v-if="!collapse" class="el-icon-s-fold"></i>
        <i v-else class="el-icon-s-unfold"></i>
      </div>
        <!-- 全屏 -->
        <div class="screen">
          <i v-if="showScreen" class="fa fa-expand" @click="screen"></i>
          <i v-else class="fa fa-compress" @click="screen"></i>
        </div>
        <div class="taggle" :locale="$i18n.locale = locale">
          <el-switch v-model="value2" :active-text="$t('i18n.chinese')" :inactive-text="$t('i18n.english')" @change="language"></el-switch>
        </div>
<!--        <div class="taggle">
          <el-switch v-model="value" :active-text="$t('i18n.light')" :inactive-text="$t('i18n.dark')" @change="change"></el-switch>
        </div>-->
    </div>
    <div class="header-right">
      <div class="header-user-con">
        
      <div class="taggle" :title="newupdate">
          <span style="font-size:14px;">
            <span v-if="locale === 'zh-cn'" style="border-right:1px solid #fff;padding-right:5px">{{week[weekindex]}}</span>
            <span v-else style="border-right:1px solid #fff;padding-right:5px">{{weeks[weekindex]}}</span>
            {{ date }} &nbsp;{{time}}
          </span>
        </div>
        <!-- 消息中心 -->
        <!-- <div class="btn-bell">
          <el-tooltip effect="dark" :content="message?`有${message}条未读消息`:`消息中心`" placement="bottom">
            <router-link to="/tabs">
              <i class="el-icon-bell"></i>
            </router-link>
          </el-tooltip>
          <span class="btn-bell-badge" v-if="message"></span>
        </div>-->
        <!-- 用户头像 -->
        <div class="user-avator">
          <img src="@/assets/img/admin.jpg" />
        </div>
        <!-- 用户名下拉菜单 -->
        <el-dropdown class="user-name" trigger="click" @command="handleCommand">
          <span class="el-dropdown-link">
            {{username}}
            <i class="el-icon-caret-bottom"></i>
          </span>
          <template #dropdown>
            <el-dropdown-menu>
              <!-- <a href="https://github.com/lin-xin/vue-manage-system" target="_blank">
                                <el-dropdown-item>项目仓库</el-dropdown-item>
              </a>-->
              <!-- <el-dropdown-item command="userManage">{{$t('i18n.user')}}</el-dropdown-item> -->
              <el-dropdown-item @click="open">
                <span>{{$t('i18n.edituser')}}</span>
              </el-dropdown-item>
              <el-dropdown-item divided command="loginout">{{$t('i18n.letout')}}</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </div>
    </div>
    <el-dialog title="修改密码" v-model="dialogVisible" width="30%" center>
      <div>
        <el-form class="demo-form-inline" ref="loginForm" :model="formInline" :rules="rules" label-width="100px">
          <el-form-item label="原密码" prop="oldPassword">
            <el-input type="password" v-model="formInline.oldPassword" placeholder="请输入原密码" @blur="getpassword"></el-input>
          </el-form-item>
          <el-form-item label="新密码" prop="newPassword">
            <el-input type="password" v-model="formInline.newPassword" placeholder="请输入新密码"></el-input>
          </el-form-item>
          <el-form-item label="确认新密码" prop="configPassword">
            <el-input type="password" v-model="formInline.configPassword" placeholder="请再次输入新密码"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button style="background: #148be4 !important;" type="primary" @click="submitchange">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>
<script>
import { computed, onMounted, reactive, ref, toRefs } from 'vue';
import { useStore } from 'vuex';
import stores from '../store/index';
import { useRouter } from 'vue-router';
import screenfull from 'screenfull';
import i18n from '@/plugins/element';
import zhCn from 'element-plus/lib/locale/lang/zh-cn';
import en from 'element-plus/lib/locale/lang/en';
import { changePassword, getPassword } from '@/api/index';
import { getToken, removeToken } from '@/utils/auth';
import { ElMessage, ElMessageBox } from 'element-plus';
export default {
  setup() {
    const state = reactive({
      showScreen: Boolean,
      bgcolor: '#242F42',
      color: '#fff',
      locale: 'zh-cn',
      value: false,
      value2: true,
      date: '',
      time: '',
      week: ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'],
      weeks: ['Sun', 'Mon', 'Tues', 'Wed', 'Thu', 'Fri', 'Sat'],
      weekindex: Number,
      dialogVisible: false,
      formInline: {
        oldPassword: '',
        newPassword: '',
        configPassword: ''
      }
    });
    const rules = {
      oldPassword: [
        {
          required: true,
          message: '请输入',
          trigger: 'blur'
        }
      ],
      newPassword: [{ required: true, message: '请输入', trigger: 'blur' }],
      configPassword: [{ required: true, message: '请输入', trigger: 'blur' }]
    };
    const username = localStorage.getItem('ms_username');
    // const message = 2;

    const store = useStore();
    const collapse = computed(() => store.state.collapse);
    // 侧边栏折叠
    const collapseChage = () => {
      store.commit('handleCollapse', !collapse.value);
    };
    const screen = () => {
      state.showScreen = screenfull.isFullscreen;
      //   console.log(screenfull.isFullscreen, 111);
      screenfull.toggle();
    };
    const change = () => {
      if (state.value === true) {
        state.bgcolor = '#00BFFF';
        stores.commit('common/getSidebarbg', state.bgcolor);
      } else {
        state.bgcolor = '#242F42';
        stores.commit('common/getSidebarbg', state.bgcolor);
      }
    };
    const newupdate = computed(() => {
      setInterval(() => {
        updatetime();
      }, 1000);
    });
    const updatetime = () => {
      var cd = new Date();
      state.weekindex = cd.getDay();
      state.time = zeroPadding(cd.getHours(), 2) + ':' + zeroPadding(cd.getMinutes(), 2) + ':' + zeroPadding(cd.getSeconds(), 2);

      // if (state.locale === zhCn.name) {
      //   state.week = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'];
      // } else {
      //   state.week = ['Sun', 'Mon', 'Tues', 'Wed', 'Thu', 'Fri', 'Sat'];
      // }
      state.date =
        // state.week[cd.getDay()] +
        // '   ' +
        zeroPadding(cd.getFullYear(), 4) + '-' + zeroPadding(cd.getMonth() + 1, 2) + '-' + zeroPadding(cd.getDate(), 2);
      // return state.date;
    };
    const zeroPadding = (num, digit) => {
      var zero = '';
      for (var i = 0; i < digit; i++) {
        zero += '0';
      }
      return (zero + num).slice(-digit);
    };
    const language = () => {
      if (state.locale === en.name) {
        state.locale = zhCn.name;
      } else {
        state.locale = en.name;
      }
    };
    // 获取原密码
    const getpassword = () => {
      getPassword().then(res => {
        if (state.formInline.oldPassword !== res.data.password) {
          ElMessage.warning('原密码输入错误,请重新输入');
          state.formInline.oldPassword = '';
        }
      });
    };
    const router = useRouter();
    // 提交修改密码
    const loginForm = ref(null);
    const submitchange = () => {
      ElMessageBox.confirm('确定修改密码?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          loginForm.value.validate(valid => {
            if (valid) {
              if (state.formInline.newPassword !== state.formInline.configPassword) {
                ElMessage.warning('两次新密码不一致');
              } else {
                changePassword({ token: getToken(), password: state.formInline.configPassword }).then(res => {
                  if (res.code === 200) {
                    ElMessage.success('修改成功,请重新登录');
                    router.push('/login');
                  } else {
                    ElMessage.error(res.msg);
                  }
                });
              }
            }
          });
        })
        .catch(() => {
          ElMessage({
            type: 'info',
            message: '取消修改'
          });
        });
    };
    const open = () => {
      state.dialogVisible = true;
      loginForm.value.resetFields();
    };
    onMounted(() => {
      if (document.body.clientWidth < 1500) {
        collapseChage();
      }
    });

    // 用户名下拉菜单选择事件
    // const router = useRouter();
    const handleCommand = command => {
      if (command == 'loginout') {
        localStorage.removeItem('ms_username');
        router.push('/login');
      } else if (command == 'user') {
        router.push('/user');
      }
    };

    return {
      ...toRefs(state),
      username,
      //   message,
      collapse,
      collapseChage,
      handleCommand,
      screen,
      change,
      zeroPadding,
      updatetime,
      newupdate,
      language,
      rules,
      loginForm,
      submitchange,
      getpassword,
      open
    };
  }
};
</script>
<style lang="scss">
.header {
  position: relative;
  box-sizing: border-box;
  width: 100%;
  height: 70px;
  font-size: 22px;
  color: #fff;
  background: var(--bg);
  display: flex;
  .head_left{
    flex: 0 200px;
  }
  .head_btn {
    // border: 1px solid red;
    flex: 0 calc(100vw - 550px);
    display: flex;
    line-height: 70px;
  }
}
.collapse-btn {
  float: left;
  padding: 0 21px;
  cursor: pointer;
  line-height: 70px;
  background: var(--bg);
}
.header .logo {
  // float: left;
  // width: 250px;
  text-align: center;
  line-height: 70px;
}
.header-right {
  text-align: right;
  padding-right: 50px;
  flex: 0 350px;
  padding-left: 20px;
  // border: 1px solid red;
}
.header-user-con {
  display: flex;
  height: 70px;
  align-items: center;
}
.btn-fullscreen {
  transform: rotate(45deg);
  margin-right: 5px;
  font-size: 24px;
}
.btn-bell,
.btn-fullscreen {
  position: relative;
  width: 30px;
  height: 30px;
  text-align: center;
  border-radius: 15px;
  cursor: pointer;
}
.btn-bell-badge {
  position: absolute;
  right: 0;
  top: -2px;
  width: 8px;
  height: 8px;
  border-radius: 4px;
  background: #f56c6c;
  color: #fff;
}
.btn-bell .el-icon-bell {
  color: #fff;
}
.user-name {
  margin-left: 10px;
}
.user-avator {
  margin-left: 20px;
  margin-right: 20px;
}
.user-avator img {
  display: block;
  width: 40px;
  height: 40px;
  border-radius: 50%;
}
.el-dropdown-link {
  color: #fff;
  cursor: pointer;
}
.el-dropdown-menu__item {
  text-align: center;
}
.screen {
  // margin: 0 15px;
}
.taggle {
  // margin: 0 10px;
}
</style>
