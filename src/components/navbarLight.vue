<template>
    <scroller class="navbar-light" :show-scrollbar="false" scroll-direction="horizontal" :style="{height:height+'px',width : getPageWidth}">
        <div class="navbar-light__item" :ref="item.id" v-for="(item,index) in list" @click="handleClick($event,item,index)" :key="index">
            <text v-if="item.name" class="navbar-light__title" :class="[index==current ? 'navbar-light__active__text' : '']">{{item.name}}</text>
            <div class="navbar-light__line" :class="[index==current ? 'navbar-light__active__line' : '']" :style='{backgroundColor:color}'></div>
        </div>
        <div class="navbar-light__fix__bottom"></div>
    </scroller>
</template>

<script>
var dom = weex.requireModule('dom');
export default {
    name: 'navbar-light',
    props: {
        list: {
            type: Array,
            default: () => []
        },
        value: {
            type: Number,
            default: 0
        },
        height: {
            type: Number,
            default: 70
        },
        width: {
            type: Number,
            default: 750
        },
        color: {
            type: String,
            default: '#f39424'
        }
    },
    data() {
        return {
            pageWidth: '750px',
            current: 0,
        }
    },
    watch: {
        value(index) {
            this.current = index;
            this.changePositionScroll(index)
        }
    },
    methods: {
        handleClick(e, item, index) {
            this.changePosition(index, this.current)
            this.current = index;
            this.$emit('navbarClick', { item, index });
        },
        changePositionScroll(index) {
            this.$nextTick().then(() => {
                dom.scrollToElement(this.$refs[this.list[index].id][0], {
                    offset: -22
                });
            });
        },
        changePosition(index, currentTab) {
            if (!this.$isIPhone()) {
                this.$nextTick().then(() => {
                    dom.scrollToElement(this.$refs[this.list[index].id][0], {
                        offset: -300
                    });
                });
                return
            }
            // 往前
            if (currentTab < index) {
                if (index < 2) return
                this.$nextTick().then(() => {
                    dom.scrollToElement(this.$refs[this.list[index].id][0], {
                        offset: -300
                    });
                });
                // 向后
            } else if (currentTab > index) {
                if (index == 0) {
                    this.$nextTick().then(() => {
                        dom.scrollToElement(this.$refs[this.list[index].id][0], {
                            offset: -22
                        });
                    });
                } else if (index == 1) {
                    this.$nextTick().then(() => {
                        dom.scrollToElement(this.$refs[this.list[0].id][0], {
                            offset: -22
                        });
                    });
                } else {
                    this.$nextTick().then(() => {
                        dom.scrollToElement(this.$refs[this.list[index].id][0], {
                            offset: -300
                        });
                    });
                }
            }
        }
    },
    computed: {
        getPageWidth() {
            return this.$isIPad() ? this.pageWidth : this.width
        }
    },
    created() {
        this.current = this.value;
        this.$isIPad() && this.$getSystemSize().then(res => {
            this.pageWidth = res.NavigationSize.Width + 'wx';
        })
    },

}
</script>
<style>
.navbar-light {
    display: flex;
    flex-direction: row;
    align-items: center;
    background-color: #fff;
}
.navbar-light__item {
    position: relative;
    margin: 0 24px;
}
.navbar-light__title {
    font-size: 28px;
    color: #000;
    background-color: #fff;
    opacity: 0.6;
    padding: 10px 0;
}
.navbar-light__active__text {
    opacity: 1;
}
.navbar-light__line {
    position: absolute;
    left: 10px;
    right: 10px;
    bottom: 0;
    height: 3px;
    background-color: #fff;
}
.navbar-light__fix__bottom {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    height: 1px;
    background-color: #eee;
}
.navbar-light__active__line {
    background-color: #f39424;
}
</style>