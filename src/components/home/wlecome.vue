<template>
  <div>
    <!-- 首页主题部分 -->
    <el-card class="new-col">
      <el-row :gutter="20" class="header">
        <el-col :span="6">
          <div class="container-card">
            <div class="img-contain">
              <img src="@/assets/image/wlecome/play.svg" alt class="wlecome-img" />
            </div>
            <div class="card-word">
              <span class="Detail-Title" style="  color: rgba(128, 202, 251, 0.9);">总播放量</span>
              <span class="number" style="color:rgba(119, 127, 232, 0.9)">{{Allplay}}</span>
            </div>
          </div>
        </el-col>
        <el-col :span="6">
          <div class="container-card">
            <div class="img-contain backpro">
              <img src="@/assets/image/wlecome/play.svg" alt class="wlecome-img" />
            </div>
            <div class="card-word">
              <span class="Detail-Title" style="  color: rgba(164, 205, 93, 0.9);">总节目数</span>
              <span class="number" style="color:rgba(98, 180, 94, 0.9)">{{Ptotal}}</span>
            </div>
          </div>
        </el-col>
        <el-col :span="6">
          <div class="container-card">
            <div class="img-contain backup">
              <img src="@/assets/image/wlecome/update.svg" alt class="wlecome-img" />
            </div>
            <div class="card-word">
              <span class="Detail-Title" style="  color: rgba(230, 132, 98, 0.9);">今日更新</span>
              <span class="number">{{dayup.length}}</span>
            </div>
          </div>
        </el-col>
        <el-col :span="6">
          <div class="container-card cursor" @click="iscomment">
            <div class="img-contain backps">
              <img src="@/assets/image/wlecome/push.svg" alt class="wlecome-img" />
            </div>
            <span style=" color: rgba(229, 200, 90, 0.9)">留言板</span>
          </div>
        </el-col>
      </el-row>
      <h4>最新节目</h4>
      <!-- 首页轮播图组件 -->
      <piclist :piclist="picimg" />
      <!-- 分类组件 -->
      <modlist :data="animation" :title="'动漫'" />
      <modlist :data="movie" :title="'电影'" />
      <modlist :data="tvseries" :title="'电视剧'" />
      <modlist :data="variety" :title="'综艺'" />
    </el-card>
    <el-dialog :title="status?'留言内容':'留言板'" :visible.sync="comment" width="50%">
      <div class v-if="status">
        <el-timeline>
          <el-timeline-item
            :timestamp="item.createtime | dateformat()"
            placement="top"
            v-for="item in commentlist"
            :key="item._id"
          >
            <el-card>
              <h4 class="commentcon">{{item.content}}</h4>
              <p class="comment-p">{{item.username}} 提交于 {{item.createtime | dateformat()}}</p>
            </el-card>
          </el-timeline-item>
        </el-timeline>
      </div>
      <div v-else>
        <el-form
          :model="comments"
          ref="comment"
          label-width="80px"
          class="demo-ruleForm"
          size="small"
        >
          <el-form-item label="留言内容">
            <el-input
              v-model="comments.content"
              type="textarea"
              placeholder="感谢你的建议！"
              class="content"
            ></el-input>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer" v-if="!status">
        <el-button @click="comment = false" size="small">取 消</el-button>
        <el-button type="primary" @click="submitcontent" size="small">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import piclist from "./piclist"
