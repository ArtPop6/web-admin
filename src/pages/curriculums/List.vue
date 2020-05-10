<template>
  <div class="curriculums">
    <div class="nav">
      <a-select @change="handleChange" class="nav__sele" defaultValue="全部门店" style="width: 220px">
        <a-select-option :value="allshop">{{allshop}}</a-select-option>
        <a-select-option :key="index" :value="list.id" v-for="(list,index) in listdata">{{list.name}}</a-select-option>
      </a-select>
      <a-select @change="typeChange" class="nav__select" defaultValue="全部课程" style="width: 220px">
        <a-select-option :value="allclass">{{allclass}}</a-select-option>
        <a-select-option :key="index" :value="type.id" v-for="(type,index) in typedata">{{type.name}}</a-select-option>
      </a-select>
      <a-input-search @search="onSearch" class="nav__select" placeholder="请输入课程名称" style="width: 260px" />
      <a-button @click="add" class="nva__add" style="width:100px" type="primary">新建课程</a-button>
    </div>
    <div class="table">
      <a-table
        :columns="columns"
        :dataSource="groups"
        :pagination="pagination"
        :rowKey="record => record.id"
        @change="handleTableChange"
      >
        <span slot="state" slot-scope="state">
          <!-- <a-tag :color="state ? '#1890ff' : ''">{{state}}</a-tag> -->
          <a-tag class="blue" v-if="state">已启用</a-tag>
          <a-tag v-if="!state">已禁用</a-tag>
        </span>
        <span slot="operation" slot-scope="text, record">
          <a @click="compile(record)">编辑</a>
          <a-divider type="vertical" />
          <a @click="start(record)" v-if="record.state">禁用</a>
          <a @click="start(record)" v-if="!record.state">启用</a>
          <a-divider type="vertical" />
          <a @click="copy(record)">复制</a>
        </span>
      </a-table>
    </div>
  </div>
</template>

<script>
const columns = [
  {
    title: '课程名称',
    dataIndex: 'course_name',
    key: 'course_name',
    width: '25%'
  },
  {
    title: '授课门店',
    dataIndex: 'stores',
    key: 'stores',
    width: '35%',
    ellipsis: true
  },
  {
    title: '状态',
    key: 'state',
    dataIndex: 'state',
    scopedSlots: { customRender: 'state' },
    width: '15%'
  },
  {
    title: '操作',
    key: 'operation',
    dataIndex: 'operation',
    scopedSlots: { customRender: 'operation' },
    width: '25%'
  }
]
export default {
  components: {},
  data() {
    return {
      allshop: '全部门店',
      allclass: '全部课程',
      groups: [],
      columns,
      listdata: [],
      typedata: [],
      storeid: '',
      categoryid: '',
      coursename: '',
      pagination: {
        total: 0,
        pageSize: 10,
        current: 1,
        showTotal: total => `共有： ${total} 条`
      }
    }
  },
  computed: {},
  methods: {
    list() {
      const groupdata = {
        store_id: this.storeid,
        category_id: this.categoryid,
        course_name: this.coursename,
        page: 1,
        size: 10
      }
      this.$http.get(`/group_exercise/list`, { params: groupdata }).then(res => {
        this.groups = res.data.items
        this.pagination.total = res.data.total
      })
    },
    shoplist() {
      this.$http.get(`/stores/list`).then(res => {
        this.listdata = res.data
      })
    },
    typelist() {
      this.$http.get(`/group_exercise/type`).then(res => {
        this.typedata = res.data
      })
    },
    //选择门店
    handleChange(value) {
      if (value == '全部门店') {
        this.storeid = ''
        this.pagination.current = 1
        this.list()
      } else {
        this.storeid = value
        this.pagination.current = 1
        this.list()
      }
    },
    //选择类型
    typeChange(value) {
      if (value == '全部课程') {
        this.categoryid = ''
        this.pagination.current = 1
        this.list()
      } else {
        this.categoryid = value
        this.pagination.current = 1
        this.list()
      }
    },
    //课程搜索
    onSearch(value) {
      this.coursename = value
      this.pagination.current = 1
      this.list()
    },
    //编辑
    compile(record) {
      this.$router.push({
        name: 'courseAdd',
        query: {
          id: record.id
        }
      })
    },
    copy(record) {
      this.$router.push({
        name: 'courseAdd',
        query: {
          id: record.id,
          copyid: 1
        }
      })
    },
    //启用禁用
    start(record) {
      if (record.state) {
        this.state = false
      } else {
        this.state = true
      }
      const startdata = {
        id: record.id,
        state: this.state
      }
      this.$http.put(`/group_exercise/state?id=${record.id}&state=${this.state}`).then(res => {
        this.pagination.current = 1
        this.list()
        if (record.state) {
          this.$message.success('禁用成功', 1.5)
        } else {
          this.$message.success('启用成功', 1.5)
        }
      })
    },
    //分页
    handleTableChange(page) {
      this.pagination.current = page.current
      const groupdata = {
        store_id: this.storeid,
        category_id: this.categoryid,
        course_name: this.coursename,
        page: page.current,
        size: 10
      }
      this.$http.get(`/group_exercise/list`, { params: groupdata }).then(res => {
        this.groups = res.data.items
        this.pagination.total = res.data.total
      })
    },
    add() {
      this.$router.push({
        name: 'courseAdd'
      })
    }
  },
  created() {
    this.list()
    this.shoplist()
    this.typelist()
  }
}
</script>

<style lang="stylus" scoped>
.curriculums
  display flex
  flex-direction column
  width 100%
  height 100%
  padding 20px 40px

  .nav
    display flex
    align-items center
    width 100%
    height 60px
    background #ffffff

    .nav__sele
      margin-left 40px

    .nav__select
      margin-left 60px

    .nva__add
      margin 0 140px 0 400px

  .table
    width 100%
    min-height 710px
    background #ffffff

  .blue
    background rgb(24, 144, 255)
    color #fff

.ant-pagination
  margin 16px 20px !important
</style>