<template>
  <div>
        <Row>
            <Col :span="24">
                <Form :inline="true">
                    <FormItem>
                        <Select placeholder="会议室" v-model="meetingroom" class="input-search-width" filterable @change="findList()">
                            <Option v-for="item in meetingRoomList" :key="item.roomId" :label="item.roomName"
                            :value="item.roomId"></Option>
                        </Select>
                    </FormItem>
                    <FormItem v-if="false">
                      <DatePicker v-model="selectDate" disabled type="date" :clearable='false' placeholder="日期" style="width: 200px"></DatePicker>
                        <!-- <el-date-picker v-model="selectDate" placeholder="日期" type="date" :picker-options="pickerOptions" clearable class="input-search-width">
                        </el-date-picker> -->
                    </FormItem>
                    <FormItem>
                        <Button type="primary" icon="el-icon-search" @click="findList('', '')">搜索</Button>
                    </FormItem>
                    <FormItem>
                        <Button type="primary" icon="el-icon-circle-plus-outline" @click="orderShow">预约</Button>
                    </FormItem>
                </Form>
            </Col>
            <Col :span="24" class="warp-table" style="margin-top:10px">
                <div style="margin-top:-15px;">
                  <FullCalendar
                    ref="fullCalendar"
                    defaultView="timeGridDay"
                    locale="zh-cn"
                    :header="{
                      left: 'prev,next today',
                      center: 'title',
                      right: 'dayGridMonth,timeGridWeek,timeGridDay,listWeek'
                    }"
                    :business-hours="{
                        startTime: '09:00',
                        endTime:'21:00',
                        daysOfWeek: [ 0, 1, 2, 3, 4, 5, 6 ]
                    }"
                    :height="800"
                    :buttonText="buttonText"
                    :plugins="calendarPlugins"
                    :weekends="calendarWeekends"
                    :events="getCalendarEvents"
                    min-time="09:00:00"
                    max-time="21:00:00"
                    :eventLimit="true"
                    eventLimitText="更多"
                    @dateClick="handleDateClick"
                    @eventClick="handleEventClick"
                    />
                </div>
            </Col>
        </Row>
    <Modal
        v-model="modal1"
        title="预定信息填写"
        @on-ok="commit"
        @on-cancel="modal1 = false">
        <!-- <div class="api" slot="content"> -->
            <Form :inline="true" v-model="orderForm">
              <Row>
                <FormItem>
                    <Select placeholder="会议室" disabled v-model="orderForm.meetingroom" class="input-search-width" filterable>
                        <Option v-for="item in meetingRoomList" :key="item.roomId" :label="item.roomName"
                        :value="item.roomId"></Option>
                    </Select>
                </FormItem>
              </Row>
              <Row>
                <FormItem>
                  <DatePicker v-model="orderForm.selectDate" :clearable='false' type="date" placeholder="日期" style="width: 200px"></DatePicker>
                    <!-- <el-date-picker v-model="selectDate" placeholder="日期" type="date" :picker-options="pickerOptions" clearable class="input-search-width">
                    </el-date-picker> -->
                </FormItem>
              </Row>
              <Row>
                <FormItem>
                  <Input v-model="orderForm.mail" placeholder="请输入邮箱" style="width: 300px" />
                </FormItem>
              </Row>
              <Row>
                <FormItem>
                  <TimePicker :disabled-hours="[0,1,2,3,4,5,6,7,8,22,23,24]" format="HH:mm" :steps="[1, 30]" v-model="orderForm.startTime" placeholder="请输入时间" style="width: 200px"></TimePicker>
                </FormItem>
              </Row>
              <Row>
                <FormItem>
                  <TimePicker  :disabled-hours="disabledhours" format="HH:mm" :steps="[1, 30]" v-model="orderForm.endTime" placeholder="请输入时间" style="width: 200px"></TimePicker>
                </FormItem>
              </Row>
              <!-- <Row>
                <FormItem>
                    <Button type="primary" icon="el-icon-circle-plus-outline" @click="commit()">提交</Button>
                </FormItem>
              </Row> -->
            </Form>
        <!-- </div> -->
    </Modal>
  </div>
</template>

<script>
import FullCalendar from '@fullcalendar/vue'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import interactionPlugin from '@fullcalendar/interaction'
import {
  roomList,
  useInfo,
  pushInfo
} from '@/api/user'