import { mapState, mapActions } from "vuex"
import modlist from "./wlecome/modlist"
export default {
  components: {
    piclist,
    modlist
  },
  data() {
    return {
      picimg: [],
      // 总节目数量
      total: 0,
      Ptotal: 0,
      timer: null,
      playtimer: null,
      // 总播放数
      play: 0,
      Allplay: 0,
      // 每日更新的节目
      dayup: [],
      // 控制留言板的显示
      comment: false,
      comments: {
        avatar: "",
        username: "",
        content: ""
      },
      commentlist: null,
      allprogram: [],
      Videos: null,
      // 分类数据
      movie: null,
      tvseries: null,
      variety: null,
      animation: null
    }
  },
  created() {
    this.getallprogram()
    this.getcomment()

    // this.content.username = this.userinfo.username
  },
  methods: {
    ...mapActions(["saveallpro"]),
    async getallprogram() {
      const { data: res } = await this.$http.get("/home/program")
      // 保存所有节目
      console.log(res)
      res.data.forEach(async item => {
        this.allprogram.push(...item.data)
      })
      // 对节目进行排序（按照热度）
      this.allprogram.sort((a, b) => {
        return b.hot - a.hot
      })
      // 总节目数量
      this.total = this.allprogram.length
      // 总播放次数
      res.data.forEach(item => {
        item.data.forEach(item2 => (this.play = this.play + item2.hot))
      })
      // 获取当前时间
      let dt = new Date()
      let nowtime = this.dateformat(dt)
      // 判断是否是今天更新的数据
      res.data.forEach(item => {
        let data = item.data.filter(item2 => {
          console.log(item2)
          let dt = new Date(item2.updatetime)
          let uptime = this.dateformat(item2.updatetime)
          item2.updatetime = dt.valueOf()
          if(nowtime===uptime){
            console.log(item2)
          }
          return nowtime === uptime
        })
        // console.log(data)
        this.dayup.push(...data)
      })
      // 今日更新数

      // 添加首页的数据
      res.data[0].data.forEach(item => {
        item.path = res.data[0].title
        console.log(item)
      })
      let movie = res.data[0].data
        .sort((a, b) => {
          
          return parseInt(b.updatetime) - parseInt(a.updatetime)
        })
        .slice(0, 3)

      res.data[1].data.forEach(item => {
        item.path = res.data[1].title
      })
      let tv = res.data[1].data
        .sort((a, b) => {
          return parseInt(b.updatetime) - parseInt(a.updatetime)
        })
        .slice(0, 2)

      res.data[2].data.forEach(item => {
        item.path = res.data[2].title
      })
      let zy = res.data[2].data.reverse().slice(0, 2)

      res.data[3].data.forEach(item => {
        item.path = res.data[3].title
      })
      let dh = res.data[3].data
        .sort((a, b) => {
          return parseInt(b.updatetime) - parseInt(a.updatetime)
        })
        .slice(0, 4)
      // 把节目数据加入到数组picimg
      this.picimg.push(...movie, ...tv, ...zy, ...dh)
      this.movie = res.data[0].data
      this.tvseries = res.data[1].data
      this.variety = res.data[2].data
      this.animation = res.data[3].data
    },
    dateformat(origantime) {
      const dt = new Date(origantime)
      const y = dt.getFullYear()
      const m = (dt.getMonth() + 1 + "").padStart(2, "0")
      const d = (dt.getDate() + "").padStart(2, "0")
      return `${y}-${m}-${d}`
    },
    iscomment() {
      this.comment = true
    },
    // 提交评论
    async submitcontent() {
      this.comments.username = this.username
      this.comments.content.trim()
      const { data: res } = await this.$http.post("/comments", this.comments)
      if (res.code !== 1) return this.$message.error(res.msg)
      this.comment = false
      return this.$message({
        message: res.msg,
        type: "success"
      })
    },
    // 获取评论列表
    async getcomment() {
      const { data: res } = await this.$http.get("/comments")
      if (res.code !== 1) return console.log(res.msg)
      this.commentlist = res.data
    },
    // 获取所有节目的视频链接地址
    async getallVideo() {
      const { data: res } = await this.$http.get("/home/video", {
        params: { type: "all" }
      })
      if (res.code === 200) {
        this.Videos = res.data
      }
    }
  },
  computed: {
    ...mapState({
      status: state => state.status,
      username: state => state.userinfo.username
    })
  },
  watch: {
    // 数字缓动效果
    total(newval) {
      console.log(newval)
      this.timer = setInterval(() => {
        this.Ptotal++
        if (this.Ptotal === newval) {
          clearInterval(this.timer)
        }
      }, 80)
    },
    play(newval) {
      console.log(newval)
      this.playtimer = setInterval(() => {
        this.Allplay+=5
        if (this.Allplay >= newval) {
          this.Allplay = newval
          clearInterval(this.playtimer)
        }
      }, 10)
    }
  }
}
</script>
<style lang="scss" scoped>
.new-col {
  display: flex;
  flex-direction: column;
}
.carousel {
  margin-top: 10px;
}
.carousel img {
  width: 100%;
  height: 100%;
}
.header {
  margin-bottom: 15px;
  font-size: 14px;
}
.container-card {
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
  padding: 15px 0;
}

.cursor {
  cursor: pointer;
}
// 系统数据模块标题
.Detail-Title {
  display: block;
  padding-bottom: 6px;
}
.number {
  font-size: 18px;
  font-weight: 700;
  color: coral;
}
.img-contain {
  width: 55px;
  height: 55px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 100%;
  background: linear-gradient(
    to right,
    rgba(119, 127, 232, 0.9),
    rgba(128, 202, 251, 0.5)
  );
  margin-right: 10px;
  box-sizing: border-box;
}
.backpro {
  background: linear-gradient(
    to right,
    rgba(98, 180, 94, 0.9),
    rgba(164, 205, 93, 0.5)
  );
}
.backup {
  background: linear-gradient(
    to right,
    rgba(241, 95, 100, 0.9),
    rgba(230, 132, 98, 0.5)
  );
}
.backps {
  background: linear-gradient(
    to right,
    rgba(239, 167, 68, 0.9),
    rgba(229, 200, 90, 0.5)
  );
}
.wlecome-img {
  width: 25px;
  height: 25px;
  border-radius: 4px;
}
.dialog-div {
  /deep/ .el-dialog__header {
    padding: 0;
  }

  /deep/ .el-dialog__body {
    padding: 0;
  }

  /deep/ .el-icon-close:before {
    content: "";
  }
}
.content {
  height: 65px;
  .el-textarea__inner {
    height: 65px;
  }
}
.createtime {
  display: flex;
  justify-content: flex-end;
  padding: 10px 0;
  font-size: 14px;
}
.commentcon {
  font-weight: 400;
  color: #1f2f3d;
  padding-bottom: 15px;
}
.comment-p {
  font-size: 14px;
  color: #5e6d82;
  line-height: 1.5em;
}
</style>