<template>
  <div class="pos">
    <div>
          <el-row>
            <el-col :span='7' class="pos-order" id="order-list">
              <el-tabs class="elTabs">
                  <el-tab-pane label="点餐">
                    <el-table :data="tableData" border style="width: 100%">
                      <el-table-column prop="goodsName" label="商品"></el-table-column>
                      <el-table-column prop="count" label="数量" width="50" align="center"></el-table-column>
                      <el-table-column prop="price" label="金额" width="70" align="center"></el-table-column>
                      <el-table-column label="操作" width="100" fixed="right" align="center">
                        <template slot-scope="scope">
                            <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                            <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                        </template>
                      </el-table-column>
                    </el-table>  
                  </el-tab-pane>
                  <el-tab-pane label="挂单">
                  挂单
                  </el-tab-pane>
                  <el-tab-pane label="外卖">
                  外卖
                </el-tab-pane>
              </el-tabs>
                <div class="totalDiv">
                    <small>数量：</small>
                    <strong>{{totalCount}}</strong> &nbsp;&nbsp;&nbsp;&nbsp;
                    <small>总计：</small>
                    <strong>{{totalMoney}}</strong> 元
                </div>
              <div class="order-btn">
                  <el-button type="warning" >挂单</el-button>
                  <el-button type="danger" @click="delAllGoods">删除</el-button>
                  <el-button type="success" @click="checkout">结账</el-button>
              </div>
            </el-col>

            <!--商品展示-->
            <el-col :span="17">
              <div class="often-goods">
                  <div class="title">常用商品</div>
                  <div class="often-goods-main">
                      <ul>
                          <li class="often-goods-list" v-for="item in oftenGoods" :key="item.index" @click="addOrderList(item)">
                              <span>{{item.goodsName}}</span>
                              <span class="o-price">￥{{item.price}}元</span>
                          </li>
                      </ul>
                  </div>
              </div>
              <div class="goods-type">
                <el-tabs class="elTabs">
                    <el-tab-pane label="汉堡">
                        <div class="often-goods-main">
                            <ul class='cookList-main'>
                                <li class='cookList' v-for="item in type0Goods" :key="item.index" @click="addOrderList(item)">
                                    <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                                    <span class="foodName">{{item.goodsName}}</span>
                                    <span class="foodPrice">￥{{item.price}}元</span>
                                </li>
                            </ul>
                        </div> 
                    </el-tab-pane>
                    <el-tab-pane label="小食">
                        <div class="often-goods-main">
                            <ul class='cookList-main'>
                                <li class='cookList' v-for="item in type1Goods" :key="item.index" @click="addOrderList(item)">
                                    <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                                    <span class="foodName">{{item.goodsName}}</span>
                                    <span class="foodPrice">￥{{item.price}}元</span>
                                </li>
                            </ul>
                        </div> 
                    </el-tab-pane>
                    <el-tab-pane label="饮料">
                        <div class="often-goods-main">
                            <ul class='cookList-main'>
                                <li class='cookList' v-for="item in type2Goods" :key="item.index" @click="addOrderList(item)">
                                    <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                                    <span class="foodName">{{item.goodsName}}</span>
                                    <span class="foodPrice">￥{{item.price}}元</span>
                                </li>
                            </ul>
                        </div> 
                    </el-tab-pane>
                </el-tabs>                  
              </div>  
            </el-col>
        </el-row>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "Pos",
  data() {
    return {
      tableData: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalMoney: 0,
      totalCount: 0
    };
  },
  created() {
    axios
      .get("http://jspang.com/DemoApi/oftenGoods.php")
      .then(res => {
        this.oftenGoods = res.data;
      })
      .catch(err => {
        console.log(err);
      });
    axios
      .get("http://jspang.com/DemoApi/typeGoods.php")
      .then(response => {
        this.type0Goods = response.data[0];
        this.type1Goods = response.data[1];
        this.type2Goods = response.data[2];
        this.type3Goods = response.data[3];
      })
      .catch(err => {
        console.log(err);
      });
  },
  mounted() {
    let orderH = document.body.clientHeight;
    document.getElementById("order-list").style.height = orderH + "px";
  },
  methods: {
    addOrderList(info) {
      this.totalCount = 0; //汇总数量清0
      this.totalMoney = 0;
      let isHave = false;
      //判断是否这个商品已经存在于订单列表
    //   for (let i = 0; i < this.tableData.length; i++) {
    //     if (this.tableData[i].goodsId == info.goodsId) {
    //       isHave = true;
    //     }
    //   }

    for(let item of this.tableData){
        if (item.goodsId == info.goodsId) {
            isHave = true;
        } 
    }

      //根据isHave的值判断订单列表中是否已经有此商品
      if (isHave) {
        // fliter过滤后，把tableData数组中符合条件的那个对象在内存中的地址赋值给了arr，然后arr就指向了tableData中符合条件的那个对象
        let arr = this.tableData.filter(o => o.goodsId == info.goodsId);
        arr[0].count++;
        arr[0].price = arr[0].count * arr[0].danjia;
      } else {
        let newGoods = {
          goodsId: info.goodsId,
          goodsName: info.goodsName,
          price: info.price,
          count: 1,
          danjia: info.price
        };
        this.tableData.push(newGoods);
      }

      this.getAllMoney();
    },
    //删除单个商品
    delSingleGoods(goods) {
      if (goods.count > 1) {
        let arr2 = this.tableData.filter(o => o.goodsId == goods.goodsId);
        arr2[0].count--;
        arr2[0].price = arr2[0].price - arr2[0].danjia;
      } else {
        this.tableData = this.tableData.filter(o => goods.goodsId != o.goodsId);
      }
      this.totalCount--;
      this.totalMoney=this.totalMoney-goods.danjia;

      if(this.tableData.length==0){
        this.totalCount = 0;
        this.totalMoney = 0;
      }
    },
    
    //删除所有商品
    delAllGoods() {
        this.tableData = [];
        this.totalCount = 0;
        this.totalMoney = 0;
    },
    //汇总数量和金额
    getAllMoney() {
      this.totalCount = 0; //汇总数量清0
      this.totalMoney = 0;
      this.tableData.forEach(element => {
        this.totalCount += element.count;
        this.totalMoney += element.price;
      });
    },
    checkout(){
        if(this.totalCount!=0){
            this.$message({
            message: '结账成功',
            type: 'success'
            });
            this.delAllGoods();
        }else{
            this.$message.error('不能空结！');
        }
    }
  }
};
</script>

