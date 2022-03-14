<template>

  <div>
    <Card>

      <p slot="title">
        归集配置管理
        <Button type="primary" size="small" @click="onReset">
          <Icon type="refresh"></Icon>
          刷新
        </Button>
      </p>


      <Row class="functionWrapper">
        <div class="btnsWrapper clearfix">
          <Button type="primary" @click="handleForm(null,null)">添加归集配置</Button>
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
          <FormItem label="最低归集数量：" prop="minnum">
            <Input v-model="formData.minnum" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="归集地址：" prop="address">
            <br/>
            <div style="color: red;">请使用加密工具加密后填写</div>
            <Input v-model="formData.address" placeholder="请输入" clearable></Input>
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
import {automainconfigCoinList, automainconfigProtocolList, automainconfigList, automainconfigSave} from '../../service/getData';

const formJson = {
  id: "",
  coinid: 0,
  coinname: "",
  protocol: "",
  protocolname: "",
  minnum: 0,
  address: "",
};
export default {
  computed: {
    comProtocolName() {
      return (protocol) => {
        let item = this.protocolList.find(o => o.protocol == protocol)
        return item.protocolname
      }
    }
  },
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
          key: "protocolname",
          render: (h, obj) => {
            let statusD = obj.row.protocol;
            return h('div', this.comProtocolName(statusD))
          }
        },
        {
          title: "最低归集数量",
          key: "minnum"
        },
        {
          title: "归集地址",
          key: "address"
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
        status: "-1",
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
        status: "-1",
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
      automainconfigCoinList()
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
      automainconfigProtocolList()
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
      automainconfigList(this.query)
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
          automainconfigSave(data).then(response => {
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
    this.getProtocolList();
  }
};
</script>

<style>
</style>
