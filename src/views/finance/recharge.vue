<template>

  <div>
    <Card>

      <p slot="title">
        充币明细
        <Button type="primary" size="small" @click="onReset">
          <Icon type="refresh"></Icon>
          刷新
        </Button>
      </p>

      <Row class="functionWrapper">
        <div class="searchWrapper clearfix">
          <div class="poptip">
            <Poptip trigger="hover" content="请输入充币地址查询" placement="bottom-start">
              <Input placeholder="请输入充币地址查询" v-model="query.address"/>
            </Poptip>
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

    </Card>
  </div>

</template>

<script>
import {rechargeCoinList, rechargeProtocolList, rechargeList, rechargeListOut} from '../../service/getData';
import util from "../../libs/util";

export default {
  components: {},
  data() {
    return {
      columnList: [
        {
          title: "邮箱/手机号",
          width: 100,
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
          title: "充值币种",
          key: "coinname"
        },
        {
          title: "协议名称",
          key: "protocolname"
        },
        {
          title: "充币地址",
          key: "address"
        },
        {
          title: "充值数量",
          key: "money"
        },
        {
          title: "状态",
          key: "status",
          render: (h, obj) => {
            let statusD = obj.row.status;
            let status = "";
            let btnType = "success";
            if (statusD == 0) {
              status = "未到账";
              btnType = "warning";
            } else if (statusD == 1) {
              status = "已到账";
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
          title: "确认数",
          render: (h, obj) => {
            let confirms = obj.row.confirms;
            let nconfirms = obj.row.nconfirms;
            let status = confirms + "/" + nconfirms;
            return h(
                "span",
                status
            );
          }
        },
        {
          title: "到账时间",
          render: (h, obj) => {
            let addtime = obj.row.addtime;
            let status = addtime > 0 ? util.parseTime(addtime) : '--';
            return h(
                "span",
                status
            );
          }
        },
      ],
      query: {
        pageNo: 1,
        pageSize: 10,
        address: "",
        coinname: "",
        protocol: 0,
        isOut: 0,
      },
      list: [],
      total: 0,
      loading: true,
      coinList: [],
      protocolList: [],
    };
  },
  methods: {
    exportExcel() {
      this.query.isOut = 1
      rechargeListOut(this.query)
          .then(res => {
            // 文件下载
            let blob = new Blob([res], {type: 'application/vnd.ms-excel'})
            let objectUrl = URL.createObjectURL(blob)
            // window.location.href = objectUrl
            const fileName = '充值记录'// 导出文件名
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
        address: "",
        coinname: "",
        protocol: 0,
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
      rechargeCoinList()
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
      rechargeProtocolList()
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
      rechargeList(this.query)
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
    }
  },
  mounted() {
  },
  created() {
    // 将参数拷贝进查询对象
    let query = this.$route.query;
    this.query = Object.assign(this.query, query);
    this.query.pageSize = parseInt(this.query.pageSize);
    // 加载表格数据
    this.getList();
    this.getCoinList();
    this.getProtocolList();
  }
};
</script>

<style>
</style>
