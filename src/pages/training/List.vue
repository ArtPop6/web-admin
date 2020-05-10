<template>
  <a-layout class="container">
    <a-layout-header class="btn-group-list">
      <a-row :gutter="8" justify="space-around" type="flex">
        <a-col :span="4" align="left">
          <a-select @change="handleChange" class="nav__sele" defaultValue="全部门店" style="width: 190px">
            <a-select-option :value="allshop">{{allshop}}</a-select-option>
            <a-select-option :key="index" :value="list.id" v-for="(list,index) in listdata">{{list.name}}</a-select-option>
          </a-select>
        </a-col>
        <a-col :span="3" align="left">
          <a-select
            :filterOption="filterOption"
            @change="coach"
            optionFilterProp="children"
            placeholder="请输入教练名"
            showSearch
            style="width: 140px"
          >
            <a-select-option :value="allcoach">{{allcoach}}</a-select-option>
            <a-select-option :key="index" :value="coach.id" v-for="(coach,index) in coachdata">{{coach.name}}</a-select-option>
          </a-select>
        </a-col>
        <a-col :span="3" align="left">
          <a-select
            :filterOption="filterOption"
            @change="course"
            optionFilterProp="children"
            placeholder="请输入课程名"
            showSearch
            style="width: 140px"
          >
            <a-select-option :value="allcourse">{{allcourse}}</a-select-option>
            <a-select-option :key="index" :value="cour.id" v-for="(cour,index) in courdata">{{cour.name}}</a-select-option>
          </a-select>
        </a-col>
        <a-col :span="4" align="left">
          <a-input placeholder="会员名字/手机" v-model="users" />
        </a-col>
        <a-col :span="5">
          <a-range-picker :placeholder="['开始日期', '结束日期']" @change="onChange" style="width: 100%" />
        </a-col>
        <a-col :span="4">
          <a-button @click="handleSearch" icon="search" type="primary">查询</a-button>
        </a-col>
      </a-row>
    </a-layout-header>
    <a-layout-content class="table-container">
      <a-table
        :columns="columns"
        :dataSource="data"
        :pagination="pagination"
        :rowKey="record => record.id"
        @change="handleTableChange"
      >
        <div slot="avatar" slot-scope="avatar, record">
          <div style="width:50px;float:left">
            <a-avatar :src="record.avatar" icon="user" shape="square" size="large" style="margin-right:10px" />
          </div>
          <div style="margin-left:5px;width:250px;">
            <div>
              <a>姓名: {{ record.user_name }}</a>
            </div>
            <a-tag color="blue" v-if="record.gender == 'MALE'">性别：男</a-tag>
            <a-tag color="pink" v-if="record.gender == 'FEMALE'">性别：女</a-tag>
          </div>
        </div>
        <div slot="store_name" slot-scope="store_name, record">
          <div>{{record.store_name}}：{{record.coach_name}}</div>
        </div>
        <div slot="begin_date" slot-scope="begin_date, record">
          <div>{{record.course_name}}：{{record.begin_date}}</div>
        </div>
        <div slot="log" slot-scope="text">点击+查看日志</div>
        <div class="record_coulm" slot="expandedRowRender" slot-scope="record" style="margin: 0">
          <div v-if="record.pt_diary_imgs==null">暂无课程规划、训练日志</div>
          <div :key="index" class="record_log" v-for="(item,index) in record.pt_diary_imgs">
            <div class="record_imss">
              <img
                :class="{'active':ims.file_store_id===selectIndex}"
                :key="index"
                :src="ims.url"
                @click="imgScc(ims.file_store_id)"
                v-for="(ims,index) in item.imgs"
              />
            </div>
            <div class="record_text">
              <span class="record_title">{{item.title}}</span>
              <span class="record_descr">{{item.description}}</span>
            </div>
          </div>
        </div>
      </a-table>
    </a-layout-content>
  </a-layout>
