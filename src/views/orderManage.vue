<template>
    <div class="orderManage">
        <div class="warp">
            <div id="search">
                <el-form label-width="85px" ref="form" :model="searchData">
                    <el-row :gutter="20">
                        <el-col :span="6">
                            <el-form-item label="代理商名称">
                                <el-input v-model="searchData.searchName" @keyup.enter.native="onSumbit" placeholder="代理商名称"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-col :span="5">
                            <el-form-item label="进货单号">
                                <el-input v-model="searchData.searchOrderNo" @keyup.enter.native="onSumbit" placeholder="进货单号"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-col :span="5">
                            <el-form-item label="状态" label-width="50px">
                                <el-select v-model="searchData.searchState" clearable placeholder="请选择">
                                    <el-option label="待审核" value="WAIT_CHECK"></el-option>
                                    <el-option label="待发货" value="WAIT_SEND"></el-option>
                                    <el-option label="已发货" value="DELIVERED"></el-option>
                                    <el-option label="已完成" value="FINISH"></el-option>
                                    <el-option label="退款成功" value="RETURN_FINISH"></el-option>
                                </el-select>
                            </el-form-item>
                        </el-col>
                        <el-col :span="6">
                            <el-form-item label="运营人员">
                                <el-input v-model="searchData.searchOperator" @keyup.enter.native="onSumbit" placeholder="运营人员"></el-input>
                            </el-form-item>
                        </el-col>

                        <!-- // <el-col :span="12">
                        //     <el-form-item label="运营人员">
                        //         <! <span class="search_left" v-if="!(addForm.operator==='')" @click="searchOperator"></span>-->                                                               
                        <!-- //         <el-autocomplete 
                        //         v-model="searchData.searchOperator"                  
                        //         :fetch-suggestions="operatorQuerySearchAsync" 
                        //         @select="handleoperatorSelect"
                        //         placeholder="请选择"
                        //         icon="caret-bottom"
                        //         >                       
                        //         </el-autocomplete>
                        //     </el-form-item>                    
                        // </el-col> -->



                        <el-col :span="2">
                            <el-button type="primary" @click='onSumbit'>查询</el-button>
                        </el-col>
                    </el-row>
                    <el-row :gutter="20">
                        <el-col :span="6">
                            <el-form-item label="代理商等级">
                                <el-select v-model="searchData.searchLevel" clearable multiple placeholder="全部">
                                    <el-option label="区域代理" value="265"></el-option>
                                    <el-option label="微店代理" value="266"></el-option>
                                    <el-option label="单店代理" value="31"></el-option>
                                </el-select>
                            </el-form-item>
                        </el-col>

                        <el-col :span="6">
                            <el-form-item label="下单时间" label-width="72px" style="padding-left:10px;padding-left:5px">
                                <el-date-picker width="200" v-model="searchData.searchTime" type="daterange" placeholder="选择日期范围" style="width:78%"></el-date-picker>
                            </el-form-item>
                        </el-col>
                    </el-row>
                </el-form>
            </div>
            <div class="orderList">
                <el-table border :data="tableData" style="margin: 20px auto;font-size: 14px;" @sort-change='store'>
                    <el-table-column prop="purchaseOrderNo" label="进货单号" width="180">
                    </el-table-column>
                    <el-table-column prop="agentGradeId" label="代理商编号" style="position: relative">
                        <template slot-scope="scope">{{ scope.row.shopNo }}
                            <p class="textBlue" v-if="scope.row.agentGradeId === 31">单店</p>
                            <p class="textOrange" v-if="scope.row.agentGradeId === 265">区域</p>
                            <p class="textYellow" v-if="scope.row.agentGradeId === 266">微店</p>
                        </template>
                    </el-table-column>
                    <el-table-column prop="name" label="代理商名称">
                    </el-table-column>
                    <el-table-column prop="receiptName" label="收件人">
                    </el-table-column>
                    <el-table-column prop="stateName" label="状态">
                    </el-table-column>
                    <el-table-column prop="orderTime" label="下单时间">
                    </el-table-column>
                    <el-table-column prop="orderSum" label="金额" sortable="custom">
                        <template slot-scope="scope">
                            <p>{{ toFixed(scope.row.orderSum) }}</p>
                        </template>
                    </el-table-column>
                    <el-table-column prop="operator" label="运营人员">
                    </el-table-column>
                    <el-table-column prop="phone" label="操作">
                        <template slot-scope="scope">
                            <router-link :to="{ name: 'orderInfo', params: { purchaseOrderNo: scope.row.purchaseOrderNo,shopNo:scope.row.shopNo }}">详情</router-link>
                        </template>
                    </el-table-column>
                </el-table>
                <div class="page">
                    <el-pagination style="float: right;margin-right: 50px" @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="currentPage" :page-size="pageSize" layout="total,prev, pager, next, jumper" :total="totalNums">
                    </el-pagination>
                </div>
            </div>
        </div>
    </div>
