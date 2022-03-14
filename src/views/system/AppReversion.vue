<template>
    <div >
      <Card>
        <p slot="title">
        版本管理
          <Button type="primary" size="small" @click="refreshPageManual">
            <Icon type="refresh"></Icon>
            刷新
          </Button>
        </p>

        <!--<Row class="functionWrapper">-->
        <!--  <div class="btnsWrapper clearfix">-->
        <!--    <Button type="primary" @click="handleForm(null,null)">添加</Button>-->
        <!--  </div>-->
        <!--</Row>-->

        <Row >
          <Table
          :columns="column_frist"
          :data="appreversion"
          :loading="ifLoading"
          border>
          </Table>
        </Row>

        <Row class="pageWrapper">
          <Page :total="pageNum"
                class="buttomPage"
                @on-change="changePage"
                                :current="currentPageIdx"
                show-elevator></Page>
        </Row>

        <Modal
            :title="formMap[formName]"
            v-model="formVisible"
            :mask-closable="false"
        >
          <Form :model="formData" :rules="formRules" ref="dataForm">
            <FormItem label="平台：" prop="platform">
              <RadioGroup v-model.trim="formData.platform">
                <Radio label="0">安卓APP</Radio>
                <Radio label="1">苹果APP</Radio>
              </RadioGroup>
            </FormItem>
            <FormItem label="版本：" prop="version">
              <Input v-model="formData.version" placeholder="请输入" clearable></Input>
            </FormItem>
            <FormItem label="下载地址：" prop="downloadUrl">
              <Input :disabled="downloadUrlLoading" v-model="formData.downloadUrl" placeholder="请输入" clearable></Input>
              <div class="demo-spin-col">
                <Spin fix v-if="downloadUrlLoading">
                  <Icon type="load-c" size=18 class="demo-spin-icon-load"></Icon>
                  <div>正在上传</div>
                </Spin>
                <Upload v-else :action="basicUrl+'admin/common/upload/oss/image'"
                        :on-success = 'uploadSuccess'
                        :on-error = "uploadFailed"
                        :on-progress = "uploadLoading"
                        :show-upload-list = "false">
                  <Button type="ghost" icon="ios-cloud-upload-outline">上传文件</Button>
                </Upload>
              </div>
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

import { sysAppRevision, sysAppRevisionSave, BASICURL } from '@/service/getData'

const formJson = {
  id: "",
  platform: "",
  version: "",
  downloadUrl: "",
  remark: ""
};

export default {
  data() {
    return {
      pageNum: null,
      currentPageIdx: 1,
      ifLoading: true,
      showForm: false,
      basicUrl: BASICURL,
      column_frist: [
        {
          title: 'ID',
          key: 'id',
          width: 80
        },
        {
          title: '平台',
          key: 'platform',
          render: (h, obj) => {
            console.log(obj)
            let str = obj.row.platform == 0 ? "安卓APP" : "苹果APP"
            return h ( 'div', str)
          }
        },
        {
          title: '当前版本',
          key: 'version'
        },
        {
          title: '操作',
          render: (h, obj) => {
            return h ( 'div', [
              h('Button',{
                props: {
                  type: 'info',
                  size: 'small'
                },
                style: {
                  'marginRight': '5px'
                },
                on:{
                  click: () =>{
                    this.handleForm(obj.index, obj.row)
                  }

                }
              }, '修改')
            ] )
          }
        },
      ],
      appreversion: [],
      index: null,
      formName: null,
      formMap: {
        add: "新增",
        edit: "编辑"
      },
      formData: formJson,
      formRules: {
        platform: [
          {required: true, message: "请选择平台", trigger: "charge"}
        ],
        version: [
          {required: true, message: "请输入版本", trigger: "blur"}
        ],
        downloadUrl: [
          {required: true, message: "请输入下载链接", trigger: "blur"}
        ],
      },
      formVisible: false,
      formLoading: false,
      downloadUrlLoading: false,
    }
  },

  methods: {
    changePage(pageIndex) {
      this.currentPageIdx = pageIndex;
      this.refreshPage({ pageNo: pageIndex, pageSize: 10 });
    },
    cancelChange() {
      this.$Message.info('已取消修改！');
    },
    refreshPageManual() {
      this.refreshPage({ pageNo: this.currentPageIdx, pageSize: 10 });
    },
    refreshPage() {
      this.ifLoading = true;
      sysAppRevision()
      .then( res => {
        if(!res.code){
          this.appreversion =  res.data.content;
          this.ifLoading = false;
        }else this.$Message.error(res.message);
      }, err => {
        console.log(err);
      })
    },
    // 显示表单
    handleForm(index, row) {
      this.formVisible = true;
      this.formData = JSON.parse(JSON.stringify(formJson));
      if (row !== null) {
        row.platform = row.platform + ""
        this.formData = Object.assign({}, row);
      }
      this.formName = "add";
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
          console.log(data)
          sysAppRevisionSave(data).then(response => {
            this.formLoading = false;
            if (response.code) {
              this.$Message.error(response.message);
              return false;
            }
            this.$Message.success("操作成功");
            this.formVisible = false;
            // 刷新表单
            this.refreshPageManual();
          });
        }
      });
    },
    uploadLoading() {
      this.downloadUrlLoading = true
      this.$Loading.start()
    },
    uploadSuccess(response) {
      this.downloadUrlLoading = false
      this.$Loading.finish()
      this.formData.downloadUrl = response.data
      this.$Message.success('上传成功');
    },
    uploadFailed(error) {
      this.downloadUrlLoading = false
      this.$Loading.finish()
      this.$Message.error('上传失败');
    },
  },
  created () {
    this.refreshPage();
  }
}
</script>

<style lang="less" scoped>
  .demo-spin-col{
    position: relative;
  }
  .demo-spin-icon-load{
    animation: ani-demo-spin 1s linear infinite;
  }
  .permissionWrapper{
    position: absolute;
    z-index: 10;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, .2);
    p{
      position: absolute;
      top: 50%;
      left: 45%;
      transform: -50%;
      font-size: 25px;
      font-style: '黑体';
      text-align: center;
      color: #fff;

    }
  }
</style>