<style scoped>
.elTabs {
  margin: 0 10px;
}
.order-btn {
  margin-top: 10px;
  text-align: center;
}
/* .pos {
    font-size: 12px;
} */
.pos-order {
  background-color: #f9fafc;
  border-right: 1px solid #c0ccda;
}
.title {
  height: 20px;
  border-bottom: 1px solid #d3dce6;
  background-color: #f9fafc;
  padding: 10px;
}
.often-goods-list {
  list-style: none;
  float: left;
  border: 1px solid #e5e9f2;
  padding: 10px;
  margin: 5px;
  background-color: #fff;
  cursor: pointer;
}
.o-price {
  color: #58b7ff;
}
.goods-type {
  clear: both;
}
.cookList {
  list-style: none;
  width: 23%;
  border: 1px solid #e5e9f2;
  height: auto;
  overflow: hidden;
  background-color: #fff;
  padding: 2px;
  float: left;
  margin: 2px;
  cursor: pointer;
}
.cookList span {
  display: block;
  float: left;
}
.foodImg {
  width: 40%;
}
.foodName {
  font-size: 18px;
  padding-left: 10px;
  color: brown;
}
.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}
.totalDiv {
  height: auot;
  overflow: hidden;
  text-align: right;
  font-size: 16px;
  background-color: #fff;
  border-bottom: 1px solid #e5e9f2;
  padding: 10px;
}
</style>