export default {
  name: 'fullcalendar_page',
  components: {
    FullCalendar
  },
  data () {
    return {
      buttonText: {
        today: '今天',
        month: '月',
        week: '周',
        day: '天',
        list: '列表'
      },
      calendarPlugins: [ // plugins must be defined in the JS
        dayGridPlugin,
        timeGridPlugin,
        interactionPlugin // needed for dateClick
      ],
      calendarWeekends: true,
      calendarEvents: [ // initial event data
        // {
        //   title: 'Event Now',
        //   start: new Date(),
        //   end: new Date(new Date().getTime() + 4800000),
        //   color: '#A61000'
        // }
      ],
      calendarApi: null,
      meetingroom: '',
      pickerOptions: {
        disabledDate (time) {
          return time.getTime() < Date.now() - 8.64e7
        }
      },
      selectDate: '',
      orderForm: {
        meetingroom: '',
        selectDate: '',
        startTime: '',
        endTime: '',
        mail: ''
      },
      modal1: false,
      disabledhours: [0, 1, 2, 3, 4, 5, 6, 7, 8, 22, 23, 24],
      meetingRoomList: [{ id: 1, name: '会议室101' }, { id: 2, name: '会议室103' }]
    }
  },
  watch: {
    'calendarApi.component.header.props.title': {
      handler (val, oldval) {
        if (this.calendarApi && this.calendarApi.component && this.calendarApi.component.header && this.calendarApi.component.header.props) {
          const title = this.calendarApi.component.header.props.title
          let beginDate = ''
          let endDate = ''
          let useTime = ''
          if (title.indexOf(' ') !== -1) {
            beginDate = title.slice(0, title.indexOf(' ')).replace('年', '-').replace('月', '-').replace('日', '')
            const endStr = title.slice(title.indexOf(' ') + 3)
            if (endStr.indexOf('月') !== -1) {
              endDate = beginDate.slice(0, 5) + endStr.replace('年', '-').replace('月', '-').replace('日', '')
            } else {
              endDate = beginDate.slice(0, 5) + beginDate.split('-')[1] + '-' + endStr.replace('年', '-').replace('月', '-').replace('日', '')
            }
          } else {
            if (title.indexOf('日') !== -1) {
              useTime = title.replace('年', '-').replace('月', '-').replace('日', '')
            } else {
              beginDate = title.replace('年', '-').replace('月', '-') + '01'
              endDate = title.replace('年', '-').replace('月', '-') + '31'
            }
          }
          this.findList(beginDate, endDate, useTime)
        }
      },
      deep: true
    },
    'orderForm.startTime': {
      handler (val, oldval) {
        const list = []
        for (let i = 0; i < 24; i++) {
          if (i < Number(val.slice(0, 2))) {
            list.push(i)
          }
        }
        this.disabledhours = list
      }
    }
  },
  methods: {
    getCalendarEvents (info, successCallback, failureCallback) {
      const events = [
        ...this.calendarEvents,
        {
          title: info.startStr,
          start: info.start.valueOf()
        }
      ]
      successCallback(events)
    },
    toggleWeekends () {
      this.calendarWeekends = !this.calendarWeekends // update a property
    },
    gotoPast () {
      this.calendarApi.gotoDate('2019-08-01') // call a method on the Calendar object
    },
    handleDateClick (arg) {
      this.modal1 = true
      this.orderForm.meetingroom = this.meetingroom
      this.orderForm.selectDate = this.selectDate
      this.orderForm.startTime = arg.date
      this.orderForm.endTime = ''
      this.orderForm.mail = ''
    },
    commit () {
      if (this.orderForm.startTime === '' || this.orderForm.endTime === '') {
        alert('请输入起止时间段')
      } else {
        this.calendarEvents.push({ // add new event data
          title: this.orderForm.mail,
          start: this.orderForm.startTime,
          end: this.orderForm.endTime
        })
        this.calendarApi.refetchEvents()
        const user = this.orderForm.mail
        let time = this.parseTime(this.orderForm.selectDate, '{y}-{m}-{d}') + ' ' + this.orderForm.startTime
        const startTime = new Date(Date.parse(time.replace(/-/g, '/')))
        time = this.parseTime(this.orderForm.selectDate, '{y}-{m}-{d}') + ' ' + this.orderForm.endTime
        const endTime = new Date(Date.parse(time.replace(/-/g, '/')))
        const useDate = this.parseTime(this.orderForm.selectDate, '{y}-{m}-{d}')
        const roomId = this.meetingroom
        pushInfo({
          user,
          startTime,
          endTime,
          useDate,
          roomId
        }).then(res => {
          if (res.messageCode === '200') {
            alert('预约成功')
            this.findList('', '')
          } else {
            alert('该时间段被占用，请检查')
          }
        })
      }
    },
    handleEventClick (info) {
      alert('start: ' + this.parseTime(info.event.start, '{y}-{m}-{d} {h}:{i}:{s}') + ';  ' +
      'end: ' + this.parseTime(info.event.end, '{y}-{m}-{d} {h}:{i}:{s}'))
    },
    orderShow () {
      if (this.meetingroom === '') {
        alert('请选择要预约的会议室')
      } else if (this.selectDate === '') {
        alert('请选择要预约的日期')
      } else {
        this.modal1 = true
        this.orderForm.meetingroom = this.meetingroom
        this.orderForm.selectDate = this.selectDate
        this.orderForm.startTime = ''
        this.orderForm.endTime = ''
        this.orderForm.mail = ''
      }
    },
    initData () {
      roomList({
      }).then(res => {
        this.meetingRoomList = res.data
        this.meetingroom = res.data[0].roomId
        this.selectDate = new Date()
        this.findList('', '')
      })
    },
    findList (beginDate, endDate, useTime1) {
      this.calendarEvents = []
      this.calendarApi.refetchEvents()
      const roomId = this.meetingroom
      let useTime = ''
      if (beginDate === '' && endDate === '') {
        useTime = useTime1 && useTime1 !== '' ? useTime1 : this.parseTime(this.selectDate, '{y}-{m}-{d}')
      } else {
        useTime = ''
      }
      useInfo({
        roomId,
        useTime,
        beginDate,
        endDate
      }).then(res => {
        if (res.data && res.data.length > 0) {
          res.data.map((item) => {
            const startTime = typeof item.useDate === 'string' ? item.useDate + ' ' + item.startTime : this.parseTime(item.useDate, '{y}-{m}-{d}') + ' ' + item.startTime
            const endTime = typeof item.useDate === 'string' ? item.useDate + ' ' + item.endTime : this.parseTime(item.useDate, '{y}-{m}-{d}') + ' ' + item.endTime
            const obj = {
              title: item.user,
              start: new Date(Date.parse(startTime.replace(/-/g, '/'))),
              end: new Date(Date.parse(endTime.replace(/-/g, '/'))),
              color: '#A61000'
            }
            this.calendarEvents.push(obj)
          })
          this.calendarApi.refetchEvents()
        }
      })
    },
    parseTime (time, fm) { // 解析时间  time: 时间戳或者实践对象 fm: 格式 默认是{y}-{m}-{d} {h}:{i}:{s}
      if (arguments.length === 0) {
        return null
      }
      const format = fm || '{y}-{m}-{d} {h}:{i}:{s}'
      let date
      if (typeof time === 'object') {
        date = time
      } else {
        if (('' + time).length === 10) time = parseInt(time) * 1000
        date = new Date(time)
      }
      const formatObj = {
        y: date.getFullYear(),
        m: date.getMonth() + 1,
        d: date.getDate(),
        h: date.getHours(),
        i: date.getMinutes(),
        s: date.getSeconds(),
        a: date.getDay()
      }
      const time_str = format.replace(/{(y|m|d|h|i|s|a)+}/g, (result, key) => {
        let value = formatObj[key]
        if (key === 'a') return ['一', '二', '三', '四', '五', '六', '日'][value - 1]
        if (result.length > 0 && value < 10) {
          value = '0' + value
        }
        return value || 0
      })
      return time_str
    }
  },
  mounted () {
    this.calendarApi = this.$refs.fullCalendar.getApi()
    this.initData()
  }
}
</script>

<style lang='less'>
// you must include each plugins' css
// paths prefixed with ~ signify node_modules
@import '~@fullcalendar/core/main.css';
@import '~@fullcalendar/daygrid/main.css';
@import '~@fullcalendar/timegrid/main.css';
</style>
