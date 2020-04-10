<template>
    <div style="flex:1;background-color: #fff;">
        <appList></appList>
    </div>
</template>

<!--引入bui-weex样式文件-->
<style lang="scss" src="bui-weex/src/css/buiweex.scss"></style>

<script>
    import appList from "./appList.vue";
    module.exports = {
        data() {
            return {}
        },
        components: {
            appList
        }
    }
</script>

<style>
</style>