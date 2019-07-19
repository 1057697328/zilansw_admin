<template>
  <div>
    <el-row>
      <el-col :span="24">
        <el-table :data="tableData"
                  style="width: 100%">
          <el-table-column prop="orderid" label="订单编号"></el-table-column>
          <el-table-column prop="createTime" :formatter="formatDate" label="订单创建时间"></el-table-column>
          <el-table-column prop="orderdetails" label="订单详情" width="270">
            <template slot-scope="scope">
              <el-table :data="scope.row.orderdetails" :border="true" style="width:100%;">
                <el-table-column prop="goods.gname" label="商品名称"></el-table-column>
                <el-table-column prop="num" :formatter="(data)=>{return data.num+'件'}" label="商品数量"></el-table-column>
                <el-table-column prop="price" :formatter="(data)=>{return data.price+'元'}"
                                 label="商品单价"></el-table-column>
              </el-table>
            </template>
          </el-table-column>
          <el-table-column prop="status" :formatter="statusFormat" label="订单状态"></el-table-column>
          <el-table-column prop="address" :formatter="addressFormat" label="送货信息"></el-table-column>
          <el-table-column prop="totalprice" :formatter="(data)=>{return data.totalprice+'元'}"
                           label="订单总额"></el-table-column>
          <el-table-column prop="expnumber" label="快递单号"></el-table-column>
          <el-table-column prop="order" label="操作" width="220">
            <template slot-scope="scope">
              <!--未支付-->
              <el-row v-if="scope.row.status===0">
                <el-col :span="24">
                  <el-tag effect="dark" type="info">此订单暂未支付, 无需执行任何操作</el-tag>
                </el-col>
              </el-row>

              <!--已支付-->
              <el-row v-if="scope.row.status===1">
                <el-col :span="12" style="margin-bottom:0.5rem;">
                  <el-button icon="el-icon-check" type="success" round @click="updateOrder(scope.row,2)">发货
                  </el-button>
                </el-col>
                <el-col :span="12">
                  <el-button icon="el-icon-money" type="danger" round @click="updateOrder(scope.row,5)">退款
                  </el-button>
                </el-col>
              </el-row>

              <!--已发货-->
              <el-row v-if="scope.row.status===2">
                <el-col :span="24">
                  <el-tag effect="dark" type="primary">等待客户收货...</el-tag>
                </el-col>
              </el-row>

              <!--已签收-->
              <el-row v-if="scope.row.status===3">
                <el-col :span="24">
                  <el-tag effect="dark" type="success">已签收</el-tag>
                </el-col>
              </el-row>

              <!--已提交退款申请-->
              <el-row v-if="scope.row.status===4">
                <el-col :span="24" style="margin-bottom:0.5rem;">
                  <el-tag effect="dark" type="warning">客户已提交退款申请, 请尽快处理</el-tag>
                </el-col>
                <el-col :span="24" style="margin-bottom:0.5rem;">
                  <el-button icon="el-icon-close" type="primary" style="width: 100%"  round @click="showAlert(scope.row.returnreason,()=>{return false;})">查看退款理由</el-button>
                </el-col>
                <el-col :span="12" style="margin-bottom:0.5rem;">
                  <el-button icon="el-icon-money" type="danger" round @click="updateOrder(scope.row,5)">退款</el-button>
                </el-col>
                <el-col :span="12" style="margin-bottom:0.5rem;">
                  <el-button icon="el-icon-close" type="warning" round @click="updateOrder(scope.row,6)">拒绝</el-button>
                </el-col>
              </el-row>

              <!--同意退款-->
              <el-row v-if="scope.row.status===5">
                <el-col :span="24">
                  <el-tag effect="dark" type="warning">该订单已完成退款, 退款金额￥{{scope.row.refundprice}}元</el-tag>
                </el-col>
              </el-row>

              <!--不同意退款-->
              <el-row v-if="scope.row.status===6">
                <el-col :span="24" style="margin-bottom:0.5rem;">
                  <el-button icon="el-icon-document" type="primary" round @click="showAlert(scope.row.returnreason,()=>{return false;})">退款理由</el-button>
                </el-col>
                <el-col :span="24">
                  <el-button icon="el-icon-document" type="danger" round @click="showAlert(scope.row.rejectreason,()=>{return false;})">拒绝退款理由</el-button>
                </el-col>
              </el-row>


            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>


    <el-dialog title="快递单号" :visible.sync="showexpdialog">
      <el-form :model="updateData">
        <el-form-item label="请填写货物快递运单号">
          <el-input v-model="updateData.expnumber" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="showexpdialog = false">取 消</el-button>
        <el-button type="primary" @click="updateStatus()">确 定</el-button>
      </div>
    </el-dialog>


  </div>
</template>

