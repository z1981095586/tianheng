<template>
  <div class="content">

    <span style="font-size: 1rem;font-weight: bold;margin-bottom: -1rem">保养物料</span>
    <i class="el-icon-arrow-left" style="position:fixed;left:17px;top:13px;" @click="back"></i>
    <div class="select">



      <select name="bbxb" id="selecte" class="shortselect" @change="change">
        <option :value="item.categories_id" v-for="item in menu">{{item.label}}</option>

      </select>


    </div>
    <!--出库单详细-->
    <div class="all-page">
      <div class="input"><input @keyup.13="seachinfo" type="search" ref="input1" placeholder="输入物料编号，名称，规格等"
          v-model="searchinfo" /> <i class="el-icon-search" style="font-size:15px;"></i>
      </div>
      <scroller height="100%" :onRefresh="refresh" :onInfinite="inf" ref="my_scroller">
        <div style="height:80%;overflow:auto;">
          <div v-for="item in datalist">
            <div class="contain" style="margin-top:10px">
              <div class="card">
                <div class="check">
                  <el-checkbox v-model="item.checked"></el-checkbox>
                </div>
                <div class="card-content">
                  <div class="content-one">
                    <span>备品名称:{{item.name}}</span>
                    <span>备品规格:{{item.specification}}</span>
                  </div>

                </div>
                <div class="card-content">
                  <div class="content-one">
                    <span>库存数量:{{item.stockQuantity}}</span>

                  </div>

                </div>
              </div>

            </div>
            <div class="contain">
              <div class="add"><span>数量：</span>
                <div class="input-number"><i class="el-icon-remove" @click="sub(item.id)"
                    style="color:rgb(255,153,102)"></i><span @click="Inputshow(item.id)" v-show="!item.isinputShow">{{item.stockQuantityShow}}</span><el-input size="small" @change="((val)=>{changeStatus(val, item.id)})" @blur="Inputshow(item.id)" v-model="item.stockQuantityShow" v-show="item.isinputShow"></el-input> <i class="el-icon-circle-plus"
                    @click="add(item.id)" style="color:rgb(49,153,102)"></i> </div>
              </div>
            </div>
          </div>
        </div>
      </scroller>
      <div class="bottom">
        <div class="submit" @click="submit"><span>提交</span></div>
        <div class="cancel" @click="cancel"><span>取消</span></div>
      </div>
    </div>

  </div>

</template>

