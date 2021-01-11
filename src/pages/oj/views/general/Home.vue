<template>
  <Col>
    <transition name="fadeInUp" mode="out-in">
      <Row type="flex" justify="space-around" v-show = "listVisible == true">
        <Col :span="19">
          <el-carousel :interval="5000" type="card" height="350px">
            <div v-if="contests.length" class="contest">
              <el-carousel-item v-for="(contest, index) of contests" :key="index" >
                <div class="contest-content">
                  <h2 style="position: absolute; top:40px; left: 20px; text-align:center">
                    <Button type="text"  class="contest-title" @click="goContest(index)">{{contests[index].title | ellipsis}}
                    </Button>
                    <Button size="small" shape="circle" :type="contest.status === '0' ? 'success' : 'warning'" >{{$t('m.' + CONTEST_STATUS_REVERSE[contest.status].name.replace(/ /g, "_"))}}
                    </Button>
                    <Button type="info" shape="circle" size="small" icon="calendar">
                      {{contest.start_time | localtime('YYYY-M-D HH:mm') }}
                    </Button>
                    <Button shape="circle" size="small" icon="android-time">
                      {{getDuration(contest.start_time, contest.end_time)}}
                    </Button>
                    <Button shape="circle" size="small" icon="trophy">
                      {{contest.rule_type}}
                    </Button>
                  </h2> 
                  <div class="contest-content-description">
                    <blockquote v-html="contest.description.slice(0, 400)"></blockquote>...
                  </div>
                  <ButtonGroup shape="circle" class="btn-group">
                    <Button :type="contest.status === '0' ? 'success' : 'warning'" @click="goContest(index)">
                      {{ contest.status === '0' ? $t('m.Enter_Contest') : $t('m.Preview_Contest') }}
                      <Icon type="chevron-right"></Icon>
                    </Button>
                </ButtonGroup>
                </div>
                
              </el-carousel-item> 
            </div>
            <el-carousel-item>
              <img src="/public/upload/img1.png" alt="" style="width:100%;height:100%"></a>
            </el-carousel-item>
            <el-carousel-item>
              <img src="/public/upload/img2.png" alt="" style="width:100%;height:100%"></a>
            </el-carousel-item>
            <el-carousel-item>
              <img src="/public/upload/img3.png" alt="" style="width:100%;height:100%">
            </el-carousel-item>
          
          </el-carousel>
          <!-- <div class="home-pick-one">
          </div> -->
        </Col>
        <Col :span="5">
          <Panel shadow style="float:right; margin-right:0%; width:100%; hight:200px">
            <div style="margin:0 10%; font-size:18px; text-align:left; width:100%; line-height:18px; color:#636e72;">Today is: </div>
            <Layout>
              <Sider style="width:25%; min-width:25%; max-width:25%; flex: 0 0 25%; background: white;">
                <div style="margin:50% 65%; font-size:18px; text-align:center; width:12px; line-height:18px; background: white; color:#636e72;margin-top: 64px;">{{nowMouth}}</div>
              </Sider>
              <Content style="background: white;">
                <div style="margin:0 auto; font-size:120px; text-align:center; background: white; color:rgb(73, 80, 96);">
                <strong>{{nowDate}}</strong>
                </div>
              </Content>
              <Sider style="width:25%; min-width:25%; max-width:25%; flex: 0 0 25%; background: white;">
                <div style="margin:50% 15%; font-size:18px; text-align:center; width:12px; line-height:18px; background: white; color:#636e72;margin-top: 64px;">{{nowWeek}}</div>
              </Sider>
            </Layout>
            <div style="margin-top:-10px; margin:0 auto; font-size:16px; text-align:center; width:80%; line-height:18px; background: transparent; color:#636e72;margin-top: 13px;">{{word}}</div>
            <Button type="primary" icon="ios-color-wand"  @click="Sighin" :class="this.wordcd ? 'disabled':''" long style="margin-top:20px; margin-bottom:20px; margin-left:10%; width:80%;">{{Omi}}</Button>
          </Panel>
        </Col>
      </Row>
    </transition>
    <Row type="flex" justify="space-around">
      <Col :span="24">
        <Panel shadow :padding="10">
          <div slot="title">
            {{title}}
          </div>
          <div slot="extra">
            <Button v-if="listVisible" type="info" @click="init" :loading="btnLoading">{{$t('m.Refresh')}}</Button>
            <Button v-else type="ghost" icon="ios-undo" @click="goBack">{{$t('m.Back')}}</Button>
          </div>

          <transition-group name="announcement-animate" mode="in-out">
            <div class="no-announcement" v-if="!announcements.length" key="no-announcement">
              <p>{{$t('m.No_Announcements')}}</p>
            </div>
            <template v-if="listVisible">
              <ul class="announcements-container" key="list">
                <li v-for="announcement in announcements" :key="announcement.title">
                  <div class="flex-container">
                    <div class="title"><a class="entry" @click="goAnnouncement(announcement)">
                      {{announcement.title}}</a></div>
                    <div class="date">{{announcement.create_time | localtime }}</div>
                    <div class="creator"> {{$t('m.By')}} {{announcement.created_by.username}}</div>
                  </div>
                </li>
              </ul>
              <Pagination v-if="!isContest"
                          key="page"
                          :total="total"
                          :page-size="limit"
                          @on-change="getAnnouncementList">
              </Pagination>
            </template>
            <template v-else>
              <div v-katex v-html="announcement.content" key="content" class="content-container markdown-body"></div>
            </template>
          </transition-group>
        </Panel>
      </Col>
      
    </Row>
  </Col>
