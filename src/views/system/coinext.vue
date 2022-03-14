<template>

  <div>
    <Card>

      <p slot="title">
        币种扩展管理
        <Button type="primary" size="small" @click="onReset">
          <Icon type="refresh"></Icon>
          刷新
        </Button>
      </p>

      <Row class="functionWrapper">
        <div class="searchWrapper clearfix">
          <div class="poptip">
            <span>币种名称：</span>
            <Input placeholder="请输入" v-model="query.coinname"/>
          </div>
          <div class="poptip">
            <span>合约地址：</span>
            <Input placeholder="请输入" v-model="query.ext"/>
          </div>

          <div class="btns">
            <Button type="info" size="small" @click="onSubmit">搜索</Button>
          </div>
          <div class="btns">
            <Button type="primary" @click="handleForm(null,null)">添加扩展</Button>
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
              :pageSize="query.pageSize"
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
          <div style="font-weight: 600;margin-bottom: 10px">币种基础信息</div>
          <FormItem label="是否启用：" prop="status">
            <RadioGroup v-model.trim="formData.status">
              <Radio :label="1">启用</Radio>
              <Radio :label="0">禁用</Radio>
            </RadioGroup>
          </FormItem>
          <FormItem label="币种名称：" prop="coinname">
            <Select v-model="formData.coinname" style="width:260px">
              <Option v-for="item in coinList" :value="item.name" :key="item.name">{{ item.name }}</Option>
            </Select>
          </FormItem>
          <FormItem label="币种协议：" prop="protocol">
            <Select v-model="formData.protocol" style="width:260px">
              <Option v-for="item in protocolList" :value="item.protocol + ''" :key="item.protocol">{{ item.protocolname }}</Option>
            </Select>
          </FormItem>
          <FormItem label="主地址：" prop="mainaddress">
            <br/>
            <div style="color: red;">该地址为提现出币私钥，请使用加密工具加密后填写，为保证资金安全,请勿将大量资金存入该地址。</div>
            <Input v-model="formData.mainaddress" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="合约地址：" prop="ext">
            <br/>
            <div style="color: red;">该地址为代币合约地址，币种为主币时可不填</div>
            <Input v-model="formData.ext" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="币种精度：" prop="decimals">
            <br/>
            <div style="color: red;">请准确填写币种精度，否则会导致充值不到账/金额不对</div>
            <Input v-model="formData.decimals" placeholder="请输入" clearable></Input>
          </FormItem>

          <div style="font-weight: 600;margin-bottom: 10px">充值设置</div>
          <FormItem label="是否可充值：" prop="isrecharge">
            <RadioGroup v-model.trim="formData.isrecharge">
              <Radio :label="1">启用</Radio>
              <Radio :label="0">禁用</Radio>
            </RadioGroup>
          </FormItem>
          <FormItem label="最低充值金额：" prop="minrecharge">
            <br/>
            <div style="color: red;">用户充值金额低于该金额，则不入账</div>
            <Input v-model="formData.minrecharge" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="充值确认数：" prop="confirms">
            <br/>
            <div style="color: red;">该值为区块确认多少次为到账</div>
            <Input v-model="formData.confirms" placeholder="请输入" clearable></Input>
          </FormItem>

          <div style="font-weight: 600;margin-bottom: 10px">提现设置</div>
          <FormItem label="是否可提现：" prop="iswithdraw">
            <RadioGroup v-model.trim="formData.iswithdraw">
              <Radio :label="1">启用</Radio>
              <Radio :label="0">禁用</Radio>
            </RadioGroup>
          </FormItem>
          <FormItem label="是否自动审核：" prop="isautowithdraw">
            <RadioGroup v-model.trim="formData.isautowithdraw">
              <Radio :label="1">启用</Radio>
              <Radio :label="0">禁用</Radio>
            </RadioGroup>
          </FormItem>
          <FormItem label="提现费率：" prop="withdrawfee">
            <br/>
            <div style="color: red;">提现费率(1=100%) ，提现手续费为内扣方式</div>
            <Input v-model="formData.withdrawfee" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="最低手续费：" prop="minwithdrawfee">
            <br/>
            <div style="color: red;">最低手续费数量(如果收取固定手续费,将提现费率设置为0即可) :手续费为内扣方式</div>
            <Input v-model="formData.minwithdrawfee" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="最小提现数量：" prop="minwithdraw">
            <Input v-model="formData.minwithdraw" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="最大提现数量：" prop="maxwithdraw">
            <Input v-model="formData.maxwithdraw" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="备注码充值的地址：" prop="memoaddress">
            <br/>
            <div style="color: red;">用户填写备注码充值的地址 如果不需要填空（如EOS）</div>
            <Input v-model="formData.memoaddress" placeholder="请输入" clearable></Input>
          </FormItem>
        </Form>
        <div slot="footer">
          <Button @click="hideForm">取消</Button>
          <Button type="primary" @click="formSubmit" :loading="formLoading">确定</Button>
        </div>
      </Modal>

    </Card>
  </div>