<script>
  import axios from 'axios'
  import VueScroller from "vue-scroller" //引入vue-scroller，用来做上拉加载下拉刷新
  export default {
    name: 'Maintenance_materials',
    data() {
      return {
 operator:"",
 maintain_type_id:null,
 isClickIn:"",
        machine_id: null, //设备id
        company_id: '', //公司id
        mac_type_id: null, //设备类型id
        flag: null, //
        type_name: null, //类型名称
        pageNum: 1, //当前页面
        totalDataNum: 0, //列表总记录条数
        datalist: [], //存放列表数据
        menu: [], //存放下拉菜单选项
        categories_id: "", //
        searchinfo: "", //搜索框信息
    workshop_id:"",
    machine_id:"",
    datalist2:[]
      }
    },
    methods: {
      seachinfo() { //搜索事件
        this.$refs.input1.blur();
        this.pageNum = 1
        this.datalist = []

        //console.log(this.searchinfo)
        if (this.searchinfo != "") { //如果搜索条件没有就显示第一个下拉菜单的数据
          this.getInventory(this.searchinfo)
        } else {
          this.getInventory()
        }

      },
      change(e) { //下拉菜单值发生变化监听事件
        //console.log(e.target.value)
        this.categories_id = e.target.value
        this.datalist = []
        this.pageNum = 1
        this.getInventory(this.searchinfo) //如果搜索框有数据，依旧会搜索当前选择的下拉值下的搜索数据
      },
      getInventory(searchinfo) { //获取保养物料数据

        if (this.menu.length == 0) { //如果下拉菜单没数据去获取分类根目录

          this.getRootCategories()
        }

        //  this.$refs.myscroller.finishPullToRefresh();
        let that = this
        let url = "http://47.110.242.174:16021/api/warehouse/getInventory" //获取库存数量
        var datas; //存放json数据
        setTimeout(() => { //过300毫秒执行，防止categories_id没有查到就执行报错
          //console.log(that.menu)
          if (that.menu.length > 0) { //如果下拉菜单有数据
            if (searchinfo) { //如果搜素输入框有东西
              datas = { //查询当前选择的下拉选项下的搜索数据


                page: that.pageNum,
                pageNum: 10,
                startDate: "1999-01-01 00:00:00",
                endDate: "2999-01-01 00:00:00",
                isFilterZero: "1",
                isDisable: "0",
                productTypeId: this.categories_id,
                queryConditions: searchinfo
              }
            } else { //查询当前选择的下拉选项下的数据
              datas = {


                page: that.pageNum,
                pageNum: 10,
                startDate: "1999-01-01 00:00:00",
                endDate: "2999-01-01 00:00:00",
                isFilterZero: "1",
                isDisable: "0",
                productTypeId: this.categories_id
              }
            }

          } else { //如果下拉菜单没有获取到，那就获取全部数据
            if (searchinfo) {
              let datas = {


                page: that.pageNum,
                pageNum: 10,
                startDate: "1999-01-01 00:00:00",
                endDate: "2999-01-01 00:00:00",
                isFilterZero: "1",
                isDisable: "0",
                productTypeId: this.categories_id,
                queryConditions: searchinfo
              }
            } else {
              datas = {


                page: that.pageNum,
                pageNum: 10,
                startDate: "1999-01-01 00:00:00",
                endDate: "2999-01-01 00:00:00",
                isFilterZero: "1",
                isDisable: "0",
                // queryConditions:""
              }

            }
          }
          axios.post(url, datas, { //开始查询
            headers: {
              'Content-Type': 'application/json',
              "companyId": that.company_id
            }
          }).then(function (res) {
            //console.log(res)
            if (res.data.data.product.length == 0) { //如果查到没数据了，那就关闭上拉加载了
              that.$message({
                message: '没有更多数据了！',
                center: true,
                duration: 1000
              });
              that.$refs.my_scroller.finishInfinite(true)
              return
            } else {
              that.totalDataNum = res.data.data.totalDataNum //设置数据总条数
              that.$refs.my_scroller.finishPullToRefresh(true) ////下拉获取数据回调函数停止使用

              //  that.noDate=that.iscurrent(that.totalDataNum,that.pageNum)

              for (let i = 0; i < res.data.data.product.length; i++) { //push消耗物料数据
               res.data.data.product[i].stockQuantitydata= res.data.data.product[i].stockQuantity
             res.data.data.product[i].isinputShow=false
               res.data.data.product[i].stockQuantityShow=0
                that.datalist.push(res.data.data.product[i])
              }
              that.$refs.my_scroller.finishInfinite(true) //上拉获取数据回调函数停止使用
            }




          })

        }, 300);

      },
      iscurrent(totalDataNum, pageNum) { //判断当前页数是否最后一条
        if (totalDataNum == pageNum) {
          return true
        } else {
          return false
        }
      },


      getParams() { //获取其他页面传过来的参数

      console.log(this.$route.params)
        this.mac_type_id = this.$route.params.dataObj2.mac_type_id
        this.flag = this.$route.params.flag
        this.type_name = this.$route.params.dataObj2.type_name
        this.machine_id = this.$route.params.machine_id
         this.operator=this.$route.params.operator
          this.workshop_id=this.$route.params.workshop_id
           this.company_id=this.$route.params.company_id
          this.machine_id=this.$route.params.machine_id
          this.datalist2=this.$route.params.datalist
          this.maintain_type_id=this.$route.params.maintain_type_id
          this.isClickIn=this.$route.params.isClickIn
          console.log(this.datalist2)
      },
      back() { //点取消返回上一个页面
        
        this.$router.push({
          path: "/Maintenance",
          name: "Maintenance",
          params: {
              dataObj2:{
                type_name: this.type_name,
              mac_type_id: this.mac_type_id,
              flag: this.flag,
   },
              operator:this.operator,
               workshop_id:this.workshop_id,
            company_id:this.company_id,
            machine_id:this.machine_id,
            datalist:this.datalist2,
            maintain_type_id:this.maintain_type_id,
            isClickIn:this.isClickIn
          }

        })
      },
      refresh() { //下拉刷新函数
        //console.log("refresh");
        this.datalist = []
        this.pageNum = 1




        setTimeout(() => { //不设置定时器会出现bug
          this.getInventory() //获取数据列表

          this.$refs.my_scroller.finishPullToRefresh() //关闭下拉刷新函数
        }, 500);

      },

      /**
       * 上拉获取
       */
      inf() {


        setTimeout(() => {

          if (this.datalist.length == this.totalDataNum) { //如果数据总条数等于当前数据列表长度了，那就关闭上拉加载了
            this.$refs.my_scroller.finishInfinite(true)
            return
          } else if (this.datalist.length == 0) { //如果查到没数据了，那就关闭上拉加载了
            this.$refs.my_scroller.finishInfinite(true)
            return
          } else { //否则就++，获取下一页数据
            this.pageNum++;
            this.getInventory(this.searchinfo)
          }

        }, 500);

      },

      getRootCategories() { //获取分类根目录
        let url = "http://47.110.242.174:8206/api/product/getRootCategories"
        let that = this
        axios.post(url, {



        }, {
          headers: {
            'Content-Type': 'application/json',
            "companyId": that.company_id
          }
        }).then(function (res) {
          //console.log(res)
          if (res.data.data.categories_id) { //如果存在id，就去获取子分类了
            that.getSubCategoriesList(res.data.data.categories_id)
          } else {

            

            that.$message({
              message: '没有获取到根分类！',
              center: true,
              duration: 1000
            });
          }

        })
      },
      getSubCategoriesList(id) { //获取子分类
        let that = this
        let url2 = "http://47.110.242.174:8206/api/product/getSubCategoriesList"

        axios.post(url2, {
          categories_id: id


        }, {
          headers: {
            'Content-Type': 'application/json',
            "companyId": that.company_id
          }
        }).then(function (res) {
          //console.log(res)
          if (res.data.data.categoriesModel.length > 0) {

            for (let i = 0; i < res.data.data.categoriesModel.length; i++) {
              res.data.data.categoriesModel[i].label = res.data.data.categoriesModel[i].name
              res.data.data.categoriesModel[i].value = res.data.data.categoriesModel[i].name
            }
            that.menu = res.data.data.categoriesModel //设置好下拉菜单选项

            //console.log(that.menu)
            that.categories_id = that.menu[0].categories_id //存储第一个默认选项的id
          } else {

            that.$message({
              message: '没有获取到子分类！',
              center: true,
              duration: 1000
            });
          }


        })
      },

      sub(id) { //点减号将当前记录的库存数量-1
        //console.log(id)

        for (let i = 0; i < this.datalist.length; i++) {
          if (this.datalist[i].id == id) {
            if (this.datalist[i].stockQuantityShow > 0) {
                 this.datalist[i].stockQuantity = this.datalist[i].stockQuantity +1
                        this.datalist[i].stockQuantityShow=this.datalist[i].stockQuantityShow -1
            } else {

              this.$message({
                message: '数量最小为0！',
                center: true,
                duration: 1000
              });
            }
          }
        }
      },
      add(id) { //点加号将当前记录的库存数量+1
        for (let i = 0; i < this.datalist.length; i++) {
          if (this.datalist[i].id == id) {
 if (this.datalist[i].stockQuantity<=0) {
      this.$message({
                message: '数量不能大于库存数量！',
                center: true,
                duration: 1000
              });
 }else{
 this.datalist[i].stockQuantity = this.datalist[i].stockQuantity -1
            this.datalist[i].stockQuantityShow=this.datalist[i].stockQuantityShow +1
 }
           

          }
        }
      },
      changeStatus(val,id){
           for (let i = 0; i < this.datalist.length; i++) {
          if (this.datalist[i].id == id) {
 if (val > this.datalist[i].stockQuantitydata) {
    this.$message({
                message: '数量不能大于库存数量！',
                center: true,
                duration: 1000
              });
          this.datalist[i].stockQuantityShow=0
 }else{
this.datalist[i].stockQuantity = this.datalist[i].stockQuantitydata -val
            this.datalist[i].stockQuantityShow=val
 }
            

          }
        }
      },
      Inputshow(id){
       for (let i = 0; i < this.datalist.length; i++) {
          if (this.datalist[i].id == id) {

            this.datalist[i].isinputShow = !this.datalist[i].isinputShow

          }
        }
      },
      submit() { //提交跳转页面，将选择的数据发送到上一个页面
        //console.log(this.datalist)
        let list = []
        for (let i = 0; i < this.datalist.length; i++) {
          if (this.datalist[i].checked == true) {
            
            list.push(this.datalist[i])
          }
        }
        for(let i=0;i<list.length;i++){
          list[i].stockQuantity=list[i].stockQuantityShow
        }
        for(let i=0;i<this.datalist2.length;i++){
      if (typeof (this.datalist2[i].filelist) != "undefined"){
          if(this.datalist2[i].filelist.length>0){
          for(let j=0;j<this.datalist2[i].filelist.length;j++){
            this.datalist2[i].filelist[j]={
              url:this.datalist2[i].filelist[j],
              name:j+'.jpg'
            }

          }
          }
           }
        }

        //console.log(list)
        this.$router.push({
          path: '/Maintenance',
          name: 'Maintenance',
          params: {

            dataObj: list,
                 operator:this.operator,
               workshop_id:this.workshop_id,
            company_id:this.company_id,
 
              type_name: this.type_name,
              mac_type_id: this.mac_type_id,
              flag: this.flag,
              machine_id: this.machine_id,
                   datalist:this.datalist2
            
          }

        })
      },
      cancel() { //点取消返回上一个页面，只发送设备信息数据
            for(let i=0;i<this.datalist2.length;i++){
            console.log(this.datalist2[i].filelist)

                if (typeof (this.datalist2[i].filelist) != "undefined"){
          if(this.datalist2[i].filelist.length>0){
          for(let j=0;j<this.datalist2[i].filelist.length;j++){
            this.datalist2[i].filelist[j]={
              url:this.datalist2[i].filelist[j],
              name:j+'.jpg'
            }

          }
          }
          }
        }
        this.$router.push({
          path: '/Maintenance',
          name: 'Maintenance',
          params: {


    
   dataObj2:{
                type_name: this.type_name,
              mac_type_id: this.mac_type_id,
              flag: this.flag,
   },
      datalist:this.datalist2,
            machine_id:this.machine_id,
             operator:this.operator,
               workshop_id:this.workshop_id,
            company_id:this.company_id
          }
        })
      }
    },
    mounted() {
      this.datalist = [] //初始化数据
      this.pageNum = 1
    
      this.getParams()//获取其他页面传的数据

      
history.pushState(null, null, window.location.href);
        window.addEventListener('popstate', function () {
            history.pushState(null, null, window.location.href);
        });

      this.getInventory()//获取保养物料数据




    },
    watch: {
      searchinfo: function (val, oldVal) { //监听搜索框数据，若清空，则获取下拉选择下的默认数据
        if (val == "") {
          this.pageNum = 1
          this.datalist = []
          this.getInventory()
        }
      }
    }

  }

