<template>
    <div class="cl-box flex-wrap col-flex">
        <v-nus v-if="isNull" txt="您还没有购买过卡豆～"></v-nus>
        <yd-infinitescroll v-if="!isNull" :on-infinite="loadList" class="scroll-wrap">
            <div slot="list" class="all-item-box">


                <div v-for="em in DATA" class="c-item-box flex-wrap col-flex">
                    <div class="se-title">{{em.transaction_id}}</div>
                    <div class="flex-wrap row-flex">
                        <div class="page se-txt">购买卡豆：<i class="v-now">{{em.beanmuch}}</i></div>
                        <div class="page se-txt">实付金额：<i class="se-red">¥ {{em.payfee}}</i></div>
                    </div>
                    <div class="se-foot">{{em.transaction_datetime}}</div>
                </div>


            </div>

            <!-- 数据全部加载完毕显示 -->
            <span slot="doneTip">～我是有底线滴～</span>

            <!-- 加载中提示 -->
            <div slot="loadingTip" class="loading visible load-boxs">
                <span class="loading-ring"> </span>
            </div>

        </yd-infinitescroll>
    </div>
</template>
<script>
import XHR from '../../api/service'
    export default {
        data() {
            return {
                page: 1,
                isNull: false,
                DATA:[]
            }
        },
        created () {
            this.$dialog.loading.open('数据加载中…')
            this.loadList()
        },
        methods: {
            loadList() {
                let self = this
                let json = {}
                json.pg = this.page
                XHR.payOrder(json)
                .then(function (res) {
                    // console.log(res)
                    if (res.data.state == '1') {
                        setTimeout(() => {
                            self.$dialog.loading.close()
                        }, 400)
                        if(res.data.body.pagerecord >= 10 && res.data.body.pagerecord !== 0){
                            if( self.page == res.data.body.pagecount){
                                window.$yduiBus.$emit('ydui.infinitescroll.loadedDone')
                            } else {
                                self.page++
                            }
                        } else {
                            window.$yduiBus.$emit('ydui.infinitescroll.loadedDone')
                        }
                        self.DATA.push(...res.data.body.payorderlist)
                        window.$yduiBus.$emit('ydui.infinitescroll.finishLoad')
                        if(self.DATA.length == '0'){
                            self.isNull = true
                        }
                    } else {
                        setTimeout(() => {
                            self.$dialog.loading.close()
                            self.$dialog.toast({
                                mes: res.data.errmsg,
                                timeout: 2000,
                                icon: 'error'
                            })
                        }, 400)
                    }
                })
                .catch(function (err) {
                    
                })

            },
        }
    }
</script>
<style lang="less" scoped>
.cl-box{width: 100%; height: 100%;}


.c-item-box{
    height: 1.9rem;
    padding: 0.12rem 0.3rem;
    border-bottom: 0.02rem solid #eee;
    background-color: #fff;
}

.se-title,.se-txt,.se-foot{font-size:0.24rem; color: #666;height: 0.6rem; line-height: 0.6rem;overflow: hidden; text-overflow:ellipsis; white-space:nowrap; border-bottom: 0.02rem solid #eee;}
.se-txt{color:#333;font-size:0.26rem;  border-bottom: 0; padding-top: 0.04rem;}
.se-foot{border-bottom: 0; height: 0.4rem; line-height: 0.4rem;}
.se-red{color: red;}
.v-now{color: #ff6500; font-weight: 500; padding-right: 0.2rem;}
.v-now:before{ content: '\e602'; font-family:'iconfont'; margin-right: 0.1rem; color: #ff9800;}

</style>
