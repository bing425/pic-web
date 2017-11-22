<template>
  
  <div>
    <span style="color:#666;font-size:42px;">无限共享图床</span>
    <br><br>
    <el-upload
      ref='fileCom'
      class="upload-demo"
      :on-change="changeFile"
      action=""
      drag>
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <div class="el-upload__tip" slot="tip">只能上传 gif / jpg / png 文件</div>
    </el-upload>
<el-row>
  <el-col :span="8"  style="padding:15px;">
    <el-date-picker
      v-model="selectDate"
      align="right"
      type="date"
      placeholder="选择日期"
      :picker-options="pickerOptions"
      style="float:left">
    </el-date-picker>
    </el-col>
</el-row>

<el-row style="column-count: 4">
  <div v-for="(o, index) in readList" :key="o" style="padding:15px;">
    <el-card :body-style="{ padding: '0px' }">
      <img style="padding-bottom:40px" :src="'https://ipfs.io/ipfs/'+o.hash" class="image" @click="()=>{open('https://ipfs.io/ipfs/'+o.hash)}">
      <div style="padding: 14px;margin-top:-46px;">
        <div class="bottom clearfix">
          <time class="time">{{o.createAt}}</time>
        </div>
      </div>
    </el-card>
  </div>
</el-row>
  <el-row>
    <el-pagination
      @current-change="handleCurrentChange"
      :current-page.sync="currentPage"
      :page-size="10"
      layout="total, prev, pager, next"
      :total="count">
    </el-pagination>
  </el-row>
  </div>
</template>
<script>
const repoPath = 'ipfs-' + Math.random()
let node;
function handleInit (err) {
  if (err) {
  throw err
  }
  node.start(() => {
  console.log('Online status: ', node.isOnline() ? 'online' : 'offline')
  })
}
export default {
  name: 'HelloWorld',
  data () {
    return {
      readList:[],
      currentPage:1,
      count:0,
      pickerOptions: {
        disabledDate(time) {
          return time.getTime() > Date.now();
        },
        shortcuts: [{
          text: '今天',
          onClick(picker) {
            picker.$emit('pick', new Date());
          }
        }, {
          text: '昨天',
          onClick(picker) {
            const date = new Date();
            date.setTime(date.getTime() - 3600 * 1000 * 24);
            picker.$emit('pick', date);
          }
        }, {
          text: '一周前',
          onClick(picker) {
            const date = new Date();
            date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
            picker.$emit('pick', date);
          }
        }]
      },
      selectDate: '',
      fileChange(){
        var _this = this;
        var that = this.$refs['upload-inner'].$refs.input,
        filereader;
        if(!that.files[0]){
          return;
        }
        if(!/(\.gif$)|(\.jpg$)|(\.png$)/g.test(that.files[0].name)){
        this.$message({
          message: '仅能上传gif、jpg、png文件',
          type: 'warning'
        });
          return;
        }
        var upFileName = that.files[0].name;
        var index1 = upFileName.lastIndexOf(".");
        var index2 = upFileName.length;
        var upFileSuffix = upFileName.substring(index1+1,index2);
        if(that.files.length>0){
          filereader = new FileReader();
          filereader.onload = function(e){
            if(node.isOnline()){
              node.files.add(new node.types.Buffer(this.result), (err, res) => {
                if (err || !res) {
                  return console.error('Error - ipfs files add', err, res)
                }
                res.forEach((file) => {
                  var server = Math.floor(Math.random() * 5);
                  var hashName = file.hash + '?' + server + '.' + upFileSuffix;
                  var picurl = 'https://ipfs.io/ipfs/' + hashName
                  _this.$http.get('/node/record/'+hashName).then((data)=>{
                      
                  });
                })
              })
            }else{
              alert('\u8fd8\u6ca1\u51c6\u5907\u597d,\u8bf7\u7a0d\u5019\u518d\u8bd5');
            }
          };
          filereader.readAsArrayBuffer(that.files[0]);
        }
      }
    };
  },
  methods:{
    open(url){
      window.open(url)
    },
    handleCurrentChange(page){
      
      this.$http.jsonp('https://24b99220.ngrok.io/node/record',{jsonp:'cb',params:{page:this.currentPage,size:10}}).then((data)=>{
        console.log(data)
        this.readList = data.body.docs;
        this.count = data.body.count;
      })
    },
    changeFile(){
        var _this = this;
        var that = this.$refs.fileCom.$el.querySelector('input'),
        filereader;
        if(!that.files[0]){
          return;
        }
        if(!/(\.gif$)|(\.jpg$)|(\.png$)/g.test(that.files[0].name)){
        this.$message({
          message: '仅能上传gif、jpg、png文件',
          type: 'warning'
        });
          return;
        }
        var upFileName = that.files[0].name;
        var index1 = upFileName.lastIndexOf(".");
        var index2 = upFileName.length;
        var upFileSuffix = upFileName.substring(index1+1,index2);
        if(that.files.length>0){
          filereader = new FileReader();
          filereader.onload = function(e){
            if(node.isOnline()){
              node.files.add(new node.types.Buffer(this.result), (err, res) => {
                if (err || !res) {
                  return console.error('Error - ipfs files add', err, res)
                }
                res.forEach((file) => {
                  var server = Math.floor(Math.random() * 5);
                  var hashName = file.hash + '?' + server + '.' + upFileSuffix;
                  var picurl = 'https://ipfs.io/ipfs/' + hashName
                  _this.$http.jsonp('https://24b99220.ngrok.io/node/record/'+hashName)
                  _this.readList.unshift({
                    createAt:new Date(),
                    hash:hashName
                  })
                })
              })
            }else{
              alert('\u8fd8\u6ca1\u51c6\u5907\u597d,\u8bf7\u7a0d\u5019\u518d\u8bd5');
            }
          };
          filereader.readAsArrayBuffer(that.files[0]);
        }
      }
  },
  mounted(){
    this.$http.jsonp('https://24b99220.ngrok.io/node/record',{jsonp:'cb',params:{page:1,size:10}}).then((data)=>{
      console.log(data)
      this.readList = data.body.docs;
      this.count = data.body.count;
    })
    node = new Ipfs({
      init: false,
      start: false,
      repo: repoPath
    })
    node.init(handleInit)
  //   window.a = this;
  //   let data = {"code":200,"data":{"docs":[{"_id":"5a13fe07c98111977876a606","hash":"QmWNoTkXDcMHiSP2ojZbpjDS8PXCivcaddzSKsX2f5kThm?3.png","creatAt":"2017-11-21T10:20:50.208Z","createAt":"2017-11-21T10:20:55.065Z"}],"count":1}}
  // this.readList = data.data.docs;
  // this.count = data.data.count;
  }
}
</script>

<style scoped>
  .time {
    font-size: 13px;
    color: #999;
  }
  
  .bottom {
    margin-top: 13px;
    line-height: 12px;
  }

  .button {
    padding: 0;
    float: right;
  }

  .image {
    width: 100%;
    display: block;
  }

  .clearfix:before,
  .clearfix:after {
      display: table;
      content: "";
  }
  
  .clearfix:after {
      clear: both
  }
</style>