<script>
  import CONSTANT from '@/assets/constant';
  import Qs from 'qs';

  export default {
    name: "ordermanager",
    data() {
      return {
        orderStatus:0,
        CONSTANT: CONSTANT,
        tableData: [],
        pageData: {},
        updateData: {
          orderid: "",
          status: 0,
          expnumber: "",
          rejectreason: "",
          refundprice: 0
        },
        //展示填写快递单号Dialog
        showexpdialog: false
      }
    },
    methods: {
      /**
       * 分页获取订单列表信息
       * @param pageindex
       */
      getList(pageindex) {
        this.axios.post("/admin/zorder/getList?pageindex=" + pageindex, {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        }).then((json) => {
          if (json.data.code !== CONSTANT.statusCode.SUCCESS) {
            this.$alert(json.data.msg);
            return false;
          } else {
            this.tableData = json.data.data.list;
            this.pageData = json.data.data;
          }
        })
      },
      /**
       * 时间格式化
       */
      formatDate(date) {
        date = new Date(date.createTime);
        let fmt = "yyyy-MM-dd hh:mm:ss";
        let o = {
          "M+": date.getMonth() + 1,     //月份
          "d+": date.getDate(),     //日
          "h+": date.getHours(),     //小时
          "m+": date.getMinutes(),     //分
          "s+": date.getSeconds(),     //秒
          "q+": Math.floor((date.getMonth() + 3) / 3), //季度
          "S": date.getMilliseconds()    //毫秒
        };
        if (/(y+)/.test(fmt))
          fmt = fmt.replace(RegExp.$1, (date.getFullYear() + "").substr(4 - RegExp.$1.length));
        for (var k in o)
          if (new RegExp("(" + k + ")").test(fmt))
            fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
        return fmt;
      },
      /**
       * 格式化更新订单状态时间
       */
      formatUpdateDate(date) {
        date = new Date(date);
        let fmt = "yyyy-MM-dd hh:mm:ss";
        let o = {
          "M+": date.getMonth() + 1,     //月份
          "d+": date.getDate(),     //日
          "h+": date.getHours(),     //小时
          "m+": date.getMinutes(),     //分
          "s+": date.getSeconds(),     //秒
          "q+": Math.floor((date.getMonth() + 3) / 3), //季度
          "S": date.getMilliseconds()    //毫秒
        };
        if (/(y+)/.test(fmt))
          fmt = fmt.replace(RegExp.$1, (date.getFullYear() + "").substr(4 - RegExp.$1.length));
        for (var k in o)
          if (new RegExp("(" + k + ")").test(fmt))
            fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
        return fmt;
      },
      /**
       * 状态格式
       */
      statusFormat(data) {
        let status = data.status;
        switch (status) {
          case 0:
            return "未支付";
          case 1:
            return "已支付";
          case 2:
            return "已发货";
          case 3:
            return "已签收";
          case 4:
            return "用户已提交退款申请";
          case 5:
            return "已同意退款申请";
          case 6:
            return "已拒绝退款申请";
          case 7:
            return "已退款";
          case 8:
            return "用户已取消订单";
          case 9:
            return "已完成"
        }
      },
      /**
       * 收货地址格式化
       * @param data
       * @returns {string}
       */
      addressFormat(data) {
        //省
        let province = data.province;
        //市
        let city = data.city;
        //区/县
        let district = data.district;
        //详细地址
        let address = data.address;
        //收件人姓名
        let name = data.name;
        //收件人电话
        let phone = data.phone;
        return province + "-" + city + "-" + district + "-" + address + " " + name + "-" + phone;
      },
      /**
       * 修改订单信息
       * @param order 订单对象
       * @param status 订单状态
       */
      updateOrder(order,status) {
        switch (status) {
          case 2:
            this.updateData=order;
            this.orderStatus=status;
            this.showexpdialog = true;
            break;
          case 5:
            this.updateData=order;
            this.orderStatus=status;
            this.$prompt('请输入退款金额', '系统提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              inputPlaceholder:"输入带小数点的纯数字,且不大于订单总金额",
            }).then(({ value }) => {
              let refundprice = parseFloat(value);
              if(refundprice>this.updateData.totalprice)
              {
                this.showAlert("您输入的金额大于订单总额,操作失败",()=>{
                  return false;
                })
              }
              else
              {
                this.updateData.refundprice=parseFloat(value);
                this.updateStatus();
              }
            }).catch(() => {

            });
            break;
          case 6:
            this.updateData=order;
            this.orderStatus=status;
            this.$prompt('请输入拒绝退款的理由', '系统提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
            }).then(({ value }) => {
              this.updateData.rejectreason=value;
              this.updateStatus();
            }).catch(() => {

            });
            break;
          default:
            break;
        }
      },
      /**
       * 修改订单
       */
      updateStatus() {
        this.showexpdialog=false;
        let that = this;
        delete that.updateData.orderdetails;

        that.updateData.status=that.orderStatus;
        //日期格式化
        that.updateData.createTime=that.formatUpdateDate(that.updateData.createTime);

        this.axios.post("/admin/zorder/update", Qs.stringify(
          that.updateData
        ), ).then((json) => {
          if(json.data.code!==CONSTANT.statusCode.SUCCESS)
          {
            this.$alert(json.data.msg,"系统提示",{
              confirmButtonText:"确定",
              callback:action => {
                return false;
              }
            })
          }
          else
          {
            this.$alert(json.data.msg,"系统提示",{
              confirmButtonText:"确定",
              callback:action => {
                this.getList(this.pageData.pageindex);
              }
            })
          }
        });
      },
      /**
       * 弹出消息框
       * @param message
       * @param callback
       */
      showAlert(message,callback)
      {
        this.$alert(message, '系统提示', {
          confirmButtonText: '确定',
          callback:callback
        });
      }
    },
    mounted() {
      this.getList(1);
    }
  }
</script>

<style scoped>

</style>
