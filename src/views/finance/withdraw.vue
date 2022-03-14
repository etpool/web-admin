<template>

  <div>
    <Card>

      <p slot="title">
        提币审核管理
        <Button type="primary" size="small" @click="onReset">
          <Icon type="refresh"></Icon>
          刷新
        </Button>
      </p>

      <Row class="functionWrapper">
        <div class="searchWrapper clearfix">
          <div class="poptip">
            <span>邮箱：</span>
            <Input placeholder="请输入" v-model="query.email"/>
          </div>
          <div class="poptip">
            <span>手机号：</span>
            <Input placeholder="请输入" v-model="query.tel"/>
          </div>
          <div class="poptip">
            <span>到账地址：</span>
            <Input placeholder="请输入" v-model="query.address"/>
          </div>
          <div class="poptip">
            <span>币种：</span>
            <Select v-model="query.coinname">
              <Option value="">全部</Option>
              <Option v-for="item in coinList" :value="item.name" :key="item.name">{{ item.name }}</Option>
            </Select>
          </div>

          <div class="poptip">
            <span>协议：</span>
            <Select v-model="query.protocol">
              <Option :value="0">全部</Option>
              <Option v-for="item in protocolList" :value="item.protocol" :key="item.protocol">{{ item.protocolname }}</Option>
            </Select>
          </div>
          <div class="poptip">
            <span>状态：</span>
            <Select v-model="query.status">
              <Option :value="-2">全部</Option>
              <Option :value="-1">驳回</Option>
              <Option :value="0">待处理</Option>
              <Option :value="1">处理中</Option>
              <Option :value="2">已处理</Option>
              <Option :value="3">失败</Option>
            </Select>
          </div>

          <div class="poptip">
            <span>哈希：</span>
            <Input placeholder="请输入" v-model="query.hash"/>
          </div>
          <div class="poptip">
            <span>提交时间：</span>
            <DatePicker type="datetimerange" v-model="addtimeArr" format="yyyy-MM-dd HH:mm:ss" placeholder="请选择" style="width: 300px"></DatePicker>
          </div>
          <div class="poptip">
            <span>审核时间：</span>
            <DatePicker type="datetimerange" v-model="processtimeArr" format="yyyy-MM-dd HH:mm:ss" placeholder="请选择" style="width: 300px"></DatePicker>
          </div>

          <div class="btns">
            <Button type="info" size="small" @click="onSubmit">搜索</Button>
            <Button type="success" size="small" @click="exportExcel">导出</Button>
          </div>
        </div>
      </Row>

      <Row>
        <Table
            :columns="columnList"
            :data="list"
            :loading="loading"
            border>
        </Table>
      </Row>


      <Row class="pageWrapper">
        <Page :total="total"
              class="buttomPage"
              @on-change="changePage"
              :current="query.page"
              show-elevator></Page>
      </Row>

      <Modal
          :title="formMap[formName]"
          v-model="formVisible"
          :mask-closable="false"
      >
        <Form :model="formData" :rules="formRules" ref="dataForm">
          <template v-if="formName === 'add'">
            <FormItem label="币种名称：" prop="coinname">
              <Input disabled v-model="formData.coinname"></Input>
            </FormItem>
            <FormItem label="币种协议：" prop="protocolname">
              <Input disabled  v-model="formData.protocolname"></Input>
            </FormItem>
            <FormItem label="提现地址：" prop="address">
              <Input disabled  v-model="formData.address"></Input>
            </FormItem>
            <FormItem label="提现数量：" prop="money">
              <Input disabled  v-model="formData.money"></Input>
            </FormItem>
            <FormItem label="手续费：" prop="fee">
              <Input disabled  v-model="formData.fee"></Input>
            </FormItem>
            <FormItem label="到账数量：" prop="real_money">
              <Input disabled  v-model="formData.real_money"></Input>
            </FormItem>
          </template>

          <template v-else>
            <FormItem label="驳回原因：" prop="withdrawinfo">
              <Input type="textarea" :autosize="{minRows: 2,maxRows: 5}" v-model="formData.withdrawinfo"></Input>
            </FormItem>
          </template>

        </Form>
        <div slot="footer" v-if="formData.status == 0">
          <Button @click="hideForm">取消</Button>
          <Button type="primary" @click="formSubmit" :loading="formLoading">立即处理</Button>
        </div>
      </Modal>

    </Card>
  </div>

</template>

<script>
import {
  withdrawCoinList,
  withdrawProtocolList,
  withdrawList,
  withdrawSave,
  withdrawListOut
} from '../../service/getData';
import util from "../../libs/util";

