<template>
  <div id="app">
    <h1>
      <img src="./assets/icon_128.png" />
      Water Drop
    </h1>
    <div class="toolbar">
      <el-select v-model="version" placeholder="请选择" @change="change">
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      <el-button type="primary" @click="weekly">Export Weekly</el-button>
    </div>
    <div v-for="cate in categories" :key="cate.key">
      <h2 :id="cate.key">{{cate.label}}</h2>
      <ul>
        <li v-for="post in list.filter(t => t.category === cate.key)" :key="post.objectId">
          <a :href="post.url" target="_blank">
            {{post.title}}
          </a>
          <span>{{post.createdAt.slice(0, 10)}}</span>
          <blockquote>{{post.description}}</blockquote>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import AV from 'leancloud-storage'

const appId = '883Ml4hpEll5MVigAMlv5W3v-gzGzoHsz'
const appKey = '1wXte4OPEo8xazj2GIoYkjSI'

AV.init({ appId, appKey })

const categories = [
  {
    key: 'language',
    label: '要闻'
  },
  {
    key: 'tool',
    label: '开源'
  },
  {
    key: 'post',
    label: '文章'
  },
  {
    key: 'study',
    label: '教程'
  },
  {
    key: 'visual',
    label: '可视化'
  },
  {
    key: 'design',
    label: '设计'
  },
  {
    key: 'commuity',
    label: '社区'
  },
  {
    key: 'resource',
    label: '资源'
  },
  {
    key: 'interview',
    label: '面试'
  },
  {
    key: 'other',
    label: '其他'
  }
]

const _start = '2017-07-28'
const _ver = 1015

export default {
  name: 'list',
  data () {
    return {
      categories: categories,
      list: [],
      version: '',
      options: []
    }
  },
  created () {
    const vers = this.genVers()

    this.options = vers.map(t => ({
      value: `${t.start}~${t.end}`,
      label: `第${t.version}期（${t.start} ~ ${t.end}）`
    }))
    this.version = this.options[vers.length - 1].value

    this.load(this.version)
  },
  methods: {
    change () {
      this.load(this.version)
    },

    genVers () {
      const days = moment().diff(moment(_start), 'days')
      let sum = Math.ceil(days / 7)

      if(moment().day() === 5 && moment().hour() >= 12) {
        sum += 1
      }

      const opts = new Array(sum).fill('').map((t, idx) => ({
        version: _ver + idx,
        start: moment(_start).add(7 * idx, 'days').format('YYYY-MM-DD'),
        end: moment(_start).add(7 * (idx + 1), 'days').format('YYYY-MM-DD')
      }))

      return opts
    },

    load (version) {
      const loading = this.$loading({
        fullscreen: true
      })

      let [start, end] = version.split('~')

      start = moment(start).hour(12)
      end = moment(end).hour(12)

      this.query(start, end).then((res) => {
        loading.close()

        this.$message({
          message: '数据加载成功',
          type: 'success'
        })

        this.list = res.map(t => t.toJSON())
      }, (e) => {
        loading.close()

        this.$message.error(e.toString())
      })
    },

    query (start, end) {
      const startQuery = new AV.Query('Waters')
      startQuery.greaterThan('createdAt', new Date(start))

      const endQuery = new AV.Query('Waters')
      endQuery.lessThan('createdAt', new Date(end))

      const query = AV.Query.and(startQuery, endQuery)

      return query.find()
    },

    weekly () {
      let md = ''

      categories.forEach(c => {
        let str = `## **${c.label}**\n\n`

        this.list.filter(t => t.category === c.key).forEach(p => {
          str += `* [${p.title}](${p.url})\n\n> ${p.description}\n\n`
        })

        md += str
      })

      const blob = new Blob([md])
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.download = `${this.version}.md`
      a.href = url
      a.click()
      URL.revokeObjectURL(url)
    }
  }
}
</script>

<style>
#app {
  font-family: "SF Pro SC","SF Pro Text","SF Pro Icons","PingFang SC","Helvetica Neue","Helvetica","Arial",sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-size: 14px;
  color: #1f2d3d;
  width: 1000px;
  margin: 50px auto;
  position: relative;
}

h1 {
  height: 50px;
  margin-bottom: 50px;
}

img {
  height: 50px;
  vertical-align: bottom;
}

.toolbar {
  position: absolute;
  top: 10px;
  right: 10px;
}

.el-select {
  width: 300px;
  margin-right: 10px;
}

a {
  display: inline-block;
  color: #20a0ff;
  margin-right: 20px;
  line-height: 30px;
}

a:hover {
  color: #4db3ff;
}

blockquote {
  padding-left: 20px;
  margin: 10px;
  border-left: 4px solid #e2e3e4;
  line-height: 30px;
}
</style>