</template>

<script>
import {coinextCoinList, coinextProtocolList, coinextList, coinextSave} from '../../service/getData';

const formJson = {
  id: "",
  coinid: 0,
  coinname: "",
  protocol: "",
  protocolname: "",
  mainaddress: "",
  ext: "",
  decimals: 6,
  status: 1,
  withdrawfee: 0,
  minwithdrawfee: 0,
  iswithdraw: 1,
  isrecharge: 1,
  isautowithdraw: 0,
  minwithdraw: 0,
  maxwithdraw: 0,
  minrecharge: 0,
  confirms: 3,
  memoaddress: ""
};
export default {
  components: {},
  data() {
    return {
      columnList: [
        {
          title: "币种名称",
          key: "coinname",
          width: 100
        },
        {
          title: "协议名称",
          key: "protocolname"
        },
        {
          title: "合约地址",
          key: "ext"
        },
        {
          title: "币种精度",
          key: "decimals"
        },
        {
          title: "状态",
          key: "status",
          render: (h, obj) => {
            let statusD = obj.row.status;
            let status = "";
            let btnType = "success";
            if (statusD == 1) {
              status = "启用";
            } else {
              status = "禁用 ";
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
          title: "提现状态",
          key: "iswithdraw",
          render: (h, obj) => {
            let statusD = obj.row.iswithdraw;
            let status = "";
            let btnType = "success";
            if (statusD == 1) {
              status = "启用";
            } else {
              status = "禁用 ";
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
          title: "充值状态",
          key: "isrecharge",
          render: (h, obj) => {
            let statusD = obj.row.isrecharge;
            let status = "";
            let btnType = "success";
            if (statusD == 1) {
              status = "启用";
            } else {
              status = "禁用 ";
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
          title: "确认数",
          key: "confirms"
        },
        {
          title: "操作",
          width: 80,
          render: (h, obj) => {
            return (
                "div",
                    [
                      h(
                          "Button",
                          {
                            props: {
                              type: "info",
                              size: "small"
                            },
                            on: {
                              click: () => {
                                this.handleForm(obj.index, obj.row)
                              }
                            }
                          },
                          "修改"
                      )
                    ]
            );
          }
        }
      ],
      query: {
        ext: "",
        coinname: "",
        pageNo: 1,
        pageSize: 10,
      },
      list: [],
      total: 0,
      loading: true,
      coinList: [],
      protocolList: [],
      index: null,
      formName: null,
      formMap: {
        add: "新增",
        edit: "编辑"
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
    changePage(pageIndex) {
      this.query.pageNo = pageIndex
      this.getList()
    },
    onReset() {
      this.query = {
        ext: "",
        coinname: "",
        pageNo: 1,
        pageSize: 10,
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
      coinextCoinList()
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
      coinextProtocolList()
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
      coinextList(this.query)
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
    handleForm(index, row) {
      // 加载币种列表
      this.getCoinList()
      // 加载协议列表
      this.getProtocolList()
      this.formVisible = true;
      this.formData = JSON.parse(JSON.stringify(formJson));
      if (row !== null) {
        this.formData = Object.assign({}, row);
      }
      this.formName = "add";
      this.resetForm()
      if (index !== null) {
        this.index = index;
        this.formName = "edit";
      }
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
          let data = Object.assign({}, this.formData);
          if (this.formName === "add") {
            delete data.id
          }
          coinextSave(data).then(response => {
            this.formLoading = false;
            if (response.code) {
              this.$Message.error(response.message);
              return false;
            }
            this.$Message.success("操作成功");
            this.formVisible = false;
            if (this.formName === "add") {
              // 向头部添加数据
              if (response.data && response.data.id) {
                data.id = response.data.id;
                this.list.unshift(data);
              }
            } else {
              this.list.splice(this.index, 1, data);
            }
            // 刷新表单
            this.getList();
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
  }
};
</script>

<style>
</style>
