<template>

  <div>
    <Card>

      <p slot="title">
        币种协议管理
        <Button type="primary" size="small" @click="onReset">
          <Icon type="refresh"></Icon>
          刷新
        </Button>
      </p>


      <Row class="functionWrapper">
        <div class="btnsWrapper clearfix">
          <Button type="primary" @click="handleForm(null,null)">添加协议</Button>
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
          <FormItem label="协议ID：" prop="protocol">
            <Input v-model="formData.protocol" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="协议名称：" prop="protocolname">
            <Input v-model="formData.protocolname" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="RPCServer：" prop="rpcserver">
            <Input v-model="formData.rpcserver" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="RPCUser：" prop="rpcuser">
            <Input v-model="formData.rpcuser" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="RPCPassword：" prop="rpcpassword">
            <Input v-model="formData.rpcpassword" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="区块浏览器：" prop="browser">
            <Input v-model="formData.browser" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="协议主币名称：" prop="symbol">
            <Input v-model="formData.symbol" placeholder="请输入" clearable></Input>
          </FormItem>
          <FormItem label="链ID：" prop="chainid">
            <Input v-model="formData.chainid" placeholder="请输入" clearable></Input>
          </FormItem>
          <!--<FormItem label="状态：" prop="status">-->
          <!--  <RadioGroup v-model.trim="formData.status">-->
          <!--    <Radio :label="1">正常</Radio>-->
          <!--    <Radio :label="0">禁用</Radio>-->
          <!--  </RadioGroup>-->
          <!--</FormItem>-->
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
import {coinprotocolList, coinprotocolSave} from '../../service/getData';

const formJson = {
  id: "",
  protocol: "",
  protocolname: "",
  rpcserver: "",
  rpcuser: "",
  rpcpassword: "",
  browser: "",
  symbol: "",
  chainid: 0,
};
export default {
  components: {},
  data() {
    return {
      columnList: [
        {
          title: "协议编号",
          key: "protocol",
          width: 100
        },
        {
          title: "协议名称",
          key: "protocolname"
        },
        {
          title: "RPCServer",
          key: "rpcserver"
        },
        {
          title: "RPCUser",
          key: "rpcuser"
        },
        {
          title: "RPCPassword",
          key: "rpcpassword"
        },
        {
          title: "区块浏览器",
          key: "browser"
        },
        {
          title: "协议主币名称",
          key: "symbol"
        },
        {
          title: "链ID",
          key: "chainid"
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
      index: null,
      formName: null,
      formMap: {
        add: "新增",
        edit: "编辑"
      },
      formData: formJson,
      formRules: {
        protocol: [
          {required: true, message: "请输入唯一协议ID", trigger: "blur"}
        ],
        protocolname: [
          {required: true, message: "请输入内容", trigger: "blur"}
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
    getList() {
      this.loading = true;
      coinprotocolList(this.query)
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
          coinprotocolSave(data).then(response => {
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
