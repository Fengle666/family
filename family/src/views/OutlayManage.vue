<template>
  <div class="OutlayManage">
    <div class="search">
      <el-form label-width="80px" :inline="true" :model="form">
        <el-row>
          <el-col :span="6">
            <el-form-item :label="$t('支出类型')">
              <el-select v-model="form.costType" placeholder="请选择支出账户" style="width:100%" filterable clearable>
                <el-option v-for="item in costTypeList" :label="item" :value="item" />
              </el-select>
              <!-- <el-input v-model="form.costType" placeholder="请输入支出类型" clearable style="width:100%"></el-input> -->
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item :label="$t('支出时间')">
              <el-date-picker
                v-model="form.date"
                clearable
                placeholder="请选择日期"
                type="date"
                format="YYYY-MM-DD"
                value-format="YYYY-MM-DD"
                style="width: 100%"
              />

              <!-- <el-input v-model="form.date" placeholder="请输入  支出时间" clearable style="width:100%"></el-input> -->
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item :label="$t('支出账户')">
              <el-select v-model="form.account" placeholder="请选择支出账户" style="width:100%" filterable clearable>
                <el-option label="微信" value="微信" />
                <el-option label="支付宝" value="支付宝" />
                <el-option label="银行卡" value="银行卡" />
                <el-option label="现金" value="现金" />
              </el-select>
              <!-- <el-input v-model="form.account" placeholder="请输入支出账号" clearable style="width:100%"></el-input> -->
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item :label="$t('支出描述')">
              <el-input v-model="form.note" placeholder="请输入支出描述" clearable style="width:100%"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item style="text-align:right;width:99%;margin-top:10px">
              <el-button style="background: #148be4 !important;" type="primary" @click="getdatalist">{{$t('查询')}}</el-button>
              <el-button style="background: #148be4 !important;" type="primary" @click="resetFrom">{{$t('重置')}}</el-button>
              <el-button style="background: #148be4 !important;" type="primary" @click="addOutlay">{{$t('新增')}}</el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </div>
    <div class="OutlayManage_table">
      <el-table :data="tableData" v-loading="loading" style="width: 100%" border stripe fit highlight-current-row>
        <!-- <el-table-column prop="id" label="编号" width="80" align="center" /> -->
        <el-table-column prop="costMoney" label="支出金额" align="center" />
        <el-table-column prop="costType" label="支出类型" align="center" />
        <el-table-column prop="account" label="支出账户" align="center" />
        <el-table-column prop="note" label="详情" align="center" />
        <el-table-column prop="createTime" label="支出时间" align="center">
          <template v-slot="scope">
            <span v-if="scope.row.createTime">{{scope.row.createTime.substr(0,10)}}</span>
          </template>
        </el-table-column>
        <!-- <el-table-column prop="familyCode" label="家庭码" width="150" align="center" />
        <el-table-column prop="userId" label="用户编号" width="150" align="center" />-->
        <el-table-column label="操作" align="center" width="150">
          <template v-slot="scope">
            <el-button type="success" size="mini" @click="edit(scope.row)">{{$t('编辑')}}</el-button>
            <!-- <el-button type="primary" size="mini">授权</el-button> -->
            <el-button type="danger" size="mini" @click="del(scope.row)">{{$t('删除')}}</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div style="text-align: center">
        <pagination :total="total" :page="form.currentPage" :limit="form.pageSize" @pagination="getList" />
      </div>
    </div>
    <div>
      <el-dialog v-model="dialogVisible" :title="form.id ? '修改' : '新增'" width="50%">
        <div>
          <el-form label-width="80px" :model="form">
            <el-row>
              <el-col :span="12">
                <el-form-item :label="$t('支出金额')">
                  <el-input v-model="form.money" clearable style="width:100%"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item :label="$t('支出账户')">
                  <!-- <el-input v-model="form.account" clearable style="width:100%"></el-input> -->
                  <el-select v-model="form.account" placeholder="请选择支出账户" style="width:100%" filterable clearable>
                    <el-option label="微信" value="微信" />
                    <el-option label="支付宝" value="支付宝" />
                    <el-option label="银行卡" value="银行卡" />
                    <el-option label="现金" value="现金" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item :label="$t('支出类型')">
                  <!-- <el-input v-model="form.costType" clearable style="width:100%"></el-input> -->
                  <el-radio-group v-model="form.costType" size="small">
                    <el-radio v-for="item in costTypeList" :label="item" :value="item" border></el-radio>
                  </el-radio-group>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item :label="$t('支出描述')">
                  <el-input v-model="form.note" type="textarea" rows="4" clearable style="width:100%"></el-input>
                </el-form-item>
              </el-col>
            </el-row>
          </el-form>
        </div>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="close">{{$t('取消')}}</el-button>
            <el-button style="background: #148be4 !important;" type="primary" @click="submitOutlay">{{$t('确定')}}</el-button>
          </span>
        </template>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import { getOutlaylist, delOutlay, addOutlayDetail, getOutlaylistDetail } from '@/api/index';