const formJson = {
  id: "",
  memberid: 0,
  addtime: 0,
  coinid: 0,
  coinname: "",
  address: "",
  money: 0,
  fee: 0,
  real_money: 0,
  processmold: 0,
  hash: "",
  status: 0,
  processtime: 0,
  withdrawinfo: "",
  remark: "",
  protocol: 0,
  protocolname: "",
};
export default {
  components: {},
  data() {
    return {
      columnList: [
        {
          title: "邮箱/手机号",
          render: (h, obj) => {
            let email = obj.row.email;
            let username = obj.row.username;
            let status = email ? email : username;
            return h(
                "span",
                status
            );
          }
        },
        {
          title: "提现币种",
          key: "coinname"
        },
        {
          title: "协议名称",
          key: "protocolname"
        },
        {
          title: "到账地址",
          key: "address"
        },
        {
          title: "提现数量",
          key: "money"
        },
        {
          title: "手续费",
          key: "fee"
        },
        {
          title: "到账数量",
          key: "real_money"
        },
        {
          title: "提现时间",
          render: (h, obj) => {
            let addtime = obj.row.addtime;
            let status = addtime > 0 ? util.parseTime(addtime) : '--';
            return h(
                "span",
                status
            );
          }
        },
        {
          title: "审核时间",
          render: (h, obj) => {
            let processtime = obj.row.processtime;
            let status = processtime > 0 ? util.parseTime(processtime) : '--';
            return h(
                "span",
                status
            );
          }
        },
        {
          title: "Hash",
          key: "hash"
        },
        {
          title: "状态",
          key: "status",
          render: (h, obj) => {
            let statusD = obj.row.status;
            let status = "";
            let btnType = "success";
            if (statusD == -1) {
              status = "驳回";
              btnType = "error";
            } else if (statusD == 0) {
              status = "待处理";
              btnType = "info";
            } else if (statusD == 1) {
              status = "处理中";
              btnType = "warning";
            } else if (statusD == 2) {
              status = "已处理";
            } else {
              status = "失败";
              btnType = "error";
            }
            return h(
                "Button",
                {
                  props: {
                    type: btnType,
                    size: "small"
                  }
                },
                status
            );
          }
        },
        {
          title: "操作",
          width: 180,
          render: (h, obj) => {
            let list = [];
            if (obj.row.status === 0) {
              list.push(
                  h(
                      "Button",
                      {
                        props: {
                          type: "info",
                          size: "small"
                        },
                        on: {
                          click: () => {
                            this.handleForm(obj.index, obj.row, 'add')
                          }
                        }
                      },
                      "审核"
                  )
              )
              list.push(
                  h(
                      "Button",
                      {
                        props: {
                          type: "info",
                          size: "small"
                        },
                        on: {
                          click: () => {
                            this.handleForm(obj.index, obj.row, 'edit')
                          }
                        }
                      },
                      "驳回"
                  )
              )
            } else if (obj.row.status === -1) {
              list.push(
                  h(
                      "Button",
                      {
                        props: {
                          type: "info",
                          size: "small"
                        },
                        on: {
                          click: () => {
                            this.handleForm(obj.index, obj.row, 'info')
                          }
                        }
                      },
                      "驳回原因"
                  )
              )
            } else if (obj.row.status === 3) {
              list.push(
                  h(
                      "Button",
                      {
                        props: {
                          type: "error",
                          size: "small"
                        },
                        on: {
                          click: () => {
                            this.resetStatus(obj.row)
                          }
                        }
                      },
                      "重新打包"
                  )
              )
            }
            return (
                "div",
                    list
            );
          }
        }
      ],
      query: {
        pageNo: 1,
        pageSize: 10,
        email: "",
        tel: "",
        address: "",
        protocol: 0,
        coinname: "",
        status: -2,
        hash: "",
        startAddTime: "",
        endAddTime: "",
        startProcessTime: "",
        endProcessTime: "",
        isOut: 0,
      },
      addtimeArr: [],
      processtimeArr: [],
      list: [],
      total: 0,
      loading: true,
      coinList: [],
      protocolList: [],
      index: null,
      formName: null,
      formMap: {
        add: "处理提现",
        edit: "驳回提现",
        info: "驳回原因"
      },
      formData: formJson,
      formRules: {
        coinname: [
          {required: true, message: "请选择币种", trigger: "charge"}
        ],
        protocol: [
          {required: true, message: "请选择协议", trigger: "charge"}
        ],
      },
      formVisible: false,
      formLoading: false,
    };
  },
  methods: {
    exportExcel() {
      this.query.isOut = 1
      withdrawListOut(this.query)
          .then(res => {
            // 文件下载
            let blob = new Blob([res], {type: 'application/vnd.ms-excel'})
            let objectUrl = URL.createObjectURL(blob)
            // window.location.href = objectUrl
            const fileName = '提现记录'// 导出文件名
            const elink = document.createElement('a') // 创建a标签
            elink.download = fileName // a标签添加属性
            elink.style.display = 'none'
            elink.href = objectUrl
            document.body.appendChild(elink)
            elink.click() // 执行下载
            URL.revokeObjectURL(elink.href) // 释放URL 对象
            document.body.removeChild(elink) // 释放标签

            this.$Message.success('导出成功')
          })
    },
    changePage(pageIndex) {
      this.query.isOut = 0
      this.query.pageNo = pageIndex
      this.getList()
    },
    onReset() {
      this.query = {
        pageNo: 1,
        pageSize: 10,
        email: "",
        tel: "",
        address: "",
        protocol: 0,
        coinname: "",
        status: -2,
        hash: "",
        startAddTime: "",
        endAddTime: "",
        startProcessTime: "",
        endProcessTime: "",
        isOut: 0,
      };
      this.getList();
    },
    onSubmit() {
      this.$router.push({
        path: "",
        query: this.query
      });
      this.getList();
    },
    getCoinList() {
      withdrawCoinList()
          .then(response => {
            if (response.code) {
              this.$Message.error(response.message);
            }
            this.coinList = response.data || [];
          })
          .catch(() => {
          });
    },
    getProtocolList() {
      withdrawProtocolList()
          .then(response => {
            if (response.code) {
              this.$Message.error(response.message);
            }
            this.protocolList = response.data || [];
          })
          .catch(() => {
          });
    },
    getList() {
      this.loading = true;
      if (this.addtimeArr.length >= 2 && this.addtimeArr[0] && this.addtimeArr[1]) {
        this.query.startAddTime = util.parseTime(this.addtimeArr[0])
        this.query.endAddTime = util.parseTime(this.addtimeArr[1])
      }
      if (this.processtimeArr.length >= 2 && this.processtimeArr[0] && this.processtimeArr[1]) {
        this.query.startProcessTime = util.parseTime(this.processtimeArr[0])
        this.query.endProcessTime = util.parseTime(this.processtimeArr[1])
      }
      withdrawList(this.query)
          .then(response => {
            this.loading = false;
            if (response.code) {
              this.$Message.error(response.message);
            }
            let list = response.data.content || [];
            let tempList = []
            for (let item of list) {
              item.protocol = item.protocol + ""
              tempList.push(item)
            }
            this.list = tempList
            this.total = response.data.totalElements || 0;
          })
          .catch(() => {
            this.loading = false;
            this.list = [];
            this.total = 0;
            this.roles = [];
          });
    },
    // 刷新表单
    resetForm() {
      if (this.$refs["dataForm"]) {
        // 刷新表单
        this.$refs["dataForm"].resetFields();
      }
    },
    // 显示表单
    handleForm(index, row, type) {
      this.formVisible = true;
      this.formData = JSON.parse(JSON.stringify(formJson));
      if (row !== null) {
        this.formData = Object.assign({}, row);
      }
      this.formName = type;
      this.resetForm()
      if (index !== null) {
        this.index = index;
      }
    },
    resetStatus(row) {
      let data = {
        id: row.id,
        status: 1,
      };
      withdrawSave(data).then(response => {
        this.formLoading = false;
        if (response.code) {
          this.$Message.error(response.message);
          return false;
        }
        this.$Message.success("重新打包成功");
        this.getList()
      });
    },
    // 隐藏表单
    hideForm() {
      // 更改值
      this.formVisible = false;
      return true;
    },
    formSubmit() {
      if (this.formLoading) {
        return false
      }
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.formLoading = true;
          let status = 1
          if (this.formName !== 'add') {
            status = -1
          }
          let data = {
            id: this.formData.id,
            status: status,
            withdrawinfo: this.formData.withdrawinfo,
          };
          withdrawSave(data).then(response => {
            this.formLoading = false;
            if (response.code) {
              this.$Message.error(response.message);
              return false;
            }
            this.$Message.success("操作成功");
            this.formVisible = false;
            this.getList()
          });
        }
      });
    },
  },
  mounted() {
  },
  created() {
    // 将参数拷贝进查询对象
    let query = this.$route.query;
    this.query = Object.assign(this.query, query);
    this.query.limit = parseInt(this.query.limit);
    // 加载表格数据
    this.getList();
    // 加载币种列表
    this.getCoinList()
    // 加载协议列表
    this.getProtocolList()
  }
};
</script>

<style>
</style>