</template>

<script>
  // import Announcements from './Announcements.vue'
  import api from '@oj/api'
  import time from '@/utils/time'
  import { STORAGE_KEY, CONTEST_STATUS, CONTEST_STATUS_REVERSE } from '@/utils/constants'
  import axios from 'axios'
  import 'element-ui/lib/theme-chalk/index.css'
  import storage from '@/utils/storage'
  import Pagination from '@oj/components/Pagination'
  export default {
    name: 'home',
    components: {
      Pagination
    },
    filters: {
      ellipsis (value) {
        if (!value) return ''
        if (value.length > 10) {
          return value.slice(0, 10) + '...'
        }
        return value
      }
    },
    data () {
      return {
        images: [],
        contests_raw: [],
        contests: [],
        index: 0,
        CONTEST_STATUS_REVERSE: CONTEST_STATUS_REVERSE,
        limit: 10,
        total: 10,
        btnLoading: false,
        announcements: [],
        announcement: '',
        listVisible: true,
        AnnouncementStyle: {'width': '75%', 'float': 'left'},
        timer: null,
        SighinStatus: false,
        nowYear: '',
        nowWeek: '',
        nowDate: '',
        nowMouth: '',
        word: '',
        days: 0,
        wordcd: false,
        Omi: 'Omi-Kuji',
        totalTime: 5
      }
    },
    mounted () {
      this.init()
      let params = { status: CONTEST_STATUS.NOT_START }
      api.getContestList(0, 5, params).then((res) => {
        this.contests = this.contests.concat(res.data.data.results)
      })
      params = { status: CONTEST_STATUS.UNDERWAY }
      api.getContestList(0, 5, params).then((res) => {
        this.contests = this.contests.concat(res.data.data.results)
      })
      // this.getExtraImage('/public/upload/img2.png')
      // this.getExtraImage('/public/upload/img3.png')
      this.timer = setInterval(() => {
        this.setNowTimes()
      }, 10)
      this.word = 'Try your luck :)'
    },
  
    methods: {
      init () {
        if (this.isContest) {
          this.getContestAnnouncementList()
        } else {
          this.getAnnouncementList()
        }
      },
      getAnnouncementList (page = 1) {
        this.btnLoading = true
        api.getAnnouncementList((page - 1) * this.limit, this.limit).then(res => {
          this.btnLoading = false
          this.announcements = res.data.data.results
          this.total = res.data.data.total
        }, () => {
          this.btnLoading = false
        })
      },
      getContestAnnouncementList () {
        this.btnLoading = true
        api.getContestAnnouncementList(this.$route.params.contestID).then(res => {
          this.btnLoading = false
          this.announcements = res.data.data
        }, () => {
          this.btnLoading = false
        })
      },
      goAnnouncement (announcement) {
        this.announcement = announcement
        this.listVisible = false
      },
      goBack () {
        this.listVisible = true
        this.announcement = ''
      },
      getDuration (startTime, endTime) {
        return time.duration(startTime, endTime)
      },
      goContest (index) {
        this.index = index
        this.$router.push({
          name: 'contest-details',
          params: { contestID: this.contests[this.index].id }
        })
      },
      async getExtraImage (url) {
        await axios
        .get(url)
        .then(response => {
          this.images.push(url)
        })
        .catch(error => {
          console.log(error)
        })
      },
      getWord () {
        axios.get('https://v1.hitokoto.cn/?c=d&c=e&c=f&c=h&c=i&c=j&c=k').then(response => {
          this.word = response.data.hitokoto
        })
      },
      setNowTimes () {
        let myDate = new Date()
        let weeks = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
        let mouth = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
        this.nowDate = String(myDate.getDate() < 10 ? '0' + myDate.getDate() : myDate.getDate())
        this.nowMouth = mouth[myDate.getMonth()]
        this.nowWeek = weeks[myDate.getDay()]
        this.nowYear = myDate.getFullYear()
      },
      Sighin () {
        if (storage.get(STORAGE_KEY.AUTHED)) {
          if (this.wordcd) return
          this.$success('Good luck!')
          this.getWord()
          this.$set(this, this.word, this.word)
          this.wordcd = true
          this.Omi = 'Cooling...' + this.totalTime + 's'
          let clock = setInterval(() => {
            this.totalTime--
            this.Omi = 'Cooling...' + this.totalTime + 's'
            if (this.totalTime < 0) {
              clearInterval(clock)
              this.Omi = 'Omi-Kuji'
              this.totalTime = 5
              this.wordcd = false
            }
          }, 1000)
        } else {
          this.$error('Please login first')
        }
      }
    },
    computed: {
      title () {
        if (this.listVisible) {
          return this.isContest ? this.$i18n.t('m.Contest_Announcements') : this.$i18n.t('m.Announcements')
        } else {
          return this.announcement.title
        }
      },
      isContest () {
        return !!this.$route.params.contestID
      }
    }
  }