import { reactive, toRefs } from '@vue/reactivity';
import { onMounted } from '@vue/runtime-core';
import Pagination from '@/components/Pagination/index.vue';
import { ElMessageBox, ElMessage } from 'element-plus';
export default {
  name: 'donate',
  components: { Pagination },
  setup() {
    const state = reactive({
      form: {
        costType: '',
        date: '',
        account: '',
        note: '',
        currentPage: 1,
        pageSize: 5,
        id: null,
        money: '',
        date: ''
      },
      costTypeList: [
        '餐饮',
        '休闲娱乐',
        '购物',
        '穿搭美容',
        '水果零食',
        '人情社交',
        '生活日用',
        '宠物',
        '养娃',
        '运动',
        '生活服务',
        '住房',
        '爱车',
        '学习',
        '网络虚拟',
        '烟酒',
        '医疗保健',
        '金融保险',
        '酒店旅行',
        '转账',
        '公益',
        '礼金',
        '互助保障',
        '其他'
      ],
      tableData: [],
      loading: false,
      total: 0,
      dialogVisible: false
    });
    const getdatalist = () => {
      const param = {
        costType: state.form.costType,
        date: state.form.date,
        account: state.form.account,
        note: state.form.note,
        currentPage: state.form.currentPage,
        pageSize: state.form.pageSize
      };
      state.loading = true;
      getOutlaylist(param)
        .then(res => {
          state.tableData = res.data.costList;
          state.total = res.data.totalCount;
          state.loading = false;
        })
        .catch(() => {
          state.loading = false;
        });
    };
    // 新增
    const addOutlay = () => {
      state.dialogVisible = true;
      state.form.id = null;
      state.form.costType = '';
      state.form.money = '';
      state.form.note = '';
      state.form.account = '';
    };
    // 重置
    const resetFrom = () => {
      state.form.costType = '';
      state.form.date = '';
      state.form.account = '';
      state.form.note = '';
      getdatalist();
    };
    const close = () => {
      state.dialogVisible = false
      state.form.costType = ''
      state.form.money = ''
      state.form.note = ''
      state.form.account = ''
    };
    // 删除
    const del = row => {
      ElMessageBox.confirm(` 确定删除${row.note}相关数据?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          delOutlay({ id: row.id }).then(res => {
            if (res.code === 200) {
              ElMessage.success('删除成功');
              getdatalist();
            }
          });
        })
        .catch(() => {
          ElMessage({
            type: 'info',
            message: '取消删除'
          });
        });
    };
    const edit = row => {
      state.form.id = row.id;
      state.dialogVisible = true;
      getOutlaylistDetail({ id: row.id }).then(res => {
        state.form.costType = res.data.costType;
        state.form.money = res.data.costMoney;
        state.form.note = res.data.note;
        state.form.account = res.data.account;
      });
    };
    const submitOutlay = () => {
      const param = {
        id: state.form.id,
        money: state.form.money,
        costType: state.form.costType,
        account: state.form.account,
        note: state.form.note
      };
      addOutlayDetail(param).then(res => {
        if (res.code === 200) {
          ElMessage.success('保存成功');
          state.dialogVisible = false;
          state.form.costType = '';
          state.form.money = '';
          state.form.note = '';
          state.form.account = '';
          getdatalist();
        } else {
          ElMessage.error(res.msg);
        }
      });
    };
    const getList = page => {
      state.form.currentPage = page.page;
      state.form.pageSize = page.limit;
      getdatalist();
    };
    onMounted(() => {
      getdatalist();
    });
    return {
      ...toRefs(state),
      getdatalist,
      getList,
      del,
      edit,
      submitOutlay,
      addOutlay,
      resetFrom,
      close
    };
  }
};
</script>

<style lang="scss">
.OutlayManage {
  height: calc(100vh - 140px);
  width: 100%;
  background: #fff;
  border-radius: 5px;
  padding-top: 15px;
  overflow: auto;
  .el-radio__input {
    display: none;
  }
  .el-radio-group {
    .el-radio {
      padding: 0;
      margin: 10px;
      // height: 20px;
      padding: 10px;
      text-align: center;
      .el-radio__label {
        display: inline-block;
        width: 50px;
        text-align: center;
        padding: 0;
      }
    }
  }
  .search {
    padding: 15px;
    margin: 0 15px;
    box-shadow: 0px 2px 4px 0px rgba(0, 0, 0, 0.07);
    border-radius: 3px;
    border: 1px solid #e7e8f2;

    .el-col {
      .el-form-item {
        width: 95%;
        margin-bottom: 0;
      }
      .el-form-item__label {
        border: 1px solid #dcdfe6;
        border-right: none;
        height: 40px;
        background: #f5f7fa;
      }
      .el-input__inner {
        border-radius: 0px;
      }
    }
  }
  .OutlayManage_table {
    margin: 15px;
    padding: 15px;
    box-shadow: 0px 2px 4px 0px rgba(0, 0, 0, 0.07);
    border: 1px solid #e7e8f2;
    border-radius: 3px;
  }
}
</style>