</template>
<script>
const columns = [
  { title: '会员', dataIndex: 'avatar', key: 'avatar', width: '25%', scopedSlots: { customRender: 'avatar' } },
  {
    title: '手机',
    dataIndex: 'phone',
    key: 'phone'
  },
  {
    title: '门店 | 教练',
    key: 'store_name',
    dataIndex: 'store_name',
    scopedSlots: { customRender: 'store_name' }
  },
  {
    title: '课程 | 时间',
    key: 'begin_date',
    dataIndex: 'begin_date',
    scopedSlots: { customRender: 'begin_date' }
  },
  {
    title: '训练日志',
    dataIndex: 'log',
    key: 'log',
    scopedSlots: { customRender: 'log' }
  }
]
export default {
  data() {
    return {
      allshop: '全部门店',
      allcoach: '全部教练',
      allcourse: '全部课程',
      userName: '',
      data: [],
      columns,
      date: '',
      className: '',
      coachid: '',
      users: '',
      begin_time: '',
      end_time: '',
      storeid: '',
      courseid: '',
      selectIndex: undefined,
      listdata: [],
      coachdata: [],
      courdata: [],
      pagination: {
        total: 0,
        pageSize: 10,
        current: 1,
        showTotal: total => `共有： ${total} 条`
      }
    }
  },
  methods: {
    //获取数据
    list() {
      const listdatas = {
        coach_id: this.coachid,
        user: this.users,
        begin_time: this.begin_time,
        end_time: this.end_time,
        store_id: this.storeid,
        course_id: this.courseid,
        page: 1,
        size: 10
      }
      this.$http.get(`/pt/training_log`, { params: listdatas }).then(res => {
        this.data = res.data.items
        this.pagination.total = res.data.total
      })
    },
    //分页
    handleTableChange(page) {
      this.pagination.current = page.current
      const listdatas = {
        coach_id: this.coachid,
        user: this.users,
        begin_time: this.begin_time,
        end_time: this.end_time,
        store_id: this.storeid,
        course_id: this.courseid,
        page: page.current,
        size: 10
      }
      this.$http.get(`/pt/training_log`, { params: listdatas }).then(res => {
        this.data = res.data.items
        this.pagination.total = res.data.total
      })
    },
    // 获取门店
    shopList() {
      this.$http.get(`/stores/list`).then(res => {
        this.listdata = res.data
      })
    },
    // 获取教练
    coachList() {
      this.$http.get(`/coaches/list`).then(res => {
        this.coachdata = res.data
      })
    },
    // 获取课程
    courseList() {
      this.$http.get(`/pt/course_list`).then(res => {
        this.courdata = res.data
      })
    },
    // 查询教练
    coach(value) {
      if (value == '全部教练') {
        this.coachid = ''
        this.list()
      } else {
        this.coachid = value
        this.list()
      }
    },
    // 查询课程
    course(value) {
      if (value == '全部课程') {
        this.courseid = ''
        this.list()
      } else {
        this.courseid = value
        this.list()
      }
    },
    filterOption(input, option) {
      return option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
    },
    // 选择日期
    onChange(date, dateString) {
      this.begin_time = dateString[0]
      this.end_time = dateString[1]
    },
    // 查询
    handleSearch() {
      this.list()
    },
    //选择门店
    handleChange(value) {
      if (value == '全部门店') {
        this.storeid = ''
        this.list()
      } else {
        this.storeid = value
        this.list()
      }
    },
    imgScc: function(index) {
      this.selectIndex = this.selectIndex === index ? undefined : index
    }
  },
  created() {
    this.list()
    this.shopList()
    this.coachList()
    this.courseList()
  }
}
</script>
<style lang="stylus">
.container
  padding 15px
  min-height 700px
  background #F0F2F5

  .btn-group-list
    background #fff
    box-shadow -2px 3px 10px rgb(212, 212, 212)

    button
      margin-right 8px

  .table-container
    background #fff
    box-shadow -2px 3px 10px rgb(212, 212, 212)
    margin-top 10px

    .record_coulm
      display flex
      justify-content space-around

      .record_log
        display flex
        flex-direction column
        align-items center

        .record_imss
          display flex

          img
            height 200px
            width 200px
            margin-right 5px
            transition all 0.5s linear

          img.active
            position relative
            top 280px
            transform scale(3)
            position absolute
            z-index 100

          .record-img img:hover
            width 200px
            height 200px

        .record_text
          display flex
          flex-direction column

          .record_title
            padding-top 10px
            text-align center
            width 200px

          .record_descr
            padding-top 10px
            max-width 200px
            word-break break-all
</style>