</script>

<style lang="less" scoped>
    .announcements-container {
    margin-top: -10px;
    margin-bottom: 10px;
    li {
      padding-top: 15px;
      list-style: none;
      padding-bottom: 15px;
      margin-left: 20px;
      font-size: 16px;
      border-bottom: 1px solid rgba(187, 187, 187, 0.5);
      &:last-child {
        border-bottom: none;
      }
      .flex-container {
        .title {
          flex: 1 1;
          text-align: left;
          padding-left: 10px;
          a.entry {
            color: #495060;
            &:hover {
              color: #2d8cf0;
              border-bottom: 1px solid #2d8cf0;
            }
          }
        }
        .creator {
          flex: none;
          width: 200px;
          text-align: center;
        }
        .date {
          flex: none;
          width: 200px;
          text-align: center;
        }
      }
    }
  }
  .content-container {
    padding: 0 20px 20px 20px;
  }
  .no-announcement {
    text-align: center;
    font-size: 16px;
  }changeLocale
  .announcement-animate-enter-active {
    animation: fadeIn 1s;
  }
  .button{
    margin-top:20px; margin-bottom:20px; margin-left:10%; width:80%;
  }
  .disabled{
    background-color: #ddd;
    border-color: #ddd;
    color:#57a3f3;
    cursor: not-allowed; // 鼠标变化
  }
  .contest {
    &-title {
      overflow: hidden;
      font-size: 21px
    }
    &-content {
      padding: 0 50px 40px 50px;
      &-description {
        height: 250px;
      }
    }
  }
  .announcement {
    margin-top: 20px;
  }
  .el-carousel{
    width: 90%;
  }
  .el-carousel__item h3 {
    color: #475669;
    font-size: 14px;
    opacity: 0.75;
    line-height: 200px;
    margin: 20px;
  }
  
  .el-carousel__item:nth-child(2n) {
    background-color: #d3dce6; 
    border-radius: 25px;
  }
  
  .el-carousel__item:nth-child(2n+1) {
    background-color: #ffffff;
    border-radius: 25px;
  }
  .contest-content-description{
    margin-top: 120px;
    height: 200px;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
  }
  // img{
  //   width: 100%;
  //   height: 100%;
  // }
  
  .btn-group{
    bottom: 30px;
  }
  .home-pick-one{
    background: white;
    height: 100px;
    margin-top: 20px;
    border-radius: 25px;
  }
  .changeVis {
    display: true;
  }
  .changeVis :after{
    display: none;
  }
</style>