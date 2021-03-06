<template>
  <basic-container>
    <avue-crud :option="option"
               :data="data"
               ref="crud"
               v-model="form"
               :permission="permissionList"
               :before-open="beforeOpen"
               @row-del="rowDel"
               @row-update="rowUpdate"
               @row-save="rowSave"
               @search-change="searchChange"
               @search-reset="searchReset"
               @selection-change="selectionChange"
               @on-load="onLoad">
      <template slot="menuLeft">
        <el-button type="danger"
                   size="small"
                   icon="el-icon-delete"
                   v-if="permission.menu_delete"
                   plain
                   @click="handleDelete">删 除</el-button>
      </template>
      <template slot-scope="{row}"
                slot="roleId">
        <el-tag>{{row.roleName}}</el-tag>
      </template>
      <template slot-scope="{row}"
                slot="deptId">
        <el-tag>{{row.deptName}}</el-tag>
      </template>
    </avue-crud>
  </basic-container>
</template>

<script>
import { getList, remove, update, add, getMenu } from "@/api/system/menu";
import { mapGetters } from "vuex";
export default {
  data() {
    return {
      form: {},
      selectionList: [],
      page: {
        pageSize: 10,
        currentPage: 1,
        total: 0
      },
      option: {
        dialogWidth: "70%",
        tree: true,
        border: true,
        index: true,
        selection: true,
        viewBtn: true,
        column: [
          {
            label: "菜单名称",
            prop: "name",
            search: true
          },
          {
            label: "菜单编号",
            prop: "code",
            search: true
          },
          {
            label: "菜单别名",
            prop: "alias"
          },
          {
            label: "路由地址",
            prop: "path"
          },

          {
            label: "上级菜单",
            prop: "parentId",
            type: "tree",
            dicUrl: "/api/blade-system/menu/tree",
            hide: true,
            props: {
              label: "title"
            }
          },

          {
            label: "菜单类型",
            prop: "category",
            type: "radio",
            dicData: [
              {
                label: "菜单",
                value: 1
              },
              {
                label: "按钮",
                value: 2
              }
            ],
            hide: true
          },
          {
            label: "排序",
            prop: "sort"
          },
          {
            label: "按钮功能",
            prop: "action",
            type: "radio",
            dicData: [
              {
                label: "工具栏",
                value: 0
              },
              {
                label: "操作栏",
                value: 1
              },
              {
                label: "工具操作栏",
                value: 2
              }
            ],
            hide: true
          },
          {
            label: "菜单备注",
            prop: "remark",
            type: "textarea",
            span: 24,
            minRows: 6,
            hide: true
          }
        ]
      },
      data: []
    };
  },

  computed: {
    ...mapGetters(["permission"]),
    permissionList() {
      return {
        addBtn: this.permission.menu_add,
        viewBtn: this.permission.menu_view,
        delBtn: this.permission.menu_delete,
        editBtn: this.permission.menu_edit
      };
    },
    ids() {
      let ids = [];
      this.selectionList.forEach(ele => {
        ids.push(ele.id);
      });
      return ids.join(",");
    }
  },
  methods: {
    rowSave(row, loading) {
      add(row).then(() => {
        loading();
        this.onLoad(this.page);
        this.$message({
          type: "success",
          message: "操作成功!"
        });
      });
    },
    rowUpdate(row, index, loading) {
      update(row).then(() => {
        loading();
        this.onLoad(this.page);
        this.$message({
          type: "success",
          message: "操作成功!"
        });
      });
    },
    rowDel(row) {
      this.$confirm("确定将选删除?", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          return remove(row.id);
        })
        .then(() => {
          this.onLoad(this.page);
          this.$message({
            type: "success",
            message: "操作成功!"
          });
        });
    },
    searchReset() {
      this.onLoad(this.page);
    },
    searchChange(params) {
      this.onLoad(this.page, params);
    },
    selectionChange(list) {
      this.selectionList = list;
    },
    handleDelete() {
      if (this.selectionList.length === 0) {
        this.$message.warning("请选择至少一条数据");
        return;
      }
      this.$confirm("确定将选择账号删除?", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          return remove(this.ids);
        })
        .then(() => {
          this.onLoad(this.page);
          this.$message({
            type: "success",
            message: "操作成功!"
          });
          this.$refs.crud.toggleSelection();
        });
    },
    beforeOpen(done, type) {
      if (["edit", "view"].includes(type)) {
        getMenu(this.form.id).then(res => {
          this.form = res.data.data;
        });
      }
      done();
    },
    onLoad(page, params = {}) {
      getList(page.currentPage, page.pageSize, params).then(res => {
        const data = res.data.data;
        this.data = data;
      });
    }
  }
};
</script>

<style>
</style>
