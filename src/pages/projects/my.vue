<template> <v-container>
    <v-row><v-col cols="6">
        <v-text-field :label="'作品名 包含：' + search.title" v-model="search.title"></v-text-field></v-col>
      <v-col cols="6"><v-text-field :label="'作品描述 包含：' + search.description"
          v-model="search.description"></v-text-field></v-col>
      <v-col cols="6">
        <v-text-field :label="'作品内容 包含：' + search.src" v-model="search.src"></v-text-field></v-col>
      <v-col cols="6">
        <v-number-input control-variant="用户ID" :label="'用户ID 为：' + search.authorid" v-model="userinfo.userid"
          disabled></v-number-input></v-col>

      <v-col cols="3"><v-select v-model="search.order" :items="orderitems" item-title="name" item-value="type"
          label="排序"></v-select></v-col>

      <v-col cols="3"><v-select v-model="search.type" :items="typeitems" item-title="name" item-value="type"
          label="项目类型"></v-select></v-col>
      <v-col cols="3"><v-select v-model="search.state" :items="searchstates" item-title="state" item-value="abbr"
          label="项目状态"></v-select></v-col>
      <v-col cols="3">
        <v-btn @click="onPageChange()"> 搜索 </v-btn>
        <v-btn @click="
          search.order = 'view_up';
        search.type = '';
        search.authorid = '';
        search.src = '';
        search.description = '';
        search.title = '';
        search.state = '';
        onPageChange();
        ">重置</v-btn></v-col></v-row>
    <br />
    <br />



    <Projects :authorid='userinfo.userid' :title="search.title" :description="search.description" :src='search.src'
      :order="search.order.type" :type="search.type.type" ref="Projects" showinfo='true' :state="search.state"
      :actions="[{ name: '信息', function: openinfo }, { name: '编辑', function: openedit }]"></Projects>
    <v-dialog v-model="dialog" max-width="70vw" persistent origin='center center'>


      <v-card prepend-icon="mdi-xml" title="作品信息">
        <v-card-text>
          <v-row dense>
            <v-col cols="12" md="12" sm="12">
              <v-text-field label="标题" required v-model="nowProject.title" hint="将会在首页展示"></v-text-field>
            </v-col>

            <v-col cols="12" md="12" sm="12">
              <v-textarea hint="介绍作品类型，玩法，并向对这个作品有帮助的人致谢！" label="简介" v-model="nowProject.description"></v-textarea>
            </v-col>




            <v-col cols="12" sm="6">
              <v-select :items="['scratch', 'python', 'text']" label="类型" required hint="不建议你改"
                v-model="nowProject.type"></v-select>
            </v-col> <v-col cols="12" sm="6"><v-select v-model="nowProject.state" :items="projectstates"
                item-title="state" item-value="abbr" label="项目状态" hint="我们鼓励开源"></v-select> </v-col>


            <!--<v-col cols="12" sm="6">
              <v-autocomplete
                :items="['Skiing', 'Ice hockey', 'Soccer', 'Basketball', 'Hockey', 'Reading', 'Writing', 'Coding', 'Basejump']"
                label="Interests" auto-select-first multiple v-model="nowProject.tags"></v-autocomplete>
            </v-col>-->
            <v-expansion-panels> <v-expansion-panel> <v-expansion-panel-title>详细数据</v-expansion-panel-title>
                <v-expansion-panel-text>
                  {{ nowProject }} </v-expansion-panel-text>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-row>

        </v-card-text>
        <v-divider></v-divider>

        <v-card-actions> <v-btn color="error" text="删除" variant="tonal" @click="deleteProject"></v-btn>

          <v-spacer></v-spacer>

          <v-btn text="取消" variant="plain" @click="dialog = false"></v-btn>

          <v-btn color="primary" text="保存" variant="tonal" @click="SaveProjectsInfo"></v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </v-container>


</template>

<script>
import openEditor from "../../stores/openEdit";

