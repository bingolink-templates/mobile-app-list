<template>
    <div class="subtitle" @click="handleClick" :style="{marginTop:marginTop+'px'}">
        <div class="subtitle__line" :style='{backgroundColor:color}'></div>
        <div class="subtitle__desc">
            <text class="subtitle__title">{{title}}</text>
            <text class="subtitle__subtitle">{{tip}}</text>
        </div>

        <div v-if="hasMore" @click="moreClick" class="subtitle__more">
            <text class="subtitle__text" @click="moreClick">{{moreText}}</text>
        </div>
    </div>
</template>
<script>

export default {
    name: 'subtitle',
    props: {
        title: {
            type: String,
            default: ''
        },
        tip: {
            type: String,
            default: ''
        },
        secondType: {
            type: String,
            default: ''
        },
        flag: {
            type: String,
            default: ''
        },
        hasMore: {
            type: Boolean,
            default: false
        },
        marginTop: {
            type: Number,
            default: 20
        },
        moreText: {
            type: String,
            default: '更多'
        },
        color: {
            type: String,
            default: '#f39424'
        },
    },
    data() {
        return {

        }
    },
    methods: {
        handleClick() {
            this.$emit('subtitleClick', {
                secondType: this.secondType,
                flag: this.flag
            });
        },
        moreClick() {
            this.$emit('moreClick', {
                secondType: this.secondType,
                flag: this.flag
            });
        }
    },
    created() {
    }
}
</script>
<style>
.subtitle {
    position: relative;
    display: flex;
    flex-direction: row;
    align-items: center;
    background-color: #fff;
    padding: 0 24px;
}
.subtitle__line {
    width: 5px;
    height: 36px;
    background-color: #f39424;
    margin-right: 12px;
}
.subtitle__desc {
    display: flex;
    flex-direction: row;
    align-items: center;
    flex: 1;
    padding: 24px 0;
}
.subtitle__title {
    font-size: 30px;
    color: #333;
    text-overflow: ellipsis;
    lines: 1;
}
.subtitle__subtitle {
    margin-left: 20px;
    font-size: 30px;
    color: #f39424;
    text-overflow: ellipsis;
    lines: 1;
}
.subtitle__more {
    flex-direction: row;
    align-items: center;
}
.subtitle__text {
    font-size: 30px;
    color: #333;
    text-overflow: ellipsis;
    lines: 1;
}
</style>