</script>

<style scoped>
  span {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

  .shortselect {
    border: 0;
    display: block;
    position: relative;
    min-height: 1.146667rem;
    line-height: 1.146667rem;
    white-space: nowrap;
    position: fixed;
    right: 15px;
    top: 12px;
    font-size: 12px;
    color: rgb(21, 153, 204);
    overflow: hidden;

    background-color: #eee;
    background: transparent;
    appearance: none;
    -moz-appearance: none;
    /* Firefox */
    -webkit-appearance: none;
    /* Safari 和 Chrome */

  }

 .all-page /deep/ ._v-container {
    position: absolute;
    top: 45px;
    left: 0;

  }

  .select span {


    margin-top: 0px;

    font-size: 12px;
    outline: none;


    color: rgb(21, 153, 204);


    border-radius: 4px;


    width: 80px;
    position: fixed;
    top: 10px;
    right: 0px;

  }

  .text {


    height: 20px;


    -webkit-appearance: none;


    appearance: none;


    border: none;


    font-size: 12px;


    padding: 0px 10px;


    display: block;


    width: 100%;


    -webkit-box-sizing: border-box;


    box-sizing: border-box;


    background-color: #FFFFFF;


    color: rgb(65, 152, 199);


    border-radius: 4px;
    

  }



  .el-dropdown {
    color: #606266;
    font-size: 17px;
    /* float: right; */
    /* position: absolute; */
    width: 100px;
    position: fixed;
    right: 0px;
    top: 10px;
  }

  .bottom {
    width: 100%;
    display: flex;
    justify-content: space-around;
    align-items: center;
    position: fixed;
    bottom: 0;
  }

  .content {
    position: relative;
    overflow: auto;
    padding-top: 10px;
  }
 



  .submit {
    width: 49%;
    height: 40px;
    text-align: center;
    display: flex;
    justify-content: center;
    align-items: center;
    background: rgb(49, 153, 102);
    color: white;
  }

  .cancel {
    width: 49%;
    height: 40px;
    text-align: center;
    display: flex;
    justify-content: center;
    align-items: center;
    background: rgb(255, 153, 102);
    color: white;
  }

  .add {
    width: 90%;
    display: flex;
    justify-content: flex-start;
    margin-top: 10px;
  }

  .input-number {
    width: 29%;
    display: flex;
    justify-content: space-around;
    align-items: center;
  }



  .input {
    width: 100%;
    height: 25px;
    background: white;
    margin-top: 5px;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
  }

  .input input {
    width: 70%;
    height: 25px;
    border: none;
  }

  .input input:focus {
    outline: none;

  }

  .input i {
    position: absolute;
    right: 14px;
    color: rgba(0, 0, 0, 0.5);
    font-weight: 700;
  }

  .input input::-webkit-input-placeholder {
    font-size: 0.7rem;
    text-align: center;
    color: rgba(0, 0, 0, 0.6);
  }

  .contain {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .all-page {
    margin-top: 10px;
    width: 100%;

    height: 100%;

    background-color: rgb(242, 242, 242);
    display: flex;
    flex-direction: column;
    position: fixed;
    left: 0;

  }

  .head {
    width: 95%;
    height: 30px;
    display: flex;
    justify-content: space-around;

  }

  .line {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .line-con {
    width: 95%;
    height: 1.5px;
    background: rgb(242, 242, 245);
  }

  .card {
    width: 95%;
    height: 62px;
    display: flex;
    flex-direction: column;
    background: white;
    font-size: 0.9rem;
    position: relative;
    font-weight: 400;
  }

  .check {
    position: absolute;
    right: 15px;
    top: 23px;

  }

  .el-checkbox__inner {
    border-radius: 8px;
  }

  .check .el-checkbox .el-checkbox__input .el-checkbox__inner {
    border-radius: 8px;
  }

  .card2 {
    width: 95%;
    height: 100px;
    display: flex;
    flex-direction: column;
    background: white;
    font-size: 0.9rem;

    font-weight: 400;
  }

  .card-head {
    width: 100%;
    height: 30%;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .card-head span {
    margin-left: 5px;
  }

  .card-content {
    width: 100%;
    height: 45%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .card-content2 {
    width: 100%;
    height: 20%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .content-one {
    width: 90%;
    display: flex;

  }

  .content-one span {
    width: 50%;
    text-align: left;

  }

  .content-two {
    width: 90%;
    display: flex;

  }

  .content-two span {
    margin-right: 10px;
    text-align: left;

  }

  .con {
    width: 45%;
    display: flex;
    align-items: center;
    justify-content: flex-start;
    font-size: 0.9rem;

    font-weight: 400;
  }

  .con span {
    margin-left: 5px;
  }

</style>
