<template>
    <div class="slideWrap">
        <h2>
            <router-link to="/">朴茶门店收银后台</router-link>
        </h2>
        <el-menu :default-active="getDefault" class="slideBar" theme="dark" router unique-opened>

            <!-- 一级菜单 -->
            <template v-for="(item,index) in slidebarConfig">

                <!-- 有二级菜单 -->
                <el-submenu :index="String(index)" v-if="item.children">
                    <template slot="title">
                        <i :class="item.icon" v-if="item.icon"></i>
                        <span>{{ item.title }}</span>
                        <div class="spot" v-if="item.spot"></div>
                    </template>

                    <template v-for="(todo,i) in item.children">
                        <!-- 有三级菜单 -->
                        <el-submenu :index="String(index) + '-' + String(i)" v-if="todo.children" class="slideBar-menu">
                            <template slot="title">
                                <i :class="todo.icon" v-if="todo.icon"></i>
                                <span>{{ todo.title }}</span>
                            </template>

                            <template v-for="(t,j) in todo.children">
                                <el-menu-item :index="t.path || String(index + 1) + '-' + String(i + 1) + '-' + String(j + 1)" :route="t.path">
                                    <i :class="t.icon" v-if="t.icon"></i>
                                    <span slot="title">{{ t.title }}</span>
                                    <span class="count" v-if="t.countName">{{ t.count }}</span>
                                </el-menu-item>
                            </template>
                        </el-submenu>

                        <!-- 没有三级菜单 -->
                        <el-menu-item :index="todo.path || String(index + 1) + '-' + (i + 1)" v-else>
                            <i :class="todo.icon" v-if="todo.icon"></i>
                            <span slot="title">{{ todo.title }}</span>
                            <span class="count" v-if="todo.countName">{{ todo.count }}</span>
                        </el-menu-item>
                    </template>

                </el-submenu>

                <!-- 没有二级菜单 -->
                <el-menu-item :index="item.path || String(index + 1)" v-else>
                    <i :class="item.icon" v-if="item.icon"></i>
                    <span slot="title">{{ item.title }}</span>
                    <span class="count" v-if="item.countName">{{ item.count }}</span>
                </el-menu-item>
            </template>
        </el-menu>
    </div>
</template>

<script>
import Vue from 'vue';
import slidebarConfig from './config/slidebar.config.js';
export default {
    data() {
        return {
            slidebarConfig,
            setting: {},
            url: [],
            index: '',
        }
    },
    methods: {
        //获取所有有count的的下拉菜单
        getName() {
            let data = this.setting.slideData;
            let counts = {};
            let setName = (obj) => {
                obj.forEach((val) => {
                    if (val.children) {
                        setName(val.children);
                    } else {
                        if (val.countName) {
                            counts[val.countName] = val;
                        }
                    }
                })
            }
            setName(data);
            this.setting.counts = counts;
        },
        setCount(name, num) {
            if (num > 99) {
                num = '99+';
            }
            this.setting.counts[name].count = num;
            if (num === 0) {
                this.setting.counts[name].count = '';
            }
            for (let i = 0; i < this.setting.slideData.length; i++) {
                if (this.setting.slideData[i].spotName === name) {
                    this.setting.slideData[i].spot = true;
                    if (num === 0) {
                        this.setting.slideData[i].spot = false;
                    }
                }
            }
            return num;
        },
        
    },
    computed:{
        getDefault() {
            return this.$route.path
        }
    },
    created() {
        this.setting.slideData = this.slidebarConfig;
        this.getName();
        this.setting.setCount = this.setCount;
        Vue.prototype.$slide = this.setting;
    }
}
</script>



<style lang="less" scoped>
@import url('../assets/less/slidebar.less');
</style>
<style>
.el-submenu__icon-arrow {
    transform: rotateZ(270deg);
}

.el-submenu.is-opened>.el-submenu__title .el-submenu__icon-arrow {
    transform: rotateZ(360deg);
}
</style>