</template>

<script type="text/javascript" src="../router.js"></script>
<script>
export default {
    data() {
        return {
            currentPage: 1, //当前页
            totalNums: 0, //数据总数
            pageSize: 30, //当前页数
            total: '', //数据总数
            order: '', //排序方式
            searchData: {
                searchPhone: '', //代理商手机
                searchName: '', //代理商名称
                searchOrderNo: '', //进货单号
                searchState: '', //订单状态
                searchTime: '', //下单时间
                searchLevel: [], //代理商等级
                Level: [], //代理商等级替代
                searchOperator:''  //运营人员
            },
            tableData: [{
                purchaseOrderNo: '', //进货单号
                agentGradeId: '', //代理商等级
                shopNo: '', //代理商编号
                phone: '', //手机号
                name: '', //代理商名称
                stateName: '', //状态
                orderTime: '', //下单时间
                orderSum: '', //金额
                operator:'',   //运营人员
                receiptName:'', //收件人
            }]
        }
    },
    methods: {
        //判断是否超时
        checkSession() {
            const self = this;
            if (window.sessionStorage) {
                let nowDate = new Date().getTime();
                let time = (nowDate - sessionStorage.haha) / 1000
                //超过30秒没操作，重新登录
                if (time > 1800) {
                    self.$router.push('/login');
                    self.$message({
                        message: '登录超时，请重新登录',
                    })
                    return false;
                } else {
                    sessionStorage.haha = nowDate;
                    return true;
                }
            }
        },
        onSumbit() {
            this.currentPage=1;
            if (!this.checkSession()) return;
            console.log(this.searchData.searchTime);
            console.log(this.searchData.searchTime[0]);
            var temp = new Date(this.searchData.searchTime[0]);
            if (temp.getFullYear() > 2006) {
                var time1 = temp.getFullYear();
                if ((temp.getMonth() + 1) < 10) {
                    time1 = time1 + '-0' + (temp.getMonth() + 1);
                } else {
                    time1 = time1 + '-' + (temp.getMonth() + 1);
                }
                if (temp.getDate() < 10) {
                    time1 = time1 + '-0' + temp.getDate();
                } else {
                    time1 = time1 + '-' + temp.getDate();
                }
                console.log(time1);
                temp = new Date(this.searchData.searchTime[1]);
                var time2 = temp.getFullYear();
                if ((temp.getMonth() + 1) < 10) {
                    time2 = time2 + '-0' + (temp.getMonth() + 1);
                } else {
                    time2 = time2 + '-' + (temp.getMonth() + 1);
                }
                if (temp.getDate() < 10) {
                    time2 = time2 + '-0' + temp.getDate();
                } else {
                    time2 = time2 + '-' + temp.getDate();
                }
                console.log(time2);
            } else {
                var time1 = '';
                var time2 = '';
            }
            if (this.searchData.searchLevel != '') {
                this.searchData.level = this.searchData.searchLevel.join(',');
            } else {
                this.searchData.level = '';
            }
            console.log(this.searchData.searchLevel);
            const self = this;
            self.$ajax({
                url: '/api/http/purchaseOrder/queryPurchaseOrderList.jhtml',
                method: 'post',
                data: {
                    'pager.pageIndex': 1,
                    'pager.pageSize': this.pageSize,
                    'purchaseOrder.phone': this.searchData.searchPhone,
                    'purchaseOrder.name': this.searchData.searchName,
                    'purchaseOrder.purchaseOrderNo': this.searchData.searchOrderNo,
                    'purchaseOrder.state': this.searchData.searchState,
                    'purchaseOrder.agentGradeIds': this.searchData.level,
                    'purchaseOrder.startTime': time1,
                    'purchaseOrder.endTime': time2,
                    'purchaseOrder.operator':this.searchData.searchOperator,
                    
                },
                transformRequest: [function(data) {
                    // Do whatever you want to transform the data
                    let ret = ''
                    for (let it in data) {
                        ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
                    }
                    return ret;
                }],
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded'
                }
            }).then(function(response) {
                if (response.data.success === 1) {
                    self.tableData = response.data.result;
                    self.totalNums = response.data.totalNums;
                } else {
                    self.$message({
                        message: response.data.msg,
                        type: 'error'
                    })
                }
            }).catch(function(error) {

            });
        },
        handleSizeChange(val) {
            console.log(`每页 ${val} 条`);
        },  
        // handleoperatorSelect(item) {
            
            
        // },
        // operatorQuerySearchAsync(queryString, callback) {
        //         var list = [{}];
        //         //调用的后台接口
        //         let url = '/api/http/purchaseOrder/queryPurchaseOrderList.jhtml?userUnit=operator' + '&userName=' + queryString;
        //         //从后台获取到对象数组
        //         axios.get( url ).then((response)=>{
        //             //在这里为这个数组中每一个对象加一个value字段, 因为autocomplete只识别value字段并在下拉列中显示
        //             for (let i of response.data.result){
        //                 i.value = i.userName;  //将CUSTOMER_NAME作为value
        //             }

        //             if(!queryString){
        //                 console.log(response.data.result)
                        
        //                 for(let item of response.data.result){
        //                     list.push(item)
        //                 }
                        
        //                 callback(list);

        //             }else{

        //             let  QS =  queryString.toLocaleLowerCase();
                        
                
                    
        //             for(let item of response.data.result){
        //                 if(item.pinyin.indexOf(QS) > -1 || item.userName.indexOf(QS) > -1 ){
        //                     list.push(item)
        //                 }
        //             }
        //             // console.log(list);
        //             if(list.length==1){
        //                 list.push({value:`没有匹配结果"${queryString}"`}); 
        //             } 
        //             }
        //             callback(list);
        //         }).catch((error)=>{
        //             console.log(error);
        //         });
        //     },


        handleCurrentChange(val) {
            if (!this.checkSession()) return;
            console.log(this.searchData.searchTime);
            console.log(this.searchData.searchTime[0]);
            var temp = new Date(this.searchData.searchTime[0]);
            if (temp.getFullYear() > 2006) {
                var time1 = temp.getFullYear();
                if ((temp.getMonth() + 1) < 10) {
                    time1 = time1 + '-0' + (temp.getMonth() + 1);
                } else {
                    time1 = time1 + '-' + (temp.getMonth() + 1);
                }
                if (temp.getDate() < 10) {
                    time1 = time1 + '-0' + temp.getDate();
                } else {
                    time1 = time1 + '-' + temp.getDate();
                }
                console.log(time1);
                temp = new Date(this.searchData.searchTime[1]);
                var time2 = temp.getFullYear();
                if ((temp.getMonth() + 1) < 10) {
                    time2 = time2 + '-0' + (temp.getMonth() + 1);
                } else {
                    time2 = time2 + '-' + (temp.getMonth() + 1);
                }
                if (temp.getDate() < 10) {
                    time2 = time2 + '-0' + temp.getDate();
                } else {
                    time2 = time2 + '-' + temp.getDate();
                }
                console.log(time2);
            } else {
                var time1 = '';
                var time2 = '';
            }
            if (this.searchData.searchLevel != '') {
                this.searchData.level = this.searchData.searchLevel.join(',');
            } else {
                this.searchData.level = '';
            }
            this.$getData({
                url: 'http/purchaseOrder/queryPurchaseOrderList.jhtml',
                data: {
                    'pager.pageIndex': val,
                    'pager.pageSize': this.pageSize,
                    'purchaseOrder.phone': this.searchData.searchPhone,
                    'purchaseOrder.name': this.searchData.searchName,
                    'purchaseOrder.purchaseOrderNo': this.searchData.searchOrderNo,
                    'purchaseOrder.state': this.searchData.searchState,
                    'purchaseOrder.agentGradeIds': this.searchData.level,
                    'purchaseOrder.startTime': time1,
                    'purchaseOrder.endTime': time2,
                    'purchaseOrder.sort': 'orderSum',
                    'purchaseOrder.order': this.order,
                    "purchaseOrder.operator ": this.searchData.searchOperator,
                },
                success(response) {
                    this.tableData = response.data.result;
                    this.totalNums = response.data.totalNums;
                },
                fail(response) {
                    this.$message({
                        message: response.data.msg,
                        type: 'error'
                    })
                },
                error(response) {
                    this.$message({
                        message: response.data.msg,
                        type: 'error'
                    })
                }
            });
            console.log(`当前页: ${val}`);
        },
        store(row, column) {
            if (!this.checkSession()) return;
            console.log(row);
            if (row.order === 'ascending') {
                this.order = 'asc';
            }
            if (row.order === 'descending') {
                this.order = 'desc';
            }
            if (!this.checkSession()) return;
            var temp = new Date(this.searchData.searchTime[0]);
            if (temp.getFullYear() > 2006) {
                var time1 = temp.getFullYear();
                if ((temp.getMonth() + 1) < 10) {
                    time1 = time1 + '-0' + (temp.getMonth() + 1);
                } else {
                    time1 = time1 + '-' + (temp.getMonth() + 1);
                }
                if (temp.getDate() < 10) {
                    time1 = time1 + '-0' + temp.getDate();
                } else {
                    time1 = time1 + '-' + temp.getDate();
                }
                console.log(time1);
                temp = new Date(this.searchData.searchTime[1]);
                var time2 = temp.getFullYear();
                if ((temp.getMonth() + 1) < 10) {
                    time2 = time2 + '-0' + (temp.getMonth() + 1);
                } else {
                    time2 = time2 + '-' + (temp.getMonth() + 1);
                }
                if (temp.getDate() < 10) {
                    time2 = time2 + '-0' + temp.getDate();
                } else {
                    time2 = time2 + '-' + temp.getDate();
                }
                console.log(time2);
            } else {
                var time1 = '';
                var time2 = '';
            }
            if (this.searchData.searchLevel != '') {
                this.searchData.level = this.searchData.searchLevel.join(',');
            } else {
                this.searchData.level = '';
            }
            this.$getData({
                url: 'http/purchaseOrder/queryPurchaseOrderList.jhtml',
                data: {
                    'pager.pageIndex': 1,
                    'pager.pageSize': this.pageSize,
                    'purchaseOrder.phone': this.searchData.searchPhone,
                    'purchaseOrder.name': this.searchData.searchName,
                    'purchaseOrder.purchaseOrderNo': this.searchData.searchOrderNo,
                    'purchaseOrder.state': this.searchData.searchState,
                    'purchaseOrder.agentGradeIds': this.searchData.level,
                    'purchaseOrder.startTime': time1,
                    'purchaseOrder.endTime': time2,
                    'purchaseOrder.sort': 'orderSum',
                    'purchaseOrder.order': this.order,
                    'purchaseOrder.name':this.searchData.searchName,
                    'purchaseOrder.operator ':this.searchData.operator,
                },
                success(response) {
                    this.tableData = response.data.result;
                    this.totalNums = response.data.totalNums;
                    this.$message({
                        message: '查询成功',
                        type: 'success'
                    })
                },
                fail(response) {
                    this.$message({
                        message: response.data.msg,
                        type: 'error'
                    })
                },
                error(response) {
                    this.$message({
                        message: response.data.msg,
                        type: 'error'
                    })
                }
            });
        },
        toFixed(num) {
            return Number(num).toFixed(6).substring(0, Number(num).toFixed(6).lastIndexOf('.') + 3);
        },
    },
    created() {
        if (!this.checkSession()) return;
        this.$getData({
            url: 'http/purchaseOrder/queryPurchaseOrderList.jhtml',
            data: {
                pageIndex: this.currentPage,
                pageSize: this.pageSize,
            },
            success(response) {
                this.tableData = response.data.result;
                this.totalNums = response.data.totalNums;
            },
            fail(response) {
                this.$message({
                    message: response.data.msg,
                    type: 'error'
                })
            },
        });
    },
}
</script>

<style lang="less" scoped>
@import url('../assets/less/orderManage.less');
</style>