import request from "../../axios/axios";
import Projects from "../../components/Projects.vue";
import { localuser } from '@/stores/user';
export default {
  components: { Projects },

  data() {
    return {
      projectstates: [
        { state: '私密', abbr: 0 },
        { state: '开源', abbr: 1 },
        { state: '被精选', abbr: 2 },
      ],

      searchstates: [
        { state: '所有', abbr: '' },
        { state: '私密', abbr: 0 },
        { state: '开源', abbr: 1 },
        { state: '被精选', abbr: 2 },
      ],
      typeitems: [
        { name: "所有", type: "" },
        { name: "Scratch", type: "scratch" },
        { name: "Python", type: "python" },
      ],
      orderitems: [
        { name: "观看量升序", type: "view_up" },
        { name: "观看量降序", type: "view_down" },
        { name: "时间升序", type: "time_up" },
        { name: "时间降序", type: "time_down" },
        { name: "序号升序", type: "id_up" },
        { name: "序号降序", type: "id_down" },
      ],
      nowProjectID: 0,
      dialog: false,
      nowProject: {},
      oldProject: {},
      search: {
        title: "",
        type: "",
        description: "",
        src: "",
        order: "view_up",
        authorid: "",
        type: "",

        state: ''
      },
      userinfo: localuser.user,
    };
  },
  methods: {
    projecttypeProps(item) {
      return {
        title: item.name,
        subtitle: item.department,
      }
    },
    onPageChange() {
      this.$refs.Projects.onPageChange(1);

    },
    addtoast(a) {
      this.$toast.add({ severity: 'info', summary: 'info', detail: a, life: 3000 });
    },
    openinfo(id, info) {
      this.nowProjectID = id
      this.nowProject = info
      this.oldProject = { ...info }

      this.dialog = true
    },

    openedit(id, info) {
      this.$toast.add({ severity: 'info', summary: 'info', detail: `尝试打开${info.type}编辑器并编辑${id}号作品`, life: 3000 });
      openEditor(id, info.type)
    },
    deleteProject() {
      this.$toast.add({ severity: 'error', summary: 'info', detail: `尝试删除${this.nowProjectID}号作品`, life: 3000 });
      request({
        url: "/project/deleteProject/" + this.nowProjectID,
        method: "delete",
      }).then((res) => {
        console.log(res)
        this.addtoast('删除成功')
        this.onPageChange()
        this.dialog=false
      }).catch((err) => {
        console.log(err)
        this.addtoast('删除失败')
      })
    },
    SaveProjectsInfo
      () {
      console.log(this.nowProject)
      console.log(this.oldProject)

      /*  request({
          url: "/projects/"+this.nowProjectID,
          method: "put",
          data: this.nowProject,
        }).then((res) => {
          console.log(res)
          this.addtoast('修改成功')
          this.dialog=false
          this.onPageChange()
        }).catch((err) => {
          console.log(err)
          this.addtoast('修改失败')
        })*/
      if (this.nowProject.title !== this.oldProject.title) {
        console.log('修改作品标题为')
        console.log(this.nowProject.title)

        request({
          url: "/project/saveProjcetTitle",
          method: "post",
          data: { id: this.nowProjectID, title: this.nowProject.title },
        }).then((res) => {
          console.log(res)
          this.addtoast('修改成功')
          this.dialog = false
          this.onPageChange()
        }).catch((err) => {
          console.log(err)
          this.addtoast('修改失败')
        })
      }


      if (this.nowProject.description !== this.oldProject.description) {
        console.log('修改作品简介为')
        console.log(this.nowProject.description)

        request({
          url: "/project/setDescription",
          method: "post",
          data: { id: this.nowProjectID, description: this.nowProject.description },
        }).then((res) => {
          console.log(res)
          this.addtoast('修改成功')
          this.dialog = false
          this.onPageChange()
        }).catch((err) => {
          console.log(err)
          this.addtoast('修改失败')
        })
      }
      if (this.nowProject.type !== this.oldProject.type) {
        console.log('修改作品简介为')
        console.log(this.nowProject.type)

        request({
          url: "/project/setType",
          method: "post",
          data: { id: this.nowProjectID, type: this.nowProject.type },
        }).then((res) => {
          console.log(res)
          this.addtoast('修改成功')
          this.dialog = false
          this.onPageChange()
        }).catch((err) => {
          console.log(err)
          this.addtoast('修改失败')
        })
      }

      if (this.nowProject.state !== this.oldProject.state) {
        if (this.nowProject.state == 1) {
          console.log('修改作品状态为公开')

          request({
            url: "/project/share",
            method: "post",
            data: { id: this.nowProjectID },
          }).then((res) => {
            console.log(res)
            this.addtoast('修改成功')
            this.dialog = false
            this.onPageChange()
          }).catch((err) => {
            console.log(err)
            this.addtoast('修改失败')
          })
        }
        if (this.nowProject.state == 0) {
          console.log('修改作品状态为公开')

          request({
            url: "/project/noshare",
            method: "post",
            data: { id: this.nowProjectID },
          }).then((res) => {
            console.log(res)
            this.addtoast('修改成功')
            this.dialog = false
            this.onPageChange()
          }).catch((err) => {
            console.log(err)
            this.addtoast('修改失败')
          })
        }

        if (this.nowProject.state == 2) {
          this.$toast.add
            ({
              severity: "error",
              summary: "错误",
              detail: "好好好自己精选作品是吧",
              life: 3000,
            });
        }
      }

    },
  },
};
</script>
