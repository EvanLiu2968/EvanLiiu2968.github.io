<template>
  <div style="position:relative;padding-left:250px;">
    <div style="position:absolute;top:0;left:0;width:250px;background:#f6f6f6">
      <h2 class="movie-title">
        电影榜单
      </h2>
      <div class="movie-billboard clearfix">
        <div class="rotateBox" v-for="(list, k) in lists" :key="k" @click="toggleList(list)">
          <div class="transBox">
            <div class="movie-list-box front" :style="{background:list.color}">
              <div class="movie-list-name">
                <span>{{ list.name }}</span>
              </div>
            </div>
            <div class="movie-list-box back" :style="{background:list.color}">
              <div class="movie-list-cover">
                <img :src="list.cover">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div>
      <h1 class="movie-list-title">
        <i class="el-icon-menu"></i>{{ activeList.name }}
      </h1>
      <div class="movie-list">
        <div v-if="subjects.length === 0" class="movie-list-title text-center" style="padding-top:100px">
          豆瓣电影已关闭对外接口，服务不可用 ("▔□▔)
        </div>
        <template v-else>
          <div class="movie-item" v-for="(subject, index) in subjects" :key="index">
            <div class="movie-cover">
              <img :src="subject.images.small">
            </div>
            <div class="movie-content">
              <div class="movie-row">
                {{ subject.title }} ( {{ subject.original_title }} )
                <el-tag class="movie-genre" type="success" v-for="(genre, i) in subject.genres" :key="i">
                  {{ genre }}
                </el-tag>
              </div>
              <div class="movie-row">
                年份：<span class="movie-tag">{{ subject.year }}</span>导演：
                <a class="movie-tag" :href="director.alt" v-for="(director, i) in subject.directors" :key="i" target="_blank">{{ director.name }}</a>
              </div>
              <div class="movie-row">
                主演：
                <a class="movie-tag" :href="cast.alt" v-for="(cast, i) in subject.casts" :key="i" target="_blank">{{ cast.name }}</a>
              </div>
              <div class="movie-row">
                评分：<span style="color:#F7BA2A">{{ subject.rating.average }}</span>
                <span class="movie-tag">/{{ subject.rating.max }}</span>
                <router-link class="movie-tag" :to="{ name: 'movie-detail', params: { id: subject.id }}">
                  查看详情 <i class="el-icon-d-arrow-right text-small"></i>
                </router-link>
              </div>
            </div>
          </div>
          <el-pagination
            @current-change="handleCurrentChange"
            :current-page="pagination.currentPage"
            :page-size="pagination.pageSize"
            layout="prev, pager, next, sizes, jumper"
            :total="pagination.total"
          >
          </el-pagination>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import { getDoubanMovieList } from '@/api'
import './index.less'

export default {
  data: function() {
    return {
      lists: [
        {
          name: 'TOP250',
          api: 'top250',
          route: 'top250',
          cover: 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p480747492.webp',
          color: '#20A0FF'
        }
        // {
        //   name: '正在热映',
        //   route: 'theaters',
        //   api: 'in_theaters',
        //   cover: '',
        //   color: '#13CE66'
        // },
        // {
        //   name: '即将上映',
        //   api: 'coming_soon',
        //   route: 'coming',
        //   cover: '',
        //   color: '#F7BA2A'
        // }
      ],
      activeList: {},
      subjects: [],
      pagination: {
        currentPage: 1,
        pageSize: 10,
        total: 0
      }
    }
  },
  beforeMount: function() {
    let listType = this.$route.query.list
    listType = listType || 'top250'
    let index = this.lists.findIndex(item => {
      return item.route == listType
    })
    index = index < 0 ? 0 : index
    this.toggleList(this.lists[index])
  },
  mounted: function() {
    this.lists.forEach((list, index) => {
      getDoubanMovieList(list.api, 0, 1).then(res => {
        console.log(res)
        this.lists[index].cover = res.subjects[0].images.large
      }).catch(e => {
        console.log(e)
      })
    })
  },
  methods: {
    toggleList(list) {
      this.$router.push({ name: 'movie', query: { list: list.route }})
      this.activeList = list
      this.paginate()
    },
    paginate() {
      const count = this.pagination.pageSize
      const start = (this.pagination.currentPage - 1) * count
      const api = this.activeList.api
      getDoubanMovieList(api, start, count).then(res => {
        this.pagination.total = res.total
        this.subjects = res.subjects
      })
    },
    handleCurrentChange(current) {
      this.pagination.currentPage = current
      this.paginate()
    }
  }
}
</script>
<style scoped>
.movie-title{
  text-align:center;
  font-size:18px;
  color:#58B7FF;
  margin-top:20px;
}
.movie-billboard{
  margin-bottom:15px;
  padding:25px;
}
.movie-list-cover{
  position:absolute;
  padding:0;
  display:block;
  top:0;left:0;bottom:0;right:0;
}
.movie-list-cover>img{
  display:block;
  width:200px;
  height:300px;
}
.movie-list-name{
  line-height:300px;
  font-size:30px;
  font-weight:bolder;
  color:#fff;
  text-align: center;
}
/*动画*/
.rotateBox{
  display: block;
  overflow:hidden; height: 300px; width: 200px; float: left; position: relative;
  -webkit-perspective: 2000px; perspective:600px;-moz-perspective: 600px;-o-perspective: 600px; -ms-perspective: 600px;
}
.transBox{
  -webkit-transform-style: preserve-3d; -moz-transform-style: preserve-3d;-o-transform-style: preserve-3d;
  position:absolute; top: 0; left: 0;height: 300px; width: 200px; transform-origin: center center -100px;
  -webkit-transition:.4s;-moz-transition:.4s;-o-transition:.5s;
}
.transBox .movie-list-box{
  position: absolute; height: 500px;
  cursor: pointer;
}
.transBox .front{
  left: 0; top:0;
}
.transBox .back{
  left: 200px; top:0;
  -webkit-transform-origin: left; -moz-transform-origin: left;-o-transform-origin: left;-ms-transform-origin: left;
  -webkit-transform: rotateY(90deg);-moz-transform: rotateY(90deg);-o-transform: rotateY(90deg);-ms-transform: rotateY(90deg);
}
.transBox .backed{
  left:200px; top:0; position: absolute
}
.rotateBox:hover .transBox{
  -webkit-transform: rotateY(-90deg);-moz-transform: rotateY(-90deg);-o-transform: rotateY(-90deg);
}
.movie-list-box{
  position:relative;height: 300px; width: 200px;float:left; display:block;overflow:hidden;
}
</style>
