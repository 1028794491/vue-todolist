<template>
  <div id="app">
    <Table border :columns="columns" :data="data"></Table>
    <Page :total="total" :page-size="10" :current="pageNum" size="small" @on-change="changePage" />
    <Button @click="showModal">添加</Button>
    <!-- 添加/修改Modal -->
    <Modal v-model="addModalShow" title="TODOLIST" :footer-hide="true">
      <Form ref="formValidate" :model="formValidate" :label-width="80">
        <FormItem label="权重" prop="weight">
          <InputNumber v-model="formValidate.weight"></InputNumber>
        </FormItem>

        <FormItem label="内容" prop="info">
          <Input
            v-model="formValidate.info"
            type="textarea"
            :autosize="{minRows: 2,maxRows: 5}"
            placeholder="Enter something..."
          ></Input>
        </FormItem>

        <FormItem>
          <Button type="primary" @click="add()" v-if="addFlag">确定</Button>
          <Button type="primary" @click="update()" v-else>确定</Button>
          <Button @click="handleReset('formValidate')" style="margin-left: 8px">取消</Button>
        </FormItem>
      </Form>
    </Modal>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      total: 0, // 数据总量
      pageNum: 1, // 当前展示页码
      addModalShow: false, // 表单modal展示标实
      addFlag: true, // 判断是否为添加
      updateId: 0, // 修改ID
      columns: [
        {
          title: "ID",
          key: "list_id"
        },
        {
          title: "内容",
          key: "list_info"
        },
        {
          title: "权重",
          key: "list_weight"
        },
        {
          title: "操作",
          width: 150,
          align: "center",
          render: (h, params) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.addFlag = false;
                      // 表单赋值
                      this.formValidate.info = params.row.list_info;
                      this.formValidate.weight = params.row.list_weight;
                      this.updateId = params.row.list_id;
                      this.addModalShow = true;
                    }
                  }
                },
                "修改"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small"
                  },
                  on: {
                    click: () => {
                      this.del(params.row.list_id);
                    }
                  }
                },
                "删除"
              )
            ]);
          }
        }
      ],
      data: [],
      formValidate: {
        weight: 1,
        info: ""
      }
    };
  },
  created() {
    this.getlists();
  },
  methods: {
    // 获得列表
    getlists() {
      this.$http(`/apiUrl/list/${this.pageNum}`)
        .then(result => {
          console.log(result);
          this.data = result.data.data;
          this.total = result.data.limits;
        })
        .catch(err => {});
    },
    // 翻页
    changePage(pageNum) {
      this.pageNum = pageNum;
      this.getlists();
    },
    // 显示表单modal
    showModal() {
      this.addModalShow = true; // 显示
      this.addFlag = true; // 修改添加标实
      this.handleReset('formValidate'); // 重置表单
    },
    // 添加
    add() {
      console.log(this.formValidate);
      this.$http.post("/apiUrl/list", this.formValidate).then(result => {
        this.getlists();
        this.addModalShow = false;
      });
    },
    // 重置表单
    handleReset(name) {
      this.$refs[name].resetFields();
      this.addModalShow = false;
    },
    // 删除
    del(id) {
      this.$http
        .delete(`/apiUrl/list/${id}`)
        .then(result => {
          this.getlists();
        })
        .catch(err => {});
    },
    // 修改
    update() {
      this.$http
        .put(`/apiUrl/list/${this.updateId}`, this.formValidate)
        .then(result => {
          this.getlists();
          this.addModalShow = false;
        })
        .catch(err => {});
    }
  }
};
</script>

<style>
</style>
