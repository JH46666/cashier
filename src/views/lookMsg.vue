<template>
    <div id="lookMsg">
        <div class="ql-container ql-snow">
            <div class="ql-editor" v-html="content" ref="editor">

            </div>
        </div>
    </div>
</template>

<script>
let qs = require('qs');
export default {
    data() {
        return {
            noticeId: '',       //id
            content: '',        //内容
            title: '',
            cardData: [],
            copyContent: '',        //内容
            url: '',             //url
            count: 0,
            arr: [],
            arr1: []
        }
    },
    methods: {
        // 获取url
        getUrl() {
            this.content = this.content + `<p style="width:600px;font-weight: 600;font-size: 28px;color: #333333;text-align: center;margin: 20px auto;overflow: hidden;white-space: nowrap;text-overflow: ellipsis;" title="${this.title}">${this.title}</p>`
            if (/<p>卡片/.test(this.copyContent)) {
                this.copyContent.split('<p>卡片</p>').map((value, index, arr) => {
                    value.split('</p><p>链接：').map(v => {
                        v.split('</p><p>sku：').map(v => {
                            if (/https:/.test(v)) {
                                this.url = this.url + v + ',';
                                this.count++;
                            }
                        })
                    })
                });
            } else {
                this.content = this.content + this.copyContent;
            }
        },
        // 保留两位小数
        toFixed(num) {
            return Number(num).toFixed(6).substring(0, Number(num).toFixed(6).lastIndexOf('.') + 3);
        },
    },
    watch: {
        count(v) {
            if (v > 0) {
                this.$ajax.post('/api/http/NoticeInfo/createCard.jhtml',
                    qs.stringify({
                        cardUrl: this.url
                    }),
                    {
                        headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                        },
                    }
                ).then(res => {
                    this.cardData = res.data.result.productList;
                    this.arr = this.copyContent.split('<p>卡片</p>');
                    this.arr.map((value, index, arr) => {
                        this.arr1 = value.split('<p>/卡片</p>').map((v, i) => {
                            if (/<p>标题：/.test(v)) {
                                this.content = this.content +
                                    `<div style="width: 100%;height: 240px;background-color: #f7f9fa;position: relative;margin-top:5px">
                                        <img style="width: 200px;position: absolute;top: 20px;left: 20px;" src="${this.cardData[index - 1].imgUrl}">
                                        <p style="position: absolute;top: 25px;left: 240px;line-height: 18px;font-size: 18px;font-weight: 600;overflow: hidden;white-space: nowrap;text-overflow: ellipsis;width:700px" title="${this.cardData[index - 1].title}">${this.cardData[index - 1].title}</p>
                                        <p style="position: absolute;top: 59px;left: 240px;line-height: 14px;font-size: 14px;color: #999999;width:700px" title="${this.cardData[index - 1].subTitle}">${this.cardData[index - 1].subTitle}</p>
                                        <p style="position: absolute;top: 95px;left: 240px;line-height: 14px;font-size: 14px;background-color: #e3e5e6;width:700px;height:1px;"></p>
                                        <p style="position: absolute;top: 114px;left: 240px;line-height: 14px;font-size: 14px;color: #999999;">醉品价：</p>
                                        <p style="position: absolute;top: 116px;left: 335px;line-height: 14px;font-size: 14px;color: #333333;font-weight:550;">￥${this.toFixed(this.cardData[index - 1].salesPrice)}</p>
                                        <p style="position: absolute;top: 147px;left: 240px;line-height: 14px;font-size: 14px;color: #999999;">代理价：</p>
                                        <p style="position: absolute;top: 147px;left: 335px;line-height: 16px;font-size: 16px;color: #ff9819;font-weight:550;">￥${this.toFixed(this.cardData[index - 1].purchasePrice)}</p>
                                        <div style="position: absolute;top: 162px;left: 650px;width:98px;height:38px;line-height: 38px;border-radius: 4px;border: solid 1px #ff9819;font-size: 14px;color: #ff9819;text-align: center;cursor: pointer;" onClick="window.open('${this.cardData[index - 1].cardUrl}')">查看详情</div>
                                        <div style="position: absolute;top: 162px;left: 800px;width:98px;height:38px;line-height: 38px;border-radius: 4px;border: solid 1px #ff9819;font-size: 14px;color: #ffffff;background-color:#ff9819;text-align: center;cursor: pointer;">加入进货单</div>
                                    </div>`
                            } else {
                                this.content = this.content + v;
                            }
                        })
                    });
                })
            }
        }
    },
    created() {
        this.$route.query.content ? this.copyContent = this.$route.query.content : null;
        this.$route.query.title ? this.title = this.$route.query.title : null;
        this.getUrl();
        if (this.$route.query.id) {
            this.noticeId = this.$route.query.id;
            this.$ajax.post('/api/http/NoticeInfo/getNoticeInfoDetail.jhtml',
                qs.stringify({
                    'noticeId': this.noticeId,
                }),
                {
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded',
                    },
                }
            ).then(res => {
                if (res.data.success === 1) {
                    this.copyContent = res.data.result.noticeContent;
                    this.title = res.data.result.noticeTitle;
                    this.getUrl();
                } else {
                    this.$message({
                        message: res.data.msg,
                        type: 'error'
                    });
                }
            })
        }
    }
}
</script>

<style lang="less" scoped>
.ql-container {
    margin: 20px auto;
    background: #ffffff;
    width: 1002px;
}
</style>


