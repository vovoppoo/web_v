<template>
  <div class="edit-dialog-wrap" v-if="ctrlDialog">
    <el-dialog @open="openDialogFn"
               :close-on-click-modal="true"
               :title="titleText"
               width="550px"
               :visible.sync="ctrlDialog">
      <div v-loading="dialogLoading" class="form-wrap">
        <el-form
          :model="formLabelData" label-position="left" :rules="rules" ref="PersonalForm" label-width="150px">
          <el-row :gutter="20">
            <el-col :span="24">

              <el-col :span="22">
                <el-form-item label="上年度决算数(万元)" prop="lastYearAccounts">
                  <el-input :maxlength="50" type="number" placeholder="填写上年度决算数"
                            v-model="formLabelData.lastYearAccounts"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="22">
                <el-form-item label="月累计支付金额(万元)" prop="totalPay">
                  <el-input :maxlength="50" type="number" placeholder="填写月累计支付金额"
                            v-model="formLabelData.totalPay"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="22">
                <el-form-item label="月支付百分比(%)" prop="totalPayPercentage">
                  <el-input :maxlength="50" type="number" placeholder="填写月支付百分比"
                            v-model="formLabelData.totalPayPercentage"></el-input>
                </el-form-item>
              </el-col>


            </el-col>
          </el-row>
        </el-form>

      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="resetForm('PersonalForm')">取 消</el-button>
        <el-button :disabled='!IsButton' :loading="submitFormLoading" @click="submitForm('PersonalForm')"
                   type="primary" class="btn-submit">保 存


        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<style lang="less" rel="stylesheet/less" scoped>

</style>

<script>
  import Api from '../../../api';
  import { dateFormat } from '../../../filter/index';

  export default {
    data() {
      return {
        titleText: '新增',
        isShowUpload: true,
        dialogLoading: false,
        IsButton: true,
        type: '',
        projectOptionsArr: [],
        loading: false,
        obj: {
          budgetMoney: true,
          currentYearPayment: true,
          bidMoney: true
        },
        unitData: [
          {
            label: '深圳市气象局',
            theValue: 1,
          }, {
            label: '深圳市国家气候观象台',
            theValue: 2,
          }, {
            label: '深圳市气象服务中心',
            theValue: 3,
          }, {
            label: '粤港澳大湾区气象监测预警预报中心',
            theValue: 4,
          },
        ],
        objChange: {
          lastYearAccounts: true,
          totalPay: true,
        },
        expenseTypeData: window.ww_FilterList.ExpenseType,
        ctrlDialog: false,
        submitFormLoading: false,
        orgFormData: {},
        formLabelData: {
          recordDate: '',
        },
        projectStateArr: window.ww_FilterList.ProjectStatus,
        purchaseArr: window.ww_FilterList.PurchaseMethod,
        isPayOverdueArr: [{ keyName: '是', keyValue: '1' }, { keyName: '否', keyValue: '2' },],
        backupForm: {},
        imageUrl: '',
        errors: {
          phone: '',
        },
        rules: {
          projectStatus: [
            { required: true, message: '请选择项目状态', trigger: 'blur' },
          ],
          recordDate: [
            { required: true, message: '请选择日期', trigger: 'blur' },
          ],
          detail: [
            { required: true, message: '填写名称', trigger: 'blur' },
          ],
          visitorsNumber: [
            { required: true, message: '请填责任人/出国名单', trigger: 'blur' },
          ],
          money: [
            { required: true, message: '填写经费', trigger: 'blur' },
          ],
          visitorsPayMoney: [
            { required: true, message: '填写参会缴纳经费', trigger: 'blur' },
          ],
          expenseType: [
            { required: true, message: '请选择费用类型', trigger: 'blur' },
          ],

        },
      };
    },
    created() {
      this.backupForm = JSON.parse(JSON.stringify(this.formLabelData));
    },
    watch: {},
    methods: {
      handleSelectSearch(key) {
        this.formLabelData.responsibleId = key[1];
        this.formLabelData.responsible = key[0];
      },

      openDialogFn() {
        if (this.type != 'add') {
          this.dialogLoading = true;
        }
      },

      getDataInfo(val) {
        const _this = this;
        _this.ctrlDialog = true;
        _this.formLabelData = JSON.parse(JSON.stringify(_this.backupForm));
        _this.orgFormData = {};
        if (val) {
          _this.formLabelData = {
            id: val.id,
            lastYearAccounts: (val.lastYearAccounts / 10000).toFixed(4),
            totalPay: (val.totalPay / 10000).toFixed(4),
            totalPayPercentage: `${(val.totalPayPercentage * 100).toFixed(2)}`,
          }
        }
      },

      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            // 处理form数据格式
            const form = JSON.parse(JSON.stringify(this.formLabelData));
            for (let item in this.objChange) {
              if (form[item]) {
                form[item] = form[item] * 10000
              }
            }

            if (form.totalPayPercentage) {
              form.totalPayPercentage = (form.totalPayPercentage / 100).toFixed(4)
            }
            // 提交数据
            this.submitFormLoading = true;
            const type = this.type === 'add' ? 'add' : 'update';
            let typeText = '添加';
            if (type === 'add') {
              Api.updateLastBudget(
                form,
              )
              .then((res) => {
                if (res.code === 0) {
                  this.$message({
                    type: 'success',
                    message: `${typeText}成功!`,
                  });
                  this.$Bus.$emit('editYSOneParamModelData', 0);
                  this.ctrlDialog = false;
                }
              });
            } else if (type === 'update') {
              typeText = '更新';
              Api.updateLastBudget(
                form,
              )
              .then((res) => {
                if (res.code === 0) {
                  this.$message({
                    type: 'success',
                    message: `${typeText}成功!`,
                  });
                  this.$Bus.$emit('editYSOneParamModelData', 0);
                  this.ctrlDialog = false;
                }
              });
            }
            this.submitFormLoading = false;
          } else {
            return false;
          }
        });
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
        this.orgFormData = {};
        this.ctrlDialog = false;
      },
    },
    components: {},
  };
